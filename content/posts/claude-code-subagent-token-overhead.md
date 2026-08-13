+++
date = '2026-08-13T07:31:37+08:00'
draft = false
title = 'Claude Code Subagent 成本实测：什么时候该并行，什么时候该串行'
description = "Subagent 是 Claude Code 最强大的能力之一，也是最容易踩坑的特性。本文用真实 benchmark 数据讲清楚：token 开销从哪来、3-5 并发的甜区怎么算、什么时候该直接用 Skill 或工具调用而不是 Spawn 一个子 Agent。"
tags = ["AI", "工具", "Claude Code"]
categories = ["AI 工具"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/08/13/img_1786579278_1.jpeg", alt = "齿轮与账本：并行 subtask 的 token 成本隐喻" }
+++

Claude Code 的 Subagent 是很多人用上就回不去的功能。Fan out 一批 Agent 并行处理，再 Fan in 合并结果，听起来很美。但实际上每个 Subagent 都是一次完整的 Agent 实例化，不只是「分一个任务出去」这么简单。

本文基于 2026 年 7-8 月社区 benchmark 数据，讲清楚 Subagent 的真实 token 开销、甜区并发数，以及最重要的——**什么时候不该用它**。

## Subagent 的 token 开销从哪里来

当你 Spawn 一个 Subagent，Claude Code 实际上做这几件事：

1. 把当前 context 里相关的部分「打包」传给子 Agent
2. 子 Agent 启动，消耗自己的完整 context window
3. 结果返回，主 Agent 再把输出接回来

每个环节都有成本。AI Pricing Guru 在 2026 年 7 月的实测数据：

- **单个小型公共 MCP Server，每次请求附加 1,000–1,400 个 token**
- 5 个小型 Server 接入，约增加 **4,900 个 token 载荷**，实际计费 token 约 **6,967 个**

这意味着什么？如果你只让 Subagent 做一件工具调用能搞定的事（比如读一个文件、查一个 API），Subagent 的启动开销远远超过任务本身的价值。

## 真实踩坑案例

我自己的一个失败场景：想用 Subagent 并行抓取 8 个网页的摘要，各自返回一句话。逻辑上完全合理——并行 8 个请求，速度应该是串行的 1/8。

实际结果：8 个 Subagent 各自启动、加载 context、执行、返回，总耗时 47 秒；token 消耗比直接串行抓取多了 **340%**。而这个任务本质上只要 `curl` 或一个 fetch 工具调用就能完成，耗时 3 秒。

**教训**：Subagent 不是免费的并行化工具，它是专门为「需要独立推理链的复杂子任务」设计的。如果子任务机械且简单，工具调用或 Skill 更合适。

## 甜区：3-5 个并发 Subagent

Totalum Blog 的 2026 年生产环境数据：

> "Three to five concurrent subagents is the sweet spot for most jobs."

超过 5 个并发，收益递减开始明显——Anthropic API 的 Tier 1 限速是 50 req/min，队列开始堆积，并行优势被抵消。MindStudio 的分析更直接：

> "Fan out, process in parallel, fan in and synthesize — that loop is the core of effective sub-agent design. Spawning a sub-agent to fetch a single URL and return one sentence is wasteful. Sub-agents have overhead — they're a full agent instantiation."

这个「Fan Out / Fan In」模式最有价值的场景：

- **代码审查 + 修复**：一个 Subagent 找问题，另一个 Subagent 修复，第三个综合验收
- **多文件分析**：每个 Subagent 负责一个模块，最后主 Agent 汇总架构问题
- **调研 + 写作**：一个 Subagent 搜索背景，一个 Subagent 整理事实，主 Agent 撰写

而**不该用 Subagent** 的场景：
- 纯机械任务（读文件、查天气、调 API）
- 单次简单计算
- 只需要返回一句话的任务

## 一个可复用的判断框架

```
任务能拆，拆完的子任务是否需要独立推理？
  ├── 是 → Subagent（值得付出完整实例化开销）
  └── 否 → 工具调用 / Skill（开销接近零）
```

换句话说：如果你 Spawn 出来的 Subagent 里只写了一两句 Prompt，大概率用错工具了。

## Context 隔离：Shadow Workspace 的另一个成本维度

提到 Subagent 就不能不提 Claude Code 的 Shadow Workspace（沙盒工作区）。它提供文件系统级和网络级隔离，让 Subagent 在独立环境里运行。

听起来安全，但有个现实成本：**每次启动 Shadow Workspace，context 里要加载隔离环境的配置和权限信息**。在 2026 年 8 月的 Claude Code Changelog 里有一条修复：

> "Fixed background session isolation not canonicalizing symlinked working directories, which could let sessions escape their workspace folder"

这条 fix 背后是：之前 symlink 处理有问题，可能导致 session 逃逸。隔离越严格，配置越复杂，context 负担越重。如果你不需要真正的隔离，只是想让 Subagent 独立工作，默认的非沙盒模式开销更小。

## 实际数字：Claude vs Codex 的 token 消耗对比

MorphLLM 的 2026 年 8 月对比测试：

| 维度 | Claude Code (Claude Opus 4.8) | OpenAI Codex (GPT-5.5) |
|------|------|------|
| SWE-bench Pro | 69.2% | 58.6% |
| SWE-bench Verified | 88.6% | 88.7% |
| 上下文窗口 | 1M token | 200K token |
| Token 消耗 | 3-4x more | 更节省 |

结论：Claude Code 的 token 消耗确实更高，但换来的是更强的推理深度和更长的上下文保持能力。如果任务需要深度分析，Claude 的贵是有道理的；如果只是批量机械任务，Codex 的性价比更高。

## 接下来看什么

1. **Anthropic API Tier 4 的 rate limit 上调**：Tier 4 用户可以跑 10+ 个 Subagent 并发，甜区数字会变
2. **Agent Teams 的稳定版发布**：Claude Code 的跨 session 多 Agent 编排目前还在实验阶段，稳定后将改变生产工作流
3. **Claude Code 缓存预热的 token 节省比例**：如果 Subagent 任务可以缓存预热，6,967 个 metered token 的开销可以显著压缩

---

## 参考资料

- [Claude Code subagents: the 2026 production playbook](https://www.totalum.app/blog/claude-code-subagents-totalum) — 3-5 并发甜区数据
- [How to Use Sub-Agents in Claude Code](https://www.mindstudio.ai/blog/claude-code-sub-agents-explained) — Fan Out/In 模式分析
- [Claude Code Token Overhead: Pricing Impact (July 2026)](https://www.aipricing.guru/blog/claude-code-token-overhead-pricing-impact-july-2026/) — MCP Server token 开销实测
- [Codex vs Claude Code (August 2026)](https://www.morphllm.com/comparisons/codex-vs-claude-code) — Token 消耗与 benchmark 对比
- [Claude Code Changelog (August 2026)](https://www.gradually.ai/en/changelogs/claude-code/) — Shadow Workspace 隔离修复记录
