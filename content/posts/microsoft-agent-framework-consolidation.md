+++
date = '2026-07-30T07:31:59+08:00'
draft = false
title = 'AutoGen 死了，Semantic Kernel 也死了：微软 Agent 框架统一事件的技术还原'
description = "2026年4月，微软将AutoGen和Semantic Kernel合并为Microsoft Agent Framework 1.0。本文还原这场合并的技术细节、对开发者生态的影响，以及协议战争新格局。"
tags = ["AI", "AI观察", "微软", "框架", "AI Agent"]
categories = ["AI观察"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/07/30/img_1785369704_1.jpeg", alt = "two robots shaking hands protocol war digital art" }
+++

2026年4月3日，微软正式发布 Microsoft Agent Framework 1.0，将两个曾被广泛使用的 AI 编排库——AutoGen 和 Semantic Kernel——合二为一。这不是一次普通的产品迭代，而是微软内部两条独立技术路线的最终收敛。本文还原这次合并的技术细节，以及它对整个 AI Agent 生态的深层影响。

## 两条路线的起源与分歧

理解这次合并，需要回到两条路线的最初设计意图。

**Semantic Kernel** 起源于微软的"企业 AI"战略，定位是轻量级编排层。它的核心抽象是 Kernel——一个插件化的 AI 能力容器，开发者通过它连接 LLM 与企业内部工具。Semantic Kernel 的设计哲学偏向"工程友好"：稳定的 API、完整的 .NET/Python 双语言支持、与 Azure AI Services 的深度集成。选用它的开发者，通常是希望在企业级应用中快速落地 AI 能力的团队。

**AutoGen** 来自微软研究院，定位是"多智能体协作研究框架"。它的核心是多智能体对话范式——多个 LLM 代理相互通信、协商、分工，完成复杂任务。AutoGen 更像一个实验平台，设计上鼓励研究者探索各种 agent 协作模式，对生产环境的稳定性要求相对宽松。

两条路线服务的是不同场景，却面向同一个开发者群体。这导致了一个尴尬局面：同一拨人要在两个框架之间做选择，而两者的文档、社区和最佳实践几乎完全割裂。

## Agent Framework 1.0 做了什么

2026年4月3日GA的 Microsoft Agent Framework 1.0，本质上是 Semantic Kernel 作为底层基础设施，承载 AutoGen 的多智能体编排能力。

具体来说，Kernel、插件模型和连接器系统保持不变——这是 Semantic Kernel 多年积累的企业级稳定性基础。AutoGen 的多智能体对话和编排概念，以图工作流引擎的形式重建在 Kernel 之上。

两条技术路线的核心价值都得到了保留：Semantic Kernel 的稳定 API 和中间件生态，AutoGen 的多代理协作模式。开发者不再需要做非此即彼的选择。

对于已经在使用这两个框架的开发者，迁移路径相对平滑。Semantic Kernel 的会话状态管理、中间件和遥测模式可以直接沿用；AutoGen 的多智能体编排模式则直接映射到新的图引擎。对微软而言，这是一个务实的决策——不是重新发明，而是重新排列组合。

## 一个被忽视的信号：MCP + A2A 双协议支持

在技术合并的喧嚣中，有一个细节值得单独拿出来说：Microsoft Agent Framework 1.0 是第一个同时支持 MCP（Model Context Protocol）和 A2A（Agent2Agent Protocol）的主流框架。

这不是偶然的设计选择，而是微软在协议层押注的两条线。

MCP，解决的是 agent 与外部工具/系统之间的连接问题——相当于"agent 的 USB 接口"。A2A，解决的是 agent 与 agent 之间的协作问题——相当于"agent 的网络协议"。两个协议处于不同的抽象层级，服务不同的需求，但合在一起，构成了完整的多智能体系统通信基础设施。

值得注意的是，在 2026年6月的框架对比中，LangGraph 和 AutoGen（合并前）都不原生支持这两个协议，CrewAI 仅支持 A2A，而 OpenAgents 是当时唯一同时支持 MCP 和 A2A 的框架。微软的这次更新，让 Agent Framework 1.0 进入了这个稀缺名单。

## 这场合并真正意味着什么

表面上，这是一个"减少开发者选择困难"的故事。AutoGen 和 Semantic Kernel 的分裂，本来就是微软内部路线之争的产物，开发者承担了磨合成本。但更深一层看，这是微软在 AI Agent 战争中的战略定位调整。

当 OpenAI、Anthropic、Google 各自推出自己的 Agent 框架时，微软没有选择继续维护两条独立路线来内卷，而是选择合并出击。Agent Framework 1.0 的定位很明确：不做最酷的那个，但做企业用起来最没负担的那个。

对整个生态而言，这加剧了框架层面的整合趋势。一个直接的数据佐证：在 2026年7月的综合对比中，Microsoft Agent Framework 1.0 与 LangGraph、CrewAI 一起被视为"生产就绪度最高"的第一梯队，而早期的 AutoGen 则在发布后不到两年便进入维护状态。

## 接下来看什么

**一、开发者迁移的实际体验。** 官方文档说平滑，但真实迁移中有没有隐藏的 Breaking Change？AutoGen 一些实验性 API 在新框架中的对等物是什么？这些问题需要等第一批吃螃蟹的开发者踩完坑才有答案。

**二、框架格局是否会引发协议格局的洗牌。** 当 MCP 和 A2A 被更多主流框架采用，两个协议之间的竞争关系会怎么演化——是最终走向融合，还是像 REST/GraphQL 一样长期共存？

**三、微软能否凭借这次整合在 Agent 框架战中真正超车。** 单纯的技术合并不等于生态胜出。LangGraph 背后有 LangChain 积累多年的社区和文档优势，OpenAgents 有 MCP+A2A 的先发优势。微软能否把"最完整的协议支持"转化为开发者心智，还要看接下来的社区运营和 GA 质量。

---

## 参考资料

- Microsoft Agent Framework 1.0 GA 发布公告（2026年4月3日）：https://www.developersdigest.tech/blog/microsoft-agent-framework-developer-guide-2026
- AutoGen 与 Semantic Kernel 合并技术分析：https://alexbevi.com/blog/2026/06/18/two-lineages-one-framework-how-autogen-and-semantic-kernel-became-the-microsoft-agent-framework/
- AI Agent 框架 2026 综合对比（框架层）：https://pecollective.com/blog/ai-agent-frameworks-compared/
- A2A vs MCP 协议层级分析：https://atlan.com/know/mcp/mcp-vs-a2a-protocol/
- OpenAgents 框架 MCP+A2A 双协议支持：https://openagents.org/blog/posts/2026-02-23-open-source-ai-agent-frameworks-compared
