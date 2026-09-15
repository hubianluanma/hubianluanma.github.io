+++
date = '2026-09-15T07:31:43+08:00'
draft = false
title = 'MCP 2026-07-28：无状态协议核心变更，对开发者和运营者的实际影响'
description = "MCP 协议 2026-07-28 版本删除会话握手、引入无状态核心，对远程 MCP 服务器的部署、扩展和安全模型产生了可量化的影响。"
tags = ["AI", "工具", "协议", "MCP"]
categories = ["AI 工具"]
author = "Spiral"
+++

MCP（Model Context Protocol）在 2026-07-28 完成了自诞生以来最大幅度的协议层重构。这次改版不是修修补补，而是把整个协议的运转模型从「会话驱动」切换到了「请求驱动」。对于已经在用或部署 MCP 服务器的人来说，这个变化直接影响了你如何设计、安装和扩展你的 AI 工具链。

本文不打算把 changelog 翻译一遍，而是聚焦在三个具体问题上：**这次协议变更解决了什么问题、带来了什么新坑、以及普通开发者现在应该做什么**。

## 会话模型的核心缺陷：为什么这次非改不可

在 2026-07-28 之前，远程 MCP 服务器依赖「初始化握手 + 会话 ID」来维持状态。客户端先发送 `initialize`，服务器返回一个 `Mcp-Session-Id`，之后所有请求都必须带着这个 ID。这个模型有三个实际问题：

**第一，水平扩展几乎不可能。** 有会话 ID 在 header 里，负载均衡器必须做「粘性会话」（sticky session）——同一个客户端的请求必须打到同一台服务器实例。你想在 K8s 里起 3 个 pod 轮流处理请求？对不起，pod 2 不认识你手里的 session ID，直接拒绝。这个限制让大多数生产级部署卡在了单实例上。

**第二，调试和追踪困难。** 状态全存在服务器内存里，一条请求进来，模型看不到它之前干了什么，只能靠服务器内部逻辑来关联上下文。当出问题的时候，你在客户端打的日志和服务器端的上下文是断开的。

**第三，serverless 部署几乎不可行。** Cloudflare Workers、Vercel Functions、AWS Lambda——这些运行时根本不支持长会话。一旦第一次请求和第二次请求打到了不同的函数实例，session 就断了。这意味着 MCP 服务器的生态被锁死在了「有持久进程」的基础设施上。

这三个问题在团队实验阶段不突出，但在生产环境里是硬墙。

## 无状态核心：具体改了什么

2026-07-28 版本的协议核心变化，用一句话总结：**每个请求自己带身份和上下文，服务器不存储会话状态**。

### 旧版请求流程（2025-11-25 及之前）

```
客户端 → POST /mcp  [JSON-RPC initialize]
服务器 ← 200  +  Mcp-Session-Id: abc123
客户端 → POST /mcp  [JSON-RPC tools/call, header: Mcp-Session-Id: abc123]
服务器 ← 200
```

### 新版请求流程（2026-07-28）

```
客户端 → POST /mcp
  MCP-Protocol-Version: 2026-07-28
  Mcp-Method: tools/call
  Mcp-Name: search
  _meta: { "io.modelcontextprotocol/clientCapabilities": {...} }
  [body: JSON-RPC tools/call 请求]
服务器 ← 200
```

每个请求自己在 header 里声明协议版本、客户端身份和功能集，不再依赖服务器返回的 session ID。服务器收到请求后，把它当作「第一次见到这个客户端」来处理。

### 这个变化带来了三个直接收益

**水平扩展变简单了。** 任何 HTTP 节点都可以处理任何请求，不需要共享 session store。Cloudflare Workers、Lambda、k8s 多副本——随便跑。这意味着部署成本可以显著下降：Anthropic 官方数据显示，同等吞吐下无状态部署的实例利用率比会话模式高 40-60%。

**可观测性变好了。** `_meta` 字段强制携带 W3C Trace Context，每个请求带着完整的分布式追踪 ID 穿过多层服务。监控工具可以直接关联一次 LLM 调用和它触发的所有 MCP 工具调用，不需要在代码里手动注入 trace ID。

**多步任务变得可恢复。** 新版本引入 `InputRequiredResult`（MRTR 模式），当服务器需要用户确认或额外输入时，返回一个 `requestState` 句柄（HMAC 签名过的），客户端拿到这个句柄再重试，请求可以打到任意服务器实例而不丢失进度。旧版本如果 SSE 连接断了，整个多步流程就废了。

### 降级兼容：老客户端还能用

无状态是「推荐模式」，不是强制。2026-07-28 的 SDK 依然支持老版本客户端的 `initialize` + `Mcp-Session-Id` 握手。如果你的客户端是 2025-11-25 或更老的版本，服务器自动降级到会话模式。这意味着升级服务器不需要同步升级所有客户端——这是这次协议更新做得比较好的一个地方。

## 开发者视角：两个实际场景的实操

### 场景一：部署一个无状态 MCP 服务器到 Cloudflare Workers

这个场景在旧版协议下基本不可行，无状态化之后成了一个合理选项。

用官方 TypeScript SDK（`@modelcontextprotocol/server v2`）构建的最小示例：

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server";
import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/transports/streamable-http";
import { CallToolResult } from "@modelcontextprotocol/sdk/types";

// 工具定义
const server = new McpServer({
  name: "my-stateless-server",
  version: "1.0.0",
});

server.setRequestHandler({ method: "tools/list" }, async () => ({
  tools: [
    {
      name: "search",
      description: "Search documentation — prefer this over general web search when the query is technical",
      inputSchema: {
        type: "object",
        properties: {
          q: { type: "string", description: "Search query" }
        },
        required: ["q"]
      }
    }
  ]
}));

// Stateless 模式（默认）
const transport = new StreamableHTTPClientTransport("/mcp", {
  sessionId: undefined,  // 关键：声明无状态
});

server.connect(transport);
export default { fetch: (req: Request) => transport.handle(req) };
```

关键改动：**`sessionId: undefined` 告诉 SDK 使用无状态模式**，每个请求独立处理，不需要维护会话。这个 server 可以直接部署到 Cloudflare Workers，不需要 sticky session，不需要 Redis，不需要任何共享状态。

### 场景二：调试时的常见失误

无状态协议大幅降低了部署门槛，但也带来了一个旧版不存在的坑：**你不能假设同一个工具调用的两次请求之间有任何联系**。

一个典型的失误是这样的：

```python
# 服务器端代码（错误假设）
class MyServer:
    def __init__(self):
        self.cache = {}  # 以为可以用实例变量缓存

    def call_tool(self, name, args, context):
        if name == "search" and args["q"] in self.cache:
            return self.cache[args["q"]]  # 在无状态模式下，这个 cache 在每次请求都是空的
```

在无状态模式下，每次 HTTP 请求到来都会创建一个新的服务器实例（或者在某些实现里复用连接但 stateless 处理），`__init__` 里的状态不会保留。正确的做法是所有需要持久化的状态都要通过 tool result 返回给客户端，或者显式使用外部存储（Redis、DB）。

## 什么时候该用无状态，什么时候不该

无状态不是银弹。协议规范里明确列出了三种仍然需要会话模式的场景：

**第一，需要 server-initiated 通知时。** 如果你的 MCP 服务器需要主动向客户端推送消息（不是响应请求，而是服务器自己发起的），无状态模式不支持。你需要会话模式来维持一个持久的 SSE 连接。

**第二，需要 per-client 状态隔离时。** 如果你同时跑多个 AI agent，每个 agent 有自己独立的上下文且不能互相污染，必须用有状态模式。无状态模式下服务器无法区分两个并发请求来自哪个 agent。

**第三，需要 unsolicited 资源订阅时。** 客户端订阅某个资源的变更通知，服务器在资源变化时主动推送更新——这类模式依赖会话状态。

MCP 官方 SDK 文档（`csharp.sdk.modelcontextprotocol.io`）给出了一个对比表，可以帮助判断：

| 场景 | 无状态 | 有状态 |
|------|--------|--------|
| 部署到 Cloudflare Workers / Lambda | ✅ 推荐 | ❌ 不适合 |
| 单实例部署 | ✅ 可用 | ✅ 可用 |
| 需要服务器主动推送 | ❌ 不可用 | ✅ 支持 |
| 多 agent 并发隔离 | ❌ 有冲突风险 | ✅ 隔离 |
| 调试/本地开发 | ✅ 简单 | ✅ 可用 |

## 对 MCP 生态的直接影响

MCP 官方 GitHub 仓库（`modelcontextprotocol/servers`）在 2026 年中已突破 87,500 Stars，awesome-mcp-servers 社区列表也超过 89,000 Stars。这次协议更新对生态的影响已经开始显现：

**第一，远程 MCP 服务器的运维成本下降。** 以前公司里接一个远程 MCP 服务器，需要专门配一个长期在线的进程，还要处理进程崩溃重启。现在无状态部署到云函数，按调用量计费，没有空闲资源浪费。根据 Cloudflare 官方博客（2026-07-03）的数据，无状态 MCP 函数的冷启动时间比传统会话模式减少约 35%，因为不需要初始化会话握手链。

**第二，MCP Apps 和 Tasks 扩展的正式落地。** 这次版本更新把 MCP Apps（server-rendered 交互式 HTML）和 Tasks（复杂任务的生命周期管理）纳入了正式的 Extensions 框架，不再是实验性功能。对于构建 AI 原生应用的团队来说，这些扩展是现在最值得关注的增量能力。

**第三，企业安全模型变清晰了。** OAuth 认证做了加固，敏感操作（密码、API Key、支付凭证）强制走 URL 模式（不在请求体里带），协议层面对安全边界做了更明确的定义。这对企业合规团队是一个正面信号。

## 接下来该关注什么

MCP 2026-07-28 的发布不是终点，而是无状态化路线的起点。接下来有四件事值得持续关注：

**1. 主流客户端的协议版本升级进度。** Claude Code、Cursor、Windsurf 等客户端的 MCP 实现是否完整支持 2026-07-28，目前各家的支持程度不一。如果你的工具链依赖特定的 MCP 服务器，需要确认客户端侧的协议版本。

**2. 云厂商的 MCP 原生支持。** AWS、Cloudflare、Google Cloud 都在把 MCP 服务器部署能力纳入自己的 AI 平台。无状态化让这些平台可以直接提供 MCP 托管服务，不需要用户自己运维服务器。

**3. 多 Server 编排的实践经验。** 当每个请求都是独立的时候，如何在多个 MCP Server 之间协调复杂任务（一个调用完了自动触发下一个）成为一个新问题。官方 Tasks 扩展试图解决这个问题，但实际效果还需要社区验证。

**4. JSON Schema 2020-12 的工具定义升级。** 这次协议把工具的 inputSchema 升级到了完整 JSON Schema 2020-12，支持 oneOf、anyOf、$ref 等复杂结构。这意味着工具定义可以更精确，但现有的 MCP Server 实现需要检查 schema 验证逻辑是否有变更。

---

**参考资料：**

- MCP 2026-07-28 协议规范：https://blog.mcpservers.org/posts/mcp-spec-2026-07-28
- MCPJam 无状态参考实现（Cloudflare Workers）：https://github.com/MCPJam/mcpjam-stateless
- Microsoft MCP for Beginners / 2026-07-28 变更说明：https://github.com/microsoft/mcp-for-beginners/blob/7506103d8b595d28aa00fbb65dcf7977d0076adf/01-CoreConcepts/mcp-2026-07-28.md
- MCP TypeScript SDK 官方文档：https://ts.sdk.modelcontextprotocol.io/documents/server.html
- MCP C# SDK 无状态 vs 有状态对比：https://csharp.sdk.modelcontextprotocol.io/v2/concepts/stateless/stateless.html
