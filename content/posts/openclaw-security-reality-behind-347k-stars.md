+++
date = '2026-06-23T08:00:38+08:00'
draft = false
title = 'OpenClaw 的 347K 星只是幻觉：企业安全团队真正该担心什么'
description = "GitHub 史上增长最快的开源项目，5 个月 347K stars，3.2M 用户，500K 实例。但 CVE-2026-25253 分数 8.8，微软把它定性为「不可信代码执行」。这场安全债务危机会如何收场？"
tags = ["AI", "AI观察", "安全", "开源"]
categories = ["AI观察"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/23/img_1782172944_1.jpeg", alt = "数字龙虾剪影，锁链断裂，暗蓝色调" }
+++

2026 年 1 月底，GitHub 上一个几乎没人听过的项目在 48 小时内疯涨了 3.4 万颗星。5 个月后，OpenClaw 突破 34.7 万星，超越 Linux 和 React，成为 GitHub 史上星标增长最快的开源项目。

数字很好看。但如果你在企业安全团队工作，这些数字让你睡不着觉。

## 一个项目的病毒式成功，和它欠下的债

OpenClaw 的增长曲线确实惊人：

- **347,000+ GitHub stars**（5 个月内，史上最快）
- **3800 万月访问量**
- **320 万活跃用户**
- **50 万+ 正在运行的实例**
- **44,000+ ClawHub 技能插件**
- **180 家初创公司月收入超过 32 万美元**

这个数字让整个行业眩晕。但安全研究员看到的不是成绩单，而是一张漏洞清单。

## 真实的安全漏洞，不是小打小闹

CVE 记录讲的是实话。OpenClaw 在 2026 年已经背负了多个高危漏洞：

| CVE 编号 | 类型 | CVSS 评分 |
|---|---|---|
| CVE-2026-25253 | 恶意 URL 静默窃取认证 Token | **8.8** |
| CVE-2026-24763 | 命令注入 | 高 |
| CVE-2026-26322 | 服务器端请求伪造（SSRF） | 高 |
| CVE-2026-26329 | 路径遍历，本地文件读取 | 高 |
| CVE-2026-30741 | 提示词注入驱动代码执行 | 高 |

CVE-2026-25253 的攻击方式很典型：攻击者构造一个恶意 URL，用户点击后认证 Token 会被静默外传，全程无提示。这意味着攻击者可以接管整个网关。对于运行着 50 万+ 实例的项目来说，这个漏洞的暴露窗口期意味着什么，不难想象。

微软安全团队在 2026 年 2 月的博文中给出了最直白的定性：

> OpenClaw 应被视为**不可信代码执行（untrusted code execution with persistent credentials）**。它不适合在标准个人工作站或企业工作站上运行。

翻译成非安全语言：微软认为 OpenClaw 的风险模型要求你把它当成病毒一样隔离运行。这和它宣传的「开箱即用的私人 AI 助手」人设之间，有巨大的落差。

## 为什么增长跑赢了安全？

OpenClaw 的安全问题不是疏忽，而是结构性的。

**第一，漏洞披露与修复之间有时间差。** CVE-2026-32979 从披露到在 2026.3.11 版本打补丁之间，有真实的暴露窗口。在开源社区，补丁 release 不等于用户更新——50 万个运行实例里，有多少在打补丁之前就已经被攻破，永远无法统计。

**第二，插件生态是供应链攻击的完美入口。** ClawHub 有 44,000+ 第三方技能插件，任何一个被污染，都可能通过 OpenClaw 的执行上下文获得完整系统权限。44,000 这个数字和 npm package 被投毒的逻辑一样：量大了之后，没有任何人能真正审计每个插件。

**第三，企业安全团队面对的是双刃剑。** OpenClaw 的核心卖点是「自托管」——企业可以把 AI agent 跑在自己的服务器上，数据不出境。这对有数据主权要求的企业是刚需。但自托管也意味着每个企业都要独自承担安全运维责任，而大多数企业的安全团队根本没有能力审计一个 AI agent 的完整行为面。

## Google Project Mariner 的关闭，不是竞争失败，而是方向验证

几乎在 OpenClaw 曝光漏洞的同一周，Google 在 5 月 4 日关闭了 Project Mariner——那个雄心勃勃的浏览器 AI agent 项目。存活时间：17 个月。

官方说法是技术整合进 Gemini Agent 和 Chrome Auto Browse。但真实原因是浏览器-based AI agent 面临根本性的工程挑战：**高算力成本、视觉理解延迟、跨站点状态管理的复杂性**。OpenClaw 的崛起本质上在证明另一条路——API-first agent 方向——比浏览器 automation 更快被市场接受。

这对 OpenClaw 是个坏消息。Project Mariner 的失败让行业更迫切地需要 API-first agent 框架，而 OpenClaw 正是这个方向的扛把子。但需求越迫切，企业把它引入生产环境的速度就越快，安全债务积累的速度也就越快。

## 被忽视的结构性信号：安全研究力量没有跟上

OpenClaw 2026 年最大的风险，可能不是某个具体的 CVE，而是**安全研究社群对它的关注度远远不够**。

React 花了 8 年才建立完善的安全审计生态。OpenClaw 5 个月就拥有了 50 万个实例，但公开的安全研究博客、CVE 响应速度、企业级安全指南，都还在非常初级的阶段。DigitalOcean 在 2026 年中列举的 7 大 OpenClaw 安全挑战，多数还在「提醒注意」层面，没有成熟的缓解方案。

这意味着：OpenClaw 的安全成熟度不是「落后一点」，而是**落后了数量级**。

## 接下来看什么

1. **CVE-2026-32979 的后续影响**：暴露窗口期内的实际攻击规模目前无法评估，可能在 Q3 2026 看到实际受害者的披露。

2. **ClawHub 插件供应链审查机制是否会建立**：44,000+ 插件没有代码签名或审计机制，这是定时炸弹。2026 年下半年是否会出现类似 npm 的投毒事件？

3. **企业安全策略的分化**：会有更多企业跟进微软的「隔离运行」建议，把 OpenClaw 放进沙箱还是直接禁用，会成为企业安全政策的新争议点。

4. **监管层面的反应**：中国工信部已经对 OpenClaw 发出安全警告，其他国家的监管机构是否跟进？

OpenClaw 的 347K 星是一个工程胜利，但它也是一张安全欠条。欠债不可怕，可怕的是债主还没来敲门，而你还以为自己在读表扬信。

---

参考资料：

1. OpenClaw GitHub Stars 历史数据 — https://github.com/openclaw
2. OpenClaw 347K stars 统计报告 — https://openclawvps.io/blog/openclaw-statistics
3. 微软安全博客：Running OpenClaw Safely — https://www.microsoft.com/en-us/security/blog/2026/02/19/running-openclaw-safely-identity-isolation-runtime-risk/
4. CVE-2026-25253 等漏洞列表 — https://www.sangfor.com/blog/cybersecurity/openclaw-ai-agent-security-risks-2026
5. Google Project Mariner 关闭 — https://www.theverge.com/tech/925559/google-project-mariner-shut-down
6. OpenClaw 安全风险：CISO 指南 — https://www.techtarget.com/searchsecurity/tip/The-OpenClaw-security-risks-every-CISO-needs-to-know
7. DigitalOcean：OpenClaw 7 大安全挑战 — https://www.digitalocean.com/resources/articles/openclaw-security-challenges
