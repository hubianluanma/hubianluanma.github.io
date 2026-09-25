+++
date = '2026-09-25T07:32:28+08:00'
draft = false
title = 'Google ADK 2.0 放弃层级制：图执行引擎才是 AI Agent 的未来架构'
description = "ADK 2.0 从 top-down hierarchy 转向 graph-based execution engine，与 Microsoft Agent Framework 1.0、Anthropic A2A 协议形成三方共振，AI Agent 框架层正式收敛。"
tags = ["AI", "AI观察"]
categories = ["AI观察"]
author = "Spiral"
+++

2026年9月24日，Google 正式发布 Go 语言版 Agent Development Kit（ADK）2.0。同一天，ADK TypeScript 2.0 已 GA 满一个月，Python 版 ADK 2.0 已跑满一个 Q。

这不是一次普通的版本迭代。ADK 2.0 的核心变化，是把执行模型从 **top-down hierarchy（自上而下层级制）彻底切换为 graph-based execution engine（图执行引擎）**。这意味着 Google 承认了自己在 AI Agent 架构方向上的路径错误，并用行动投票纠偏。

与此同时，Microsoft Agent Framework 1.0（2026年4月）已支持原生 MCP 和 A2A；Anthropic 主导的 A2A（Agent-to-Agent）协议也在 2026 年逐步落地。三方在不到半年内，完成了从各玩各的到架构收敛的过程。

这篇文章说清楚一件事：**图执行引擎为什么是正确方向，以及这次收敛对开发者意味着什么**。

## 从「指挥官模式」到「状态机模式」

理解这个转变，先要搞清楚旧模式错在哪。

ADK 1.x 的执行模型是典型的层级制：有一个顶层的 Orchestrator Agent，它负责规划任务，然后把子任务分发给下层的 Worker Agent，等待结果，汇总，再往下发。类似金字塔，命令从塔尖一层层往下传。

这个模式的致命问题在于：**一旦某个中间步骤失败，整个链条要从头重来**。没有状态记忆，没有分支回滚，没有条件跳转。复杂一点的工作流，比如"先查数据库、发现数据缺失就改查 API、API 也失败了再查缓存"，在层级制下要写大量 if-else 分支，代码耦合到没法维护。

图执行引擎的思路完全不同：**每个节点是一个有状态的执行单元，边是状态转换条件**。你可以理解为把工作流写成一张有向图，节点执行完告诉图引擎"我现在在状态B"，图引擎根据预设的边决定下一步去哪。如果分支B失败了，图引擎知道怎么回退到状态A走分支C，而不是让整个金字塔崩塌。

LangGraph（LangChain 的图执行库）在 2025 年最早把这个模型做成熟，2026 年被整个行业快速复制。

## Google ADK 2.0 的三个关键变化

ADK 2.0 的官方文档把新架构描述为 "mixing deterministic code with adaptive AI"——用确定性代码混入自适应 AI。这个说法背后是三个实质变化：

**第一，执行引擎图化。** 不再是 Orchestrator 统领一切的层级调用，而是多个 Agent 形成一张有向图。每个 Agent 有明确的输入状态和输出状态，转换条件由代码显式声明。这让工作流的失败恢复、循环检测、条件分支全部变成图遍历问题，有成熟的算法可以处理。

**第二，managed MCP servers 内置。** ADK 2.0 原生支持 MCP（Model Context Protocol）服务器，且是 managed 模式——开发者不需要自己运维 MCP 服务端，MCP 服务器生命周期由 ADK 框架管理。这降低了 MCP 的接入门槛，也是 Google 对 Anthropic 主导的 MCP 协议的明确站队。

**第三，多语言 SDK 同时到位。** Python、TypeScript（8月21日 GA）、Go（9月24日）、Kotlin（1.0）四个语言版本，Kotlin 宣布与 Python 版功能对齐。这意味着企业内部用不同语言构建的 Agent 可以互相通信——也是 A2A 协议的一次实践。

## Microsoft Agent Framework 1.0：企业级的对标

Google 的动作不是孤立的。

2026年4月3日，Microsoft 发布 Agent Framework 1.0，定位是"生产级多 Agent 协调框架"。核心特性：stable APIs、**原生 MCP 支持**、**A2A 协议支持**、长期支持承诺。

这是 Microsoft 第一次把 Agent 框架当成正式产品来维护，而不是实验性项目。1.0 意味着 API 不会在 minor version 里 break，这对企业用户是生死线级别的承诺。

有趣的是 Microsoft 的文档里明确提到：框架解决的是"Agent 如何规划、如何调用工具、如何协调"三个问题。这和 ADK 2.0 的设计目标几乎一样。两家公司在同一时间，得出了相同的架构结论。

## A2A 协议：真正的互联互通

如果只有图执行引擎，各家框架还是各玩各的。真正让生态收敛的，是 MCP 和 A2A 两个协议。

**MCP（Model Context Protocol）**解决的是 Agent 与工具/数据源的通信问题——Agent 调用什么工具、工具返回什么格式，有一个标准化的约定。Anthropic 在 2024 年底发起，现在 Google、Microsoft、OpenAI 全部采纳。

**A2A（Agent-to-Agent）**解决的是 Agent 之间的通信问题——两个 Agent 之间如何发现彼此、如何协商任务、如何传递上下文。Anthropic 主导，Google ADK 2.0 明确支持，Microsoft Agent Framework 1.0 原生集成。

这两个协议组合起来，构成了 AI Agent 的"网络层+传输层"。就像 HTTP+TCP 让互联网应用互联，MCP+A2A 让不同框架开发的 Agent 可以互相操作。

2026年9月这个节点，三个最大的 AI Agent 框架玩家（Google、Microsoft、Anthropic 生态）全部站到了同一套协议栈上。这是 AI Agent 发展史上第一次出现标准协议的实质性落地。

## 反共识观点：协议收敛不等于生态健康

标准协议出现，通常意味着生态成熟。但 AI Agent 领域这次收敛有一个被忽视的问题：**协议收敛掩盖了执行层的碎片化**。

MCP 和 A2A 只定义了"怎么说"，不定义"说什么"。不同框架对"一个任务的输入状态长什么样""工具调用的返回格式是什么"理解并不一致。LangGraph 用的是自己的状态对象，ADK 2.0 用的是 agent.Context API，Microsoft 用的是另一套图描述格式。三家表面上都说支持 A2A，实际上 A2A 消息的语义解析可能不兼容。

更根本的问题是：图执行引擎本身还没有统一标准。LangGraph 的图是代码优先（用 Python 定义图结构），ADK 2.0 的图是声明式优先（用 YAML 或 JSON 描述工作流），Microsoft 用的是自己的 DSL。开发者选了一个框架，就绑定了它的图描述语言和状态管理方式，想迁移到另一个框架成本极高。

所以这轮"收敛"的真实含义是：**通信层收敛了，执行层还在混战**。这对短期互操作性是好事，对长期生态健康是个隐患。

## 开发者现在该怎么选

如果你是企业开发者，现在选框架的逻辑很简单：

**选 ADK 2.0**的理由是 Google 的多语言 SDK 策略——Python/TypeScript/Go/Kotlin 四个语言可以共存，适合内部异构团队。缺点是 ADK 2.0 太新，社区还在生长期，生产案例少。

**选 Microsoft Agent Framework**的理由是 1.0 意味着稳定，企业内部合规审查容易通过。缺点是绑定 Azure 云，迁移成本高。

**选 LangGraph**的理由是生态最成熟，文档质量最高，社区案例最多。缺点是纯 Python，在多语言场景下是孤岛。

一个务实策略：**先用 LangGraph 跑通核心逻辑，等 ADK 2.0 的生产案例多起来再做迁移决策**。协议层 MCP+A2A 是必须跟进的，但框架层选型要等市场验证。

## 接下来看什么

AI Agent 框架的竞争还没定局，接下来三个观察点：

**1. MCP 注册表的生态之争。** MCP 协议支持第三方扩展，现在有官方 MCP Servers、OpenAI MCP Servers、Google MCP Servers 三套注册表，谁的生态越丰富，谁的 Agent 就越强。这个竞争会在 2026 Q4 加速。

**2. A2A 的安全边界。** A2A 让两个 Agent 可以互相传递敏感上下文，跨企业的 Agent 协作场景下，谁来保证数据不泄漏？这个问题的答案会影响 A2A 在企业场景的落地速度。

**3. 图执行引擎的性能基准。** 现在的图执行引擎 demo 都很漂亮，但生产环境下的延迟、内存占用、并发处理能力没有公开数据。2026 年底到 2027 年初，会有第一批大规模生产部署的基准报告出来，届时才能判断哪个框架真的经得住压力。

---

## 参考资料

- Google ADK 2.0 官方文档：https://adk.dev/
- ADK TypeScript 2.0 GA 公告（2026年8月21日）：https://adk.dev/2.0/
- Google 发布 Go ADK 2.0（2026年9月24日）：https://developers.googleblog.com/announcing-the-agent-development-kit-for-go-build-powerful-ai-agents-with-your-favorite-languages/
- Google ADK Kotlin 1.0 与 Python 功能对齐：https://www.infoq.com/news/2026/09/google-adk-1-0-released/
- Microsoft Agent Framework 1.0（2026年4月3日）：https://atlan.com/know/ai-agent/microsoft/agent-framework/
- Anthropic A2A 协议概述：https://docs.anthropic.com/en/docs/agentic-ai-concepts
- LangGraph v1.2.7 发布：https://www.shakudo.io/blog/top-9-ai-agent-frameworks
