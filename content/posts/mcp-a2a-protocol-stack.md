+++
date = '2026-06-15T08:00:00+08:00'
draft = false
title = 'MCP 和 A2A 协议栈正在吞噬软件世界：一次来自 2026 年的现状报告'
description = "Model Context Protocol 月均 97M SDK 下载、5800+ 公共服务器；A2A 协议一年内召集 150+ 成员——这不是标准战胜利，这是协议层收敛的开始。本文拆解两层协议的分工逻辑、技术现实与对开发者生态的深远影响。"
tags = ["编程", "技术", "AI", "协议"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/15/img_1781481720_1.jpeg", alt = "Two AI agents connecting through glowing protocol bridges, neural network nodes" }
+++

2025 年 11 月，Anthropic 开源了 MCP（Model Context Protocol），当时业界普遍把它当成又一个"连接 AI 和工具"的小众标准。2026 年 6 月，这个判断被彻底推翻——MCP 月均 SDK 下载突破 9700 万次，公共服务器超过 5800 个，OpenAI、Google、Microsoft、Amazon 全部在 13 个月内完成了原生支持。同月，Google 在 Cloud Next '26 宣布 A2A 协议（Agent-to-Agent Protocol）已在 150 家企业的生产环境跑通，Microsoft、AWS、Salesforce、SAP 全部在列。这不是一场标准战争——这是协议层收敛的速度已经超过了大多数人的感知阈值。

## 两个协议，两层问题

在深入数据之前，先把技术边界说清楚。MCP 和 A2A 解决的是不同层次的问题，硬要把它们放在同一场比赛里比较，只会得出错误的结论。

**MCP 定义的是 agent 如何调用工具**：数据库、API、文件系统、第三方服务——MCP 负责让 AI 模型"够得着"外部世界。你可以把它理解成 USB 协议在 AI 时代的对应物——一个通用接口层，让任何支持 MCP 的客户端（Claude Desktop、Cursor、ChatGPT 桌面端、OpenAI Agents SDK）能插进任何支持 MCP 的服务器。

**A2A 定义的是 agent 如何和其他 agent 协作**：任务委派、状态共享、协作流程——A2A 让自主的 AI agent 之间能够对话，而不需要人类在中间做翻译。这是机器对机器的通信协议，不是给人类发消息的聊天接口。

这两个协议在架构上是正交的。MCP 处理的是 agent 到工具的连接，A2A 处理的是 agent 到 agent 的协作。它们不是竞争关系，而是互补关系——一个成熟的多 agent 系统会同时依赖两者。

## MCP：已经赢了工具层，但代价是复杂性

先看数字。根据 Anthropic 官方博客披露（2026 年 5 月），MCP 在不到两年的时间内实现了：

- **Python + TypeScript 月均 SDK 下载：9700 万次**（从发布时的 10 万次增长而来）
- **公共 MCP 服务器：超过 5800 个**（官方 registry），实际估算（含社区私有服务器）超过 13000 个
- **企业用户**：Anthropic 自己在 2025 年 12 月将 MCP 捐赠给 Linux Foundation 旗下的 Agentic AI Foundation，OpenAI 和 Block 作为 co-founder 加入，AWS、Google、Microsoft、Cloudflare、Bloomberg 作为 supporting members 在列

2026 年 3 月发布的 MCP 2026 路线图列出了四个优先级：Transport Scalability（让 Streamable HTTP 能无状态扩展，支持负载均衡器）、Agent Communication（深化 agent 间通信）、Governance Maturation（SEP-1302 正式化了 Working Groups，SEP-2085 建立了修正程序）、Enterprise Readiness（企业级安全和治理）。

这四个方向的潜台词是同一个：**MCP 已经从"可以用"进入了"必须企业级"的阶段**。协议本身通过了验证期，现在要解决的是规模化部署的工程问题。

但这里有一个反直觉的点值得单独说：**MCP 的成功恰恰带来了复杂性问题**。当 5800+ 服务器存在时，服务发现、版本管理、认证鉴权成为了真实的工程挑战。路线图中"Transport Scalability"被列为第一优先级，恰好说明了这一点——不是协议本身有问题，而是协议太好用了，导致规模暴涨后基础设施跟不上。

## A2A：一年时间，从 0 到生产级

A2A 的故事更具戏剧性。Google 在 2025 年 4 月的 Cloud Next 大会上宣布了 A2A，6 月捐赠给 Linux Foundation。到 2026 年 4 月，A2A 成立一周年时已经交出了这样的成绩单：

- **150+ 组织支持**：Google、Microsoft、AWS、Salesforce、SAP、ServiceNow、Workday、IBM、Cisco、PayPal、MongoDB、Cohere 全在名单里
- **生产级落地**：Microsoft、AWS、Salesforce、SAP、ServiceNow 均已有 active production deployments
- **协议版本**：v1.0 已发布，带 Signed Agent Cards；AP2 作为正式扩展正在推进

关键的是，A2A 的定位和 MCP 完全不同——它解决的是协作层的问题。想象一个典型场景：一个数据分析 agent 收到了用户请求，发现需要先调用搜索 agent 查资料，再调用代码执行 agent 做计算，最后调用写作 agent 生成报告。在 A2A 出现之前，这三步协作需要开发者自己写胶水代码；有了 A2A，agent 之间可以用标准协议交换任务状态和结果，而不需要每对 agent 组合都写定制代码。

**A2A 真正的价值不是"让 agent 能通信"，而是"让 agent 协作规模化"**。当协作接口标准化后，专业化 agent 的生态才真正成为可能——一个专门做数据检索的 agent 可以被任何支持 A2A 的系统调用，而不需要重新实现适配层。

## Linux Foundation 的角色：意外但合理

2026 年最被低估的故事之一，是 Linux Foundation 同时托管了 MCP、A2A 和 ACP（IBM/AGNTCY 的 Agent Communication Protocol）三套协议。

这不是 Linux Foundation 主动争取来的。是 Anthropic 捐 MCP，Google 捐 A2A，IBM 捐 ACP——三个主要玩家在不到半年的时间内，不约而同地选择了 Linux Foundation 作为 governance 托管方。原因是相同的：治理需要中立性，标准需要可信度，而 Linux Foundation 是目前少数几个能让 Google、Microsoft、Amazon、Anthropic 同时信任的机构。

**这标志着 AI 协议治理正在走向传统软件基金会的成熟路径**。不是某一家公司控制标准，而是通过 SEP（Specification Amendment Process）和 Working Groups 实现社区治理。SEP-1302 正式化 Working Groups 和 Interest Groups，SEP-2085 建立修正和继承程序——这两个提案的存在意味着协议演进有了透明机制，而不是靠某家公司拍脑袋决定。

## 对开发者的实际影响：不是学不学，是什么时候用

写到这里，文章已经够长，但我想单独拉出一个角度，因为这是大多数技术博主会忽略的点：**这个协议收敛对开发者而言不是"要不要学新东西"的问题，而是"架构决策正在被协议层接管"**。

当 MCP 成为事实标准后，你的 AI 应用接入外部工具的方式就被锁定在 MCP 生态里了——不是因为 MCP 有多好，而是因为所有主流模型提供商都支持它，你没有别的选择。A2A 同理：当 150 家企业在生产环境里用 A2A 做 agent 协作时，未来的多 agent 系统会默认支持 A2A——不是因为它是最好的设计，而是因为它是"大家都用"的那个。

这意味着开发者的角色从"实现协作逻辑"变成了"选择和配置协议"——更准确地说，是从"写代码"转向了"搭架构"。协议层收敛降低了接入成本，但提高了架构选择的权重。当工具层和协作层都被标准协议覆盖后，应用层的差异化就变成了：在哪里组合这些协议，以及如何设计 prompt 和 agent 的职责边界。

**这不是危言耸听，这是每波基础设施标准化都会发生的事**。TCP/IP 标准化后，应用层开发者不需要写底层网络代码；HTTP 标准化后，Web 应用开发者不需要操心 socket 编程。MCP + A2A 正在做的事，是把这个逻辑从网络层搬到 AI agent 层。

## 接下来看什么

- **MCP 路线图 Q3/Q4 2026**：Transport Scalability 的实现进度——Streamable HTTP 无状态扩展是否真的能解决企业级负载均衡问题，这是判断 MCP 能不能撑住大规模部署的关键
- **A2A AP2 扩展**：Signed Agent Cards + AP2 的组合是否能让 agent 身份认证和授权真正落地，这是企业采用的核心门槛
- **Agentic AI Foundation 的独立运营能力**：Linux Foundation 托管三个竞争性协议的经验能否复制到 AI 领域，治理透明度和演进速度之间的平衡点在哪里
- **中国区的 MCP/A2A 采用情况**：腾讯云、阿里云、百度智能云是否跟进，有哪些本土化实现值得关注

## 参考资料

- [MCP 官方路线图 2026](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/)
- [Anthropic 宣布将 MCP 捐赠给 Linux Foundation](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation)
- [A2A 协议一周年成绩单 - Linux Foundation](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year)
- [MCP vs A2A: 2026 AI Agent 协议完整指南 - DEV Community](https://dev.to/pockit_tools/mcp-vs-a2a-the-complete-guide-to-ai-agent-protocols-in-2026-30li)
- [MCP SDK 下载数据 - Knak](https://knak.com/blog/mcp-adoption-in-2026-what-marketers-need-to-know/)
- [A2A 协议官网](https://a2a-protocol.org/latest/)
- [MCP 服务器生态 2026 统计 - Digital Applied](https://www.digitalapplied.com/blog/mcp-adoption-statistics-2026-model-context-protocol)
- [A2A vs MCP 选择指南 - onereach.ai](https://onereach.ai/blog/guide-choosing-mcp-vs-a2a-protocols/)
- [MCP 2026 企业采纳完整指南 - DEV Community](https://dev.to/x4nent/complete-guide-to-mcp-model-context-protocol-in-2026-architecture-implementation-and-4a11)
- [AI Agent 协议栈：MCP、A2A 与下一步 - TURION.AI](https://turion.ai/blog/ai-agent-protocol-stack-2026/)