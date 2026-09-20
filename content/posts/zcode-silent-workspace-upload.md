+++
date = '2026-09-20T13:28:14+08:00'
draft = false
title = 'ZCode 被曝静默上传整个工作区：87% 是 .git 历史，解密密钥只在智谱云端'
description = "开发者 ferstar 逆向分析发现，智谱 Z.ai 的 AI 编程工具 ZCode 会在登录后静默打包整个工作区（含完整 Git 历史、LFS 缓存、reflog），加密后直传阿里云 OSS——而加密私钥只存在于智谱服务器。两个隐私开关均无效，唯一的防御手段是操作系统级的目录锁。"
tags = ["AI", "安全", "隐私"]
categories = ["AI观察"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/09/20/zcode-cover.jpeg", alt = "云端加密文件库示意图" }
+++

9 月 18 日，开发者 ferstar 发布了一篇逆向分析文章，揭开了智谱（Z.ai）官方 AI 编程桌面应用 ZCode 的一个惊人行为：**只要用户处于登录状态，客户端就会静默打包整个工作区——包括完整的 .git 目录、Git LFS 资产缓存、reflog 和全局应用配置——加密后直传阿里云 OSS**。整个过程不出现在任何界面提示里，隐私政策中也没有一个字提到。

这篇原本只是"清理磁盘时发现 ~/.zcode 占了 700 多 MB"的日常排查，最终演变成 2026 年 AI 编程工具领域最严重的信任危机之一。

## 发现过程：从 313MB 的待传文件开始

ferstar 在释放磁盘空间时发现 ZCode 的数据目录 `~/.zcode` 异常庞大，其中 `v2/checkpoints/` 目录下躺着一个 313MB 的 `.enc` 加密文件和一份状态元数据：

```json
{
  "workspacePath": "/Users/ferstar/myprojects/<某个商业项目>",
  "lastCompressedSize": {
    "encryptedSizeBytes": 313070842,
    "workspaceSizeBytes": 345549173
  },
  "kind": "baseline",
  "failureCount": 564
}
```

一个 345MB 的商业项目被整体打包成 313MB 的加密归档，标注为 baseline（全量快照），因为超过大小限制已经**失败重传了 564 次**，仍在本地 pending 目录里排队等下一轮重试。

对文件清单（Manifest 是明文保存在本地的）进一步拆解，42,411 个文件的构成令人咋舌：

- `.git/lfs/`：196.1MB，占 56.8%——LFS 缓存，所有曾下载的二进制资产
- `.git/objects/`：102.2MB，占 29.6%——完整提交历史对象库
- `.git/logs/`：reflog，本地分支历史和未推送的操作痕迹
- 真正的源码和文档：只有约 46.2MB，占 13.4%

**.git 目录一个就占了整个上传包的 86.6%**。这意味着云端拿到的远不只是你当前的代码，而是这个仓库自第一天以来的全部谱系：后来在提交里删掉的历史 API key 和敏感配置、从未推送过的本地分支名（直接暴露未发布的产品计划）、`.git/config` 里的内部 GitLab 主机名和仓库路径。

此外还有一个跨工作区打包的 `repo_snapshot_extra_manifest`，把你 ZCode 的全局配置文件哈希一并捎上。

## 上传链路：两段式直传阿里云

ferstar 拆开客户端的 `app.asar` 还原了完整链路：

1. 客户端向 `zcode.z.ai` 请求上传凭证，服务器返回 OSS 表单签名、动态 Object Key、大小限制，以及**本轮加密用的 RSA 公钥**；
2. 客户端本地打 tar.gz 包、用 AES-256-CTR 流式加密、RSA-OAEP-SHA256 封装对称密钥；
3. 然后**绕过智谱自己的应用服务器**，直接通过 HTTP POST 表单把 `tar.gz.enc` 传到阿里云 OSS，OSS 再回调智谱后端登记快照。

运行进程的活跃 Socket 也印证了这一点：ZCode 进程与 zcode.z.ai 的 IP 以及两个阿里云 OSS 存储节点保持着长连接。

## 最讽刺的部分：钥匙在服务器手里

这是整个事件里最值得反复咀嚼的细节。ZCode 用的是教科书式的**信封加密**：

- 内容用一次性对称密钥（AES-256-CTR）加密；
- 对称密钥用服务器下发的 RSA 公钥做 RSA-OAEP-SHA256 封装；
- **对应的私钥从不接触你的机器，只存在于智谱云端**。

ferstar 用系统里所有本地私钥尝试解封全部失败。换句话说：那个躺在你自己磁盘上、由你自己的文件打出来的几百 MB 密文，**你和 ZCode 客户端自己都打不开，只有智谱后端能打开**。

如果这个功能真的是面向用户的回滚或跨设备同步而设计，密钥应该像 Git 或 Time Machine 一样保存在本地。一把只有服务器能用的密钥，用途只有一个：**保证服务器随时可以读你的代码**。

## 开关是假的

正常人发现后的第一反应是进设置关掉它。ferstar 把 UI 选项和代码交叉比对后给出的结论更糟：

| 开关 | 你以为的作用 | 实际作用 |
| --- | --- | --- |
| Optimize Experience | 关闭遥测/数据收集 | 只控制数据是否授权用于模型训练，快照打包上传照常运行 |
| Repo Snapshot Indexing | 关闭快照功能 | 只控制服务器是否索引已上传的快照，本地打包和上传不间断 |

汇编级代码显示：采集/上传 sidecar 在启动时被**无条件实例化**，用户偏好上没有任何门控判断，唯一要求是 `tokenProvider` 能返回有效 JWT。**只要登录，这条后台管线就永久活跃，没有任何 UI 设置能关掉它。**

采集触发点有两处：`captureBeforePrompt`（每次提问前）和任务完成时（带 `repo-wiki-update` 标签）。会话日志里，单个活跃会话最多产生过 62 次采集事件。

## 智谱的回应与未竟之问

9 月 18 日 17:44，智谱在用户社区发布官方声明（IT之家当晚跟进报道），要点包括：

- 问题来自"代码库索引"功能，用于本地索引、会话检查点恢复和 Repo Wiki；
- 生成 Wiki 页面时"可能"触发仓库数据上传；
- Wiki 生成后上传数据"立即销毁，不留存"；
- 该功能上线初期默认开启，"问题已修复"；
- ZCode 将尽快开源接受第三方审查，所有用户补偿一次每周配额重置。

ferstar 随后对比了旧版（3.12.3，事发版本）与新版（3.14.0）客户端，确认新版物理移除了上传管线、云端凭证端点已返回 404——**上传事实本身智谱已不再否认**。但他也明确指出，声明与证据之间仍有多个对不上的地方：

- 官方称上传由"生成 Wiki"触发，而逆向显示是**登录即驻留、每次提问前无条件采集**；
- 官方称数据"生成后立即销毁"，但这与官方自己宣称的"检查点恢复"功能逻辑矛盾——云端恢复必然需要留存；
- "立即销毁"从外部无法验证：已有的云端加密快照是否被物理清除？谁持有私钥的解密权？
- 一个小型公开仓库（538 个文件，压缩加密后约 15KB）的快照确实被服务器成功接收——"数据到底有没有离开过机器"的答案是：至少这一份离开了。

事件后续还在发酵：9 月 20 日，太原成明科技向智谱发出正式法律函件，称涉及数据包括源代码、系统架构、数据库密码和云服务凭证，要求智谱说明数据存储、跨境传输、第三方共享和模型训练用途，彻底删除服务器、缓存和备份中的相关数据，并提供访问日志和删除证明。

## 防御：删除是打地鼠，锁目录才是终点

ferstar 最初只是删掉了待传包，结果半小时内客户端就重新打包了一份新的，重试计数器从 564 跳到了 565。手动删除是打地鼠，可靠的方案是在文件系统层面加不可变标志，在内核层面拒绝写入：

macOS：

```bash
rm -rf ~/.zcode/v2/checkpoints
mkdir -p ~/.zcode/v2/checkpoints
chflags uchg ~/.zcode/v2/checkpoints
# 验证：touch 应报 "Operation not permitted"
```

Linux：

```bash
rm -rf ~/.zcode/v2/checkpoints
mkdir -p ~/.zcode/v2/checkpoints
sudo chattr +i ~/.zcode/v2/checkpoints
```

代价是"检查点回滚/时间线"功能失效（它本来就以上传代码为前提），聊天、补全、工具调用不受影响。要恢复的话，macOS 执行 `chflags nouchg`，Linux 执行 `sudo chattr -i`。

## 为什么这事比"又一个隐私丑闻"更严重

ZCode 今年 7 月发布时的卖点恰恰是**信任**。彼时 Anthropic 的 Claude Code 刚因隐藏遥测陷入争议，智谱把 ZCode 定位成更透明的替代方案，高管还在 X 上公开回应"不会实现网站列明之外的任何间谍软件"。而 ferstar 本人正是 GLM Coding Max 的长期付费用户——他 9 月 17 日晚上还在开发者群里兴致勃勃地安利 ZCode，不到半天就被现实打脸。

往大了看，这是今年第二次"AI 编程工具整仓上传云端"事件（此前 Grok Build 被曝把整仓传到 Google Cloud），暴露的是一个结构性问题：

**模型的权重是开源的，不代表跑模型的 harness 是可信的。** GLM 系列权重开放、社区里有大量本地部署，但 ZCode 是闭源的官方 harness，很多人误以为"GLM 开源 = ZCode 开源"。权重开放和客户端可信之间，隔着一整条数据链路。

对开发者的启示朴素而直接：

1. **把 AI 编程工具默认当作会带走你代码的软件来评估**，无论它来自谁、模型是否开源；
2. 用它之前先读隐私政策里"收集什么"的清单，但更关键的是**逆向层面的实际行为**——这次就是政策没写、客户端照传；
3. 商业代码、含密钥历史的仓库，要么用本地模型 + 开源 harness（如 Aider、Continue），要么至少做一次网络层审计（看进程连了哪些 IP）；
4. 涉密项目的工作区和日常编码工作区物理隔离，密钥用短期凭证并定期轮换——因为 .git 历史里的旧密钥永远不会"删了就没有"。

工具是工具，但边界要用户自己划。当软件不给你关闭的开关时，就用操作系统内核给它造个笼子。

## 参考资料

- [Inside ZCode: Silently Uploading Your Entire Git History to the Cloud — ferstar](https://blog.ferstar.org/en/posts/zcode-silent-workspace-snapshot-upload)（一手逆向分析，2026-09-18 发布，09-19 更新）
- [ZCode secretly uploads entire project workspaces to the cloud — AlexTech.ai](https://alextech.ai/en/news/zcode-secretly-uploads-entire-project-workspaces-to-the-cloud)
- [ZCode, the GLM coding agent, silently uploads your Git history — arc-codex](https://arc-codex.com/article/a9c0c507eb4fe5bb78d824336659ff3a)
- [Zhipu's free coding agent uploaded whole repositories to the cloud — ainvest](https://ainvest.com/news/zhipu-free-coding-agent-uploaded-repositories-cloud-trust-price-real-moat-2609)
- [ZCode Uploads Your Git History: Settings Do Nothing — ByteIota](https://byteiota.com/zcode-uploads-your-git-history-settings-do-nothing)
- [ZCode uploads your whole repo — .git history included, keys held by Z.ai — ai-tldr](https://ai-tldr.dev/releases/zai-zcode-workspace-upload)
- [太原成明科技就 ZCode 代码上传事件向智谱发函 — TechFlow](https://techflowpost.com/en-US/newsletter/136973)
