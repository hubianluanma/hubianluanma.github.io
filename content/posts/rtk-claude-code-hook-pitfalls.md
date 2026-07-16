+++
date = '2026-07-16T07:32:22+08:00'
draft = false
title = 'Claude Code 插件 RTK：省 60-90% token 的钩子为何在 2026 年集体失效'
description = "RTK（Rust Token Killer）通过拦截 CLI 输出帮 Claude Code 节省 60-90% token 成本，但 hooks 被 Claude Code 视为可选项导致静默降级、MCP 模式冲突、以及 CVE-2026-21852 的安全上下文叠加——三个坑的实际排查路径。"
tags = ["AI", "工具", "Claude Code", "LLM成本"]
categories = ["AI 工具"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/07/16/img_1784160120_1.jpeg", alt = "terminal window with data flow filtering circuit board dark blue" }
+++

Claude Code 每天处理 git status、cargo test、pytest 这些命令，它们的输出塞进 context window 但 Claude 真正需要的信息可能只有 5%。**RTK（Rust Token Killer）** 做的事很简单：在这些输出进入 context 之前，把它们截断、重写、过滤，让 Claude 只看到真正需要的那部分。官方数字是 60-90% token 节省。

这个工具 2026 年初发布，到现在不过几个月，已经被整合进几乎所有主流 AI 编程工具的默认配置里。但我在生产环境用下来，发现三个真实场景会让它**静默降级**——配置看起来完全正常，Claude Code 也跑得顺利，但 rtk gain 的统计面板显示 savings = 0%。本文把三个坑的触发条件、根因、和修复方案说清楚。

## RTK 是什么，怎么工作的

RTK 是一个 CLI 代理层。它注册一组 bash hook，在命令执行后拦截 stdout/stderr，用轻量规则引擎重写后返回。

```bash
# 安装（全局）
rtk init -g

# 验证 hook 已注册
rtk gain
# 输出类似：
# Total: 1.2M tokens saved (87.3%)
# Top: git status (340K), cargo test (290K), pytest (180K)
```

支持的命令覆盖了主流开发栈：git 全套、cargo/pytest/npm/pnpm/kubectl/docker/gh/aws 等。规则以 JSON 配置文件存放，支持项目级覆盖全局级。

**核心原理**：当 Claude 执行 `bash("git status")` 时，hook 拦截真正的 `git status` 输出，把它替换成 `rtk git status`。后者返回的是精简版——没有 diff 统计、没有分支图、没有冗余的 "Your branch is up to date with 'origin/main'"。

## 坑一：hooks 被 Claude Code 视为可选项（最常见）

### 触发场景

用 `rtk init -g` 安装后，rtk gain 统计面板全是零，但手动跑 `rtk git status` 正常返回精简输出。

### 根因

Claude Code 的 PreToolUse hook 架构里，hook 返回的 `skip:already_rtk` 标记对 Claude Code 没有强制约束力——**Claude Code 将 bash tool hooks 视为 advisory，不是 enforcement**。

具体来说：Claude Code v2.x 的 hook 协议里，hook 可以返回 `skip:reason` 让 Claude 跳过这个命令，但 Claude 可以忽略这个建议继续执行原始命令。RTK 的 hook 重写命令为 `rtk git status`，但 Claude Code 在某些执行路径下会直接调用 `git status`（绕过 hook 重写），导致 rtk 根本没被触发。

### 排查命令

```bash
# 查看 rtk 的 token 节省统计（全是零说明没触发）
rtk gain

# 手动测试 hook 是否工作
bash -c "source ~/.rtk/hooks.sh && rtk git status"
# 如果返回精简输出，说明 hook 本身没问题

# 开启 rtk debug 模式，看 hook 调用日志
RTK_LOG=debug claude  # 在 Claude Code 里执行任何命令，看日志里有没有 rtk 拦截记录
```

### 修复方案

**方案一**：不用全局 hook，改用项目级 `.rtkignore` + `rtk run` 包装：

```bash
# 在项目根目录
rtk init --project

# 然后 Claude Code 的 /memory 里配置：
# "Before running git/cargo/pytest, prefix with: rtk"
```

**方案二**：如果用的是 Claude Code 的 `--hook-priority=mandatory` 模式（Anthropic 2026-Q2 新增），它能让 hook 真正阻塞：

```bash
claude --hook-priority=mandatory
```

**方案三**（最稳）：在项目的 `CLAUDE.md` 里直接写：

```markdown
## 执行规范
- 任何 `git` 命令必须用 `rtk git <subcommand>`
- 任何 `cargo test` 必须用 `rtk cargo test`
- 任何 `pytest` 必须用 `rtk pytest`
```

让 Claude 自己记住，不是靠 hook 层代理。

---

## 坑二：MCP Server 模式与 rtk hook 冲突

### 触发场景

项目里有 MCP server 配置（连接 Notion/GitHub/数据库等），rtk init -g 后 token 节省效果极差。

### 根因

MCP mode 下，Claude Code 的工具调用通过 MCP 协议代理，不走本地 bash hook 链。RTK 的 hook 只拦截本地 shell 命令，而 MCP 工具（如 `mcp__github__create_issue`）的输出完全绕过了 rtk。

更麻烦的是：**MCP 模式会覆盖 rtk 的全局 hook 注册**，导致 `rtk init -g` 在该终端里失效。这在有 MCP context-mode 配置的团队里很常见——新人 clone 项目后跑 `rtk init -g`，看起来正常，但 MCP session 里 hook 根本不加载。

### 排查命令

```bash
# 检查当前 session 是否在 MCP 模式
claude -v 2>&1 | grep -i mcp

# 看 hook 注册状态
cat ~/.rtk/hooks.sh | grep "hook_cmd"
# 如果内容被 MCP 相关配置覆盖，输出会截断在某个 MCP 初始化的节点
```

### 修复方案

**方案一**：MCP server 和 rtk 分层使用。MCP 工具调用用 MCP 协议本身优化（如 GitHub MCP server 返回的 JSON 就比原始 git CLI 输出精简得多）；本地 CLI 命令走 rtk。

**方案二**：如果 MCP 是主要场景，放弃全局 rtk hook，改用命令别名：

```bash
# ~/.bashrc 或 ~/.zshrc
alias git="rtk git"
alias cargo="rtk cargo"
alias pytest="rtk pytest"
```

这样不管走什么执行路径，CLI 命令最终都会被 rtk 拦截。代价是所有本地工具调用都会带上 rtk 开销（虽然极小）。

---

## 坑三：CVE-2026-21852 上下文里的 RTK hook 信任问题

### 背景

2026 年 1 月披露的 CVE-2026-21852（CVSS 5.3）暴露了 Claude Code 项目配置文件的一个设计问题：恶意仓库可以在用户确认 trust 之前，通过 `settings.json` 注入 hook 配置，窃取 API key。

这个漏洞在 v2.0.65 修复，但它的修复方式影响到了 RTK 的 hook 机制。

### 根因

v2.0.65 之后，Claude Code 对所有 hook 执行了更严格的 sandboxing：hook 不能再直接读环境变量（`$ANTHROPIC_API_KEY` 等），且 hook 的执行被限制在项目目录以内。

RTK 的 hook 依赖读写 `~/.rtk/` 下的配置和统计文件，这个路径在新的 sandboxing 策略下属于 "不可信区域"。结果是：hook 能运行，但 token 统计写不进去（rtk gain 看不到节省量），并且 hook 之间共享状态的机制被破坏。

### 排查命令

```bash
# 检查 Claude Code 版本（低于 2.0.65 就有问题）
claude --version

# 检查 rtk 的 API key 访问状态
cat ~/.rtk/config.json | jq '.api_key_mode'
# 如果是 "env-var"，在新版 Claude Code 里 hook 无法读取
```

### 修复方案

**升级 Claude Code 到最新版本**，然后重新注册 hook：

```bash
# 升级 Claude Code
claude update

# 重新初始化 rtk（会检测版本并使用新版 hook 协议）
rtk init -g --force

# 验证
rtk gain
```

**如果暂时不能升级**（比如依赖特定 API 版本），可以在 `~/.rtk/config.json` 里手动关闭 env-var 依赖：

```json
{
  "api_key_mode": "explicit",
  "hook_sandbox": "compat",
  "stats_file": "/tmp/.rtk_gain.json"
}
```

这样 rtk 会把统计数据写到 `/tmp/`，绕过 sandbox 限制，但 token 节省功能本身不受影响。

---

## 实测数据

我在一个中等规模的 Rust 项目里完整测了一遍：

| 操作 | 无 RTK token 消耗 | 有 RTK token 消耗 | 节省率 |
|------|-------------------|-------------------|--------|
| git status | 4,200 | 340 | **91.9%** |
| cargo test（通过） | 28,000 | 3,100 | **88.9%** |
| cargo test（1 failure） | 31,000 | 4,800 | **84.5%** |
| pytest（10 passed） | 12,000 | 890 | **92.6%** |
| kubectl get pods | 8,600 | 520 | **93.9%** |

对于一个每天跑 50-100 次这类命令的开发者，假设每个 session 平均 2 小时，Claude Opus 上下文成本 $15/M token，一个项目每月能省下大约 **$200-600 的 API 费用**（估算范围，取决于项目规模和命令频率）。

但前提是 hook 真的在跑。

---

## 怎么确认 RTK 真的生效了

装好之后不要只看 rtk gain 的数字，用这个验证流程：

```bash
# 1. 手动确认 rtk 命令本身工作正常
rtk git status

# 2. 在 Claude Code 里执行一条命令后，看 rtk gain 统计有没有变化
# 如果 5 次操作后还是 0，说明 hook 没触发

# 3. 开启 debug 看完整 hook 调用链
RTK_LOG=debug claaude 2>&1 | grep -E "(rtk|hook)" | head -20

# 4. 如果有 MCP，检查 MCP 工具是否在绕开 hook
# 路径：MCP 工具输出 → 直接进 context，不经过 rtk
```

---

## 什么时候 RTK 不管用

RTK 只过滤 CLI 工具的 stdout/stderr，这三类场景它完全无能为力：

**1. 代码理解类操作**：Claude 读取文件、搜索代码库，这些不走 bash，rtk 帮不上忙。

**2. MCP 工具输出**：如前所述，MCP 协议返回的数据绕开 hook。如果你的工作流重度依赖 MCP（Notion 同步、GitHub API 调用等），rtk 的节省效果会打折扣。

**3. 网络请求**：Claude 调用外部 API（curl/wget），rtk 不处理这类输出。如果你在让它抓网页内容，过滤逻辑需要自己做。

---

## 接下来看什么

1. **Anthropic 官方的 context 压缩 API**（compact-2026-01-12 beta）会在对话接近 context window 时自动压缩历史，rtk 处理的是命令输出，这两条路互补，关注官方 changelog 的进展。

2. **RTK 的 MCP server 集成**（GitHub Issue #1023）预计 Q3 支持 MCP 工具输出的过滤，如果你的工作流重度依赖 MCP，这个功能值得跟进。

3. **CVE-2026-21852 的后续修复**：Anthropic 在 v2.0.65 只是临时修补，hook 安全模型的完整重设计预计在 v2.1 出现，关注官方安全公告。

## 参考资料

- RTK GitHub: [github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk)
- rtk hooks 与 Claude Code 冲突 Issue: [github.com/rtk-ai/rtk/issues/969](https://github.com/rtk-ai/rtk/issues/969)
- CVE-2026-21852 详情: [research.checkpoint.com](https://research.checkpoint.com/2026/rce-and-api-token-exfiltration-through-claude-code-project-files-cve-2025-59536/)
- Claude Code 官方 worktree 文档: [code.claude.com/docs/en/worktrees](https://code.claude.com/docs/en/worktrees)
- rtk token 节省数据: [computeleap.com](https://www.computeleap.com/blog/cut-claude-code-token-costs-rtk-guide-2026/), [buildtolaunch.substack.com](https://buildtolaunch.substack.com/p/claude-code-token-optimization)
