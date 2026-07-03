+++
date = '2026-07-03T07:32:20+08:00'
draft = false
title = 'MCP 终于去掉"粘性会话"：这次不是修 bug，是重写契约'
description = "MCP 2026-07-28 版本让协议层彻底告别状态依赖。背后推力不是设计失误，而是 2024 年那个让所有人大跌眼镜的采用速度——以及它撞上 Kubernetes 时暴露的规模化裂缝。"
tags = ["编程", "技术", "AI", "协议"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/07/03/img_1783037035_1.jpeg", alt = "MCP stateless protocol illustration" }
+++

Anthropic 2024 年底推出 MCP（Model Context Protocol）时，定位是"让 AI 模型安全可控地调用工具"的标准。没人预料到它会成为 2025-2026 年 AI Agent 领域普及最快的协议——也没有人充分预估它会在 Kubernetes 容器化部署时遇到那个致命的设计裂缝：协议层依赖会话状态。

7 月 28 日正式发布的 MCP 2026-07-28 候选规范（RC 已开放），正在用六个 SEP（Specification Enhancement Proposals）把这个裂缝焊死。本文拆解这次改动的真实工程含义，以及它对正在把 MCP 部署到生产环境的人意味着什么。

## 发生了什么：MCP 从"有状态会话"变成"无状态请求"

MCP 最早的传输层设计借鉴了 SSE（Server-Sent Events）：客户端连接后维持一个长连接，服务端通过这个通道主动推送消息。这个模型对本地开发工具友好——你启动一个 MCP Server，`stdio` 直接连上，不需要考虑网络拓扑。

但一旦把这个模式搬到云端，进了容器编排系统，问题就来了。

**具体故障链是这样的**：

1. 客户端通过负载均衡连接到了 Pod A，建立了 SSE 会话
2. 下一秒 Kubernetes 做了一次滚动部署，或者 Horizontal Pod Autoscaler 新增了一个 Pod B
3. 下一个请求被路由到 Pod B——但 B 没有这个会话的内存状态
4. Pod B 返回 `404 session not found`，客户端崩溃

这不是边缘案例。GitHub Issue #153（mcp-use）、#520（python-sdk）以及大量 Medium 技术博客都记录了完全相同的故障模式。**会话亲和性（sticky session）**是当时的标准解法：让负载均衡器把所有来自同一客户端的请求都打到同一个 Pod。但 sticky session 本身又带来了新问题——单 Pod 热点、无法横向扩容、部署窗口期内请求丢失。

MCP 2026-07-28 的答案不是打补丁，是**重新设计协议核心**：把状态从协议层剔除，交给应用层处理。

## 六个 SEP 在做什么

官方用"stateless at the protocol layer"描述这次改动，但实现是由六个 SEP 协同完成的：

### SEP-2243：可路由头部（Routable Headers）

Streamable HTTP 传输现在要求 `Mcp-Method` 和 `Mcp-Name` 两个 HTTP 头。负载均衡器、API 网关和限流中间件可以直接根据头部信息做路由或限速，而无需解析 HTTP body。这让 MCP 请求可以在标准 HTTP 代理层（HAP、Caddy、Nginx）之间流通，而不需要应用层感知。

### SEP-2549：可缓存的列表（Cacheable Lists）

`tools/list` 和 `resource-read` 的响应现在携带 `ttlMs` 和 `cacheScope` 字段，参考 HTTP `Cache-Control` 的语义。客户端知道工具列表的保鲜期是多长，在 TTL 有效期内可以直接用缓存，不用为了"列表有没有变"而维持一条 SSE 长连接。这是把网络层缓存机制直接嵌入协议语义。

### SEP-2322：多轮请求模式（Multi Round-Trip Requests）

原来需要服务端主动推送的场景（如工具执行进度、流式输出），现在改为客户端主动轮询的多轮请求模式。消除了服务端发起的回调需求，也就消除了对长连接的依赖。

### 会话初始化握手下放

客户端和服务器之间的 `initialize` 握手不再依赖连接状态。协议版本和客户端信息通过每个请求的 `_meta` 字段传递，服务器从请求本身读取上下文，而非从内存中的会话对象查询。

## 这次不是亡羊补牢——是协议在收敛

有一种观点认为"MCP 搞了两年才发现这个设计缺陷"，把这次改动解读为"救火"。但这个解读并不准确。

MCP 2025 年底的"Future of MCP Transports"博客已经预告了方向。关键在于**时间差**：当 Anthropic 发布 MCP 时，它是作为一个开发者工具协议设计的，用 `stdio` 和本地 SSE 足够；Kubernetes 生产部署是社区在使用过程中发现的场景，超出了原始设计视野。

协议设计者通常会等"实际使用模式"稳定后才锁定语义。**2026-07-28 的改动不是设计失误的修正，而是协议走向成熟的标志——它意味着社区已经把这个协议用到了原始设计者没有预测到的规模，并且协议层选择跟随实践而非对抗实践。**

这个区别很重要：协议层面的 Stateless 设计不是妥协，是收敛到 HTTP 已经被证明两十年的架构模式——应用层在无状态请求之上管理自己的状态（Cookie、Token、Opaque Handle），协议层只负责请求-响应语义。

## 工程影响：谁需要立刻行动

这次发布包含**Breaking Changes**，不是简单的向后兼容升级。以下是分场景的行动清单：

**如果你在用官方 SDK（Python / C#）**

Python SDK v2 和 C# Preview 的 HTTP 传输已默认切换到 2026-07-28 模式。升级后，现有服务器同时支持新旧两个协议版本——新版客户端用 2026-07-28 握手，旧版客户端自动降级到 `2025-11-25` 握手。这层兼容性保护了升级窗口期，但 SDK maintainer 建议对依赖项加上限界：`mcp>=1.27,<2`，防止 stable v2 发布时意外引入破坏性变更。

**如果你自己实现了 MCP 服务器**

需要升级到 beta SDK 并切换到 stateless HTTP 传输模式。关键变化：不再依赖 `session ID` 头，改为从每个请求的 `_meta` 字段读取协议版本和客户端元数据。如果你的服务器目前依赖内存中的会话对象存储状态，需要迁移到外部存储（Redis、数据库）或者接受每次请求都携带完整上下文。

**如果你在 Kubernetes 上跑 MCP 并用了 sticky session**

这是最需要尽快处理的群体。升级到 2026-07-28 后，负载均衡器不再需要 sticky session——你甚至可以把 `session-affinity` 策略关掉，换成纯粹的 round-robin。这直接降低了运维复杂度，也消除了 pod 重启时的请求中断窗口。

**如果你是 MCP 客户端开发者**

需要确认你的客户端对会话丢失的处理逻辑。如果目前是"会话丢失就报错停止"，需要改成"会话丢失时重新初始化连接"。另外，多轮请求模式下，服务端不再主动推送，客户端需要实现自己的轮询或订阅机制来获取流式结果。

## 还没解决什么：MCP 的下一步

Stateless 解决了传输层规模化问题，但 MCP 的 2026 路线图上还有几件大事未完成：

- **Agent 间通信（A2A）**：路线图明确提到，但 SEP 还未定稿。目前 MCP 定义的是"客户端-服务端"通信模型，多个 Agent 之间如何发现彼此、如何建立信任、如何协商能力集，还没有标准。
- **服务端发现机制**：如何在不手动配置 URL 的情况下让客户端找到可用的 MCP 服务器，目前仍是手动配置为主。
- **企业级认证**：SSO 集成、审计日志、基于属性的访问控制（ABAC）——这些是企业部署的必要条件，但路线图文档中标记为"in progress"。

换句话说：**MCP 2026-07-28 解决的是让它在 Kubernetes 里跑得起来的问题，但让它在企业里用得安心，还有距离。**

## 接下来看什么

- **7 月 28 日正式规范发布**：RC 阶段 SDK 维护者会根据自己的实际 workload 验证这六个 SEP，7 月 28 日的最终版本是否会有调整值得关注。
- **Python SDK v2 稳定版发布**：目前是 beta，它是大多数 MCP 服务器的底层依赖。v2 的稳定版发布意味着生产级部署有了明确的基线。
- **MCP 官方注册表（Registry）进展**：Anthropic 2024 年的原始发布没有包含服务端发现，规范发布后是否有官方的 central registry 来做动态发现，会直接影响大型多租户系统的架构设计。
- **A2A 协议 SEP 进展**：如果 MCP 继续扩展到 Agent 间通信，它与 LangChain 的 LangGraph、Microsoft 的 AutoGen 生态之间的竞争合作关系会进入新阶段。

---

## 参考资料

- [The 2026-07-28 MCP Specification Release Candidate](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/)
- [MCP Just Went Stateless — Microsoft Community Hub](https://techcommunity.microsoft.com/blog/appsonazureblog/mcp-just-went-stateless-%E2%80%94-what-the-2026-spec-changes-about-scaling-on-app-servic/4530222)
- [MCP 2026-07-28: The Stateless Release Candidate, Explained — MCP.Directory](https://mcp.directory/blog/mcp-2026-07-28-release-candidate)
- [Your MCP Server Works Locally. Then Kubernetes Kills the Session — SaaSForge](https://saasforge.cz/blog/mcp-stateless-kubernetes/)
- [Exploring the Future of MCP Transports — Model Context Protocol Blog](https://blog.modelcontextprotocol.io/posts/2025-12-19-mcp-transport-future/)
- [MCP Server Session Lost in Multi-Worker Environment — GitHub Issue #520](https://github.com/modelcontextprotocol/python-sdk/issues/520)
- [How to Migrate Your MCP Server to Stateless Mode — Chanl Blog](https://www.channel.tel/blog/mcp-stateless-spec-2026-guide)
