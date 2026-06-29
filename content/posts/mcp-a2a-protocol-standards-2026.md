+++
date = '2026-06-29T07:30:00+08:00'
draft = false
title = 'MCP 97M 下载之后的岔路口：为什么 A2A 无法取代 MCP'
description = "Anthropic 将 MCP 捐给 Linux 基金会后，Google 和 OpenAI 联手推 A2A 协议。两套标准同台竞技，开发者该押注哪一个？本文梳理事实，厘清技术定位差异与生态博弈。"
tags = ["AI", "AI观察"]
categories = ["AI观察"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/29/img_1782691308_1.jpeg", alt = "电路板分叉路口，蓝线通向工具图标，琥珀线通向机器人图标" }
+++

2026年3月，Anthropic 将自己发起的 Model Context Protocol（MCP）捐给了新成立的 Agentic AI Foundation（AAIF）——一个隶属于 Linux Foundation 的定向基金，共同发起方还包括 Block、OpenAI，以及 Google、Microsoft、AWS、Cloudflare、Bloomberg 等巨头。发布时公布了一组让整个行业无法忽视的数字：**97M+ 月度 SDK 下载量，16 个月，从 0 到史上最快扩散的 AI 基础设施标准**。

然而这并不是终局。Google Cloud Next 2026 期间，A2A（Agent-to-Agent）协议高调亮相，OpenAI 同步为其背书。这两套协议在叙事上高度重叠——都是"让 AI  agent 互相连接"——以至于大量开发者的第一反应是：**它们不是竞争关系吗？我该押注哪一个？**

这个问题背后，其实混淆了两个不同维度的技术问题。

## 两个协议，解决的是不同层次的连接问题

### MCP：AI 与工具的 USB 接口

MCP 的核心抽象是**将外部工具、数据源、服务标准化为一个可以被任意 LLM 直接调用的接口层**。开发者写一个 MCP server，任何支持 MCP 的模型（Claude、GPT-4o、Gemini、Llama 等）都可以连接它。打个比方：MCP 就像是 USB 接口——键盘、鼠标、硬盘都通过 USB 连接 PC，而不是每台 PC 需要为每个设备写专门的驱动。

MCP 解决的是 **"模型 → 工具"** 的连接问题。

### A2A：Agent 之间的「外交语言」

A2A（Agent-to-Agent Protocol）则是另一个层次的抽象。当一个 AI agent 需要与另一个 AI agent 协作时——比如你的日历 agent 需要调用你同事的任务分配 agent——A2A 定义了它们如何互相发现、如何交换任务上下文、如何通知对方状态变更。

A2A 解决的是 **"agent ↔ agent"** 的连接问题。

这两个问题本质上是正交的。你可以想象成：MCP 是 PC 内部的扩展总线，A2A 是公司内部不同部门之间的 OA 系统。它们解决不同问题，并不直接竞争。

## 为什么 A2A 不能取代 MCP

Google 和 OpenAI 联手推 A2A，表面上像是「围剿 Anthropic」，但只要稍微深究技术细节就会发现：**A2A 的设计从第一天起就依赖 MCP 作为其工具调用层**。

Google Cloud Next 的演示里，A2A agent 调用外部工具走的是 MCP server。Anthropic 的 MCP 实现里，也早就支持了 agent 间的上下文传递。换句话说，现在看到的 A2A 实现，几乎都是 **「A2A 做协调层 + MCP 做执行层」** 的组合。

这背后有一个很现实的原因：如果 A2A agent 需要真正执行代码、查数据库、调用 API，它需要一个比"协商任务分配"更底层的协议来做这些操作。MCP 恰恰填补了这个空白。

所以更准确的技术图景是：

> **A2A + MCP = 完整的 agent 通信栈**
> - A2A：协调层（任务分发、状态同步、协作流程）
> - MCP：执行层（代码执行、数据获取、工具调用）

一个只支持 A2A 的 agent 本质上是个「只吵架不动手」的协调者。这也是为什么，即便 Google 在大力推 A2A，它的 Agent Development Kit（ADK）里依然内建了 MCP 支持。

## 97M 下载量背后的生态锁定风险

MCP 97M 月度 SDK 下载是个什么概念？这个数字意味着全球大多数 AI 项目在立项时已经把「有没有 MCP server」列为技术选型的默认检查项——就像 2015 年前后 Docker 容器的扩散速度。

但这个优势也带来了一个被低估的风险：**生态锁定**。

当 10,000 个 MCP server 已经上线运行，当主流 IDE（VS Code）和主流开发工具（GitHub Copilot、Cursor）都内建了 MCP 支持，切换协议的成本就变得极高。这和当年 USB 被广泛采用后，厂商很难再推私有接口的故事如出一辙。

OpenAI 和 Google 之所以在推 A2A 而不是「取代 MCP」，正是因为取代成本已经高到不可接受。**它们的选择是：在 MCP 之上叠一层 A2A 来定义协调标准，从而在不完全颠覆现有生态的前提下抢到定义权**。

这才是当前标准竞争的实质：不是 MCP vs A2A，而是**谁在 MCP 之上定义的协调层更能捕获价值**。

## 6 月的两件大事让这个叙事更清晰

### GPT-5.6 Sol 的政府门控发布

6月26日，OpenAI 发布 GPT-5.6 Sol——迄今为止最强的网络攻击能力模型（网络攻击基准测试 96.7% 通过率）。但这一次，OpenAI 史无前例地将首发权交给了美国政府——约 20 个政府审批的合作伙伴优先使用，普通开发者和消费者暂时无法访问。

与此同时，Anthropic 的 Mythos 5（更强版本的网络安全模型）也在 6 月获得了美国商务部的有限授权——约 100 个美国关键基础设施运营商和网络安全公司获得了访问权限。

这两件事放在一起，揭示了一个正在形成的新范式：**前沿 AI 模型的发布，正在从「先公开再治理」转向「先治理再公开」**。模型门控成为常态，协议层的重要性只会更高——因为门控本质上是通过协议来执行的。

### MCP 捐给 Linux 基金会：标准的中立化

Anthropic 将 MCP 捐给 AAIF（Linux Foundation 旗下），而不是自己继续主导，这个动作的战略意图被很多人低估了。

Anthropic 的算盘是：**如果 MCP 继续由 Anthropic 直接控制，Google 和 OpenAI 就有动机联合推一个替代品来分摊风险**。但当 MCP 进入 Linux Foundation 生态，它就变成了一个多方共同管理的公共财产——任何一家单独「fork」这个协议的正当性都会被大幅削弱。

这和 Linux、Kubernetes 早期的开源治理逻辑完全一致。Red Hat 把 Kubernetes 捐给 CNCF，不是放弃控制权，而是**让竞争在更高层次展开**——在 Kubernetes 之上做商业化，而不是在协议层重新发明轮子。

## 开发者该怎么应对

当前阶段，**押注 MCP 是更稳妥的选择**。原因有三：

**第一，MCP 的先发优势已经形成生态壁垒。** 97M 下载量背后是数千个已经上线的 MCP server，这些 server 不会因为 A2A 的出现而自动消失。

**第二，A2A 的实现高度依赖 MCP。** 你如果已经在用 MCP 构建工具层，升级到 A2A 协作层几乎是顺滑的增量。但如果你押注 A2A 而忽略 MCP，你会发现自己的 agent 有协调能力却没有执行能力。

**第三，标准收敛需要时间，现在学两套不如精通一套。** MCP 的 SDK 文档、community server 生态、调试工具链都已经相对成熟。A2A 还处于早期规范阶段，生产级最佳实践尚未凝固。

更务实的建议是：**以 MCP 为执行层构建工具，以 A2A 为协调层规划多 agent 架构**。两者不是二选一，而是互补的。

## 接下来看什么

以下几个信号可以帮助你判断这个领域接下来的走向：

1. **AAIF 下半年是否会有第一个独立的 protocol 版本发布**——这会显示多方治理是否真的能高效运作，还是会陷入标准僵局

2. **Google 的 ADK 和 OpenAI 的 Agent Framework 是否会在 A2A 实现上出现分化**——如果两者实现不兼容，「标准」二字名存实亡

3. **MCP server registry 的商业化进展**——Anthropic 开放了社区 registry，当商业公司开始为优质 MCP server 付费，生态会进入正向飞轮

4. **非英语市场的 MCP 采用速度**——当前 97M 下载量的地理分布数据尚未公开，中国、中东、东南亚开发者的选择会决定下一个 100M 的方向

MCP 已经不是「Anthropic 的协议」了——它现在是整个 AI 行业的公共基础设施。在这个阶段，与其争论谁取代谁，不如先确保自己的 agent 能在两张网上都正常工作。

---

## 参考资料

- [Anthropic 官方公告：Donating the Model Context Protocol and establishing the Agentic AI Foundation](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation)
- [Linux Foundation 官方公告：Agentic AI Foundation 成立](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation)
- [OpenAI 官方：A2A 协议与 Agentic AI Foundation](https://openai.com/index/agentic-ai-foundation/)
- [The Verge：MCP Dev Summit North America 2026 报道](https://en.wikipedia.org/wiki/Model_Context_Protocol)
- [TechCrunch：OpenAI limits GPT-5.6 rollout after government request (June 26, 2026)](https://techcrunch.com/2026/06/26/openai-limits-gpt-5-6-rollout-after-government-request-says-restrictions-shouldnt-be-the-norm/)
- [VentureBeat：OpenAI unveils GPT-5.6 Sol, Terra and Luna — limited preview](https://venturebeat.com/technology/openai-unveils-gpt-5-6-sol-terra-and-luna-models-but-only-accessible-to-limited-preview-partners-for-now-per-us-gov)
- [Fortune：Anthropic's Mythos 5 AI model cleared by U.S. for wider use (June 27, 2026)](https://fortune.com/2026/06/27/anthropic-mythos-5-ai-model-us-commerce-department-clearance-fable/)
- [The New Stack：Anthropic Donates MCP to Agentic AI Foundation (March 2026)](https://thenewstack.io/anthropic-donates-the-mcp-protocol-to-the-agentic-ai-foundation/)
