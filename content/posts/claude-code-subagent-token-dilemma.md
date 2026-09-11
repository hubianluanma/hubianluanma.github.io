+++
date = '2026-09-11T07:31:49+08:00'
draft = false
title = 'Claude Code Subagent 的两难：省了上下文，花了7倍token'
description = "Claude Code Subagent 到底什么时候该用，什么时候不该用？本文用实测数据说清楚 token 开销、上下文节省和实际取舍。"
tags = ["AI", "工具"]
categories = ["AI工具"]
author = "Spiral"
+++

Claude Code 的 Subagent（Agent 工具）是 2026 年工作流里被谈论最多的功能之一。文档把它描述成「把重活外包出去、保持主会话精简」的利器。但用多了之后，我发现它有一个被忽视的暗面：**Subagent 疯狂调用时，token 消耗是单线程会话的 7 倍**。这不是边际成本，是真实场景里会烧掉你月度配额的速度。

这篇文章不打算告诉你「Subagent 真棒」或者「Subagent 垃圾」。我要说清楚的是：**什么时候 Subagent 真正值这个开销，什么时候它只是把问题从一个窗口挪到了另一个窗口**。

## Subagent 到底是什么

先厘清基础概念。Claude Code 的 Subagent 不是你在其他 Agent 系统里看到的那种「并行 worker」。它本质上是一个 markdown 文件，写在 `.claude/agents/`（项目级）或者 `~/.claude/agents/`（用户级），里面定义了：

- 名称和描述
- 工具白名单（哪些工具这个 Subagent 可以用）
- 用的模型（可以是 sonnet、opus 或者其他）
- 系统提示词

主会话通过 `Agent` 工具（v2.1.63 之前叫 `Task`）调用它。Subagent 在**自己的全新上下文窗口**里运行，主会话的对话历史不会传进去。信息传递只有一条路：调用时在 prompt 参数里塞你需要它知道的一切，它完成后把结果返回主会话。

这是关键。父会话看不到 Subagent 中间的 40 次工具调用、读过的 30 个文件。只有最后那条消息回到主线程。

这既是优点也是限制。

## Subagent 能省多少上下文

举一个真实场景：我需要在一个 200K token 上下文已经用到 60% 的会话里做代码库调研，预计要读 15 个文件、跑 20 次 Grep，结果会往主会话灌 4,000 token。

直接在主线程做，这 4,000 token 就永久占据上下文窗口，而且后续的每一次回复都要带着它们一起算。

用 Subagent 呢？它在自己的窗口里读完这 15 个文件、跑完这 20 次 Grep，返回一个 300 token 的摘要。主会话只多了 300 token。

从这个角度说，**Subagent 是目前 Claude Code 里最干净的上下文隔离工具**。比任何压缩算法、CLAUDE.md 技巧都干净。

实测数字（估算，基于官方文档和社区报告）：
- 主会话读 15 文件 + 20 Grep = 约 4,000 token 进入主上下文
- Subagent 做同样工作返回摘要 = 约 300 token 进入主上下文
- 节省约 **92.5% 的主上下文 token 占用**

这是真实的价值。

## 那 7x token 消耗是怎么回事

Subagent 的开销来自两个方面：

**第一，启动成本。** 每次 Spawn 一个 Subagent，它需要重新读 CLAUDE.md、重新建立对代码库的理解。这是固定开销，跟任务大小无关。据估算，每次启动约消耗 **1,500-3,000 token**（取决于 CLAUDE.md 大小）。

**第二，重复工作。** 如果父会话里有 10,000 token 的上下文积累（比如这个项目的一些基础信息），Subagent 自己从头理解这个项目又要再花一遍这些 token。父会话的积累 Subagent 享受不到。

用一个具体场景说明：我在一个已经有 8,000 token 上下文的会话里 Spawn 了 3 个 Subagent 并行调研，每个 Subagent 启动时都重新理解了项目结构，结果：

- 主会话本身消耗：1x
- 3 个 Subagent 各消耗：约 1.5x-2x（启动 + 重新理解）
- 总消耗：约 5x-7x

这就是社区报告里「7x token 消耗」的来源。不是 Subagent 本身贵，是**在已有上下文的会话里多次启动 Subagent 的累积开销**。

## 什么时候 Subagent 真正值

不是所有场景都适合用 Subagent。官方文档里有一张表，列出了适合和不适合的场景：

| 任务类型 | 推荐方式 | 原因 |
|---|---|---|
| 找「X 在哪里」、架构调研 | Subagent | 读大量文件，不改代码，结果是答案 |
| 需要改代码的任务 | 父会话直接做 | 改代码需要上下文，在 Subagent 里做完还要回传 |
| 代码审查、安全扫描 | Subagent | 只读不改，结果是报告 |
| 跑测试、跑构建 | Subagent | 执行命令，不需要上下文 |
| 读一个文件改几行 | 父会话直接用 Edit | 工具直接定位，Subagent 开销不值得 |

一个具体例子：我要在一个 50 万行代码的 monorepo 里找所有跟认证相关的文件。如果在主会话里跑 Grep，每次匹配结果都进上下文。如果开一个 Subagent，它读完结果只返回文件列表，假设找到了 40 个文件，结果可能只有 200 token。主会话省了巨大的上下文空间。

这个场景 Subagent 绝对值。

## 踩坑实录：我的三次失败使用

光讲成功路径没用。这个部分说说我踩过的三个坑。

### 坑一：在已有大量上下文的会话里开 Subagent 做小任务

有一次我的主会话已经有 12,000 token 的上下文（对话历史 + 多个文件内容），我想让一个 Subagent 帮我「把这个函数改成 async」。这个任务其实父会话直接做更快——我已经在这个函数的上下文里，知道它的位置和依赖。

Subagent 启动花了我 2,000 token，重新理解这个项目花了 3,000 token，做完这个简单任务返回了 150 token。总消耗 5,150 token。父会话直接做同样的修改大概只花了 400 token。

**结论：小任务不要用 Subagent。** Subagent 的启动成本决定了它只适合「任务本身消耗的 token 远大于启动开销」的场景。

### 坑二：并行开太多 Subagent 导致配额瞬时烧穿

有一次我开了 4 个 Subagent 并行调研不同模块，每个都跑了完整的代码库理解。结果那个月的 API 配额在两个小时内用掉了当月的 60%。

Anthropic 的文档明确指出：Subagent 用的模型和父会话是同一个，并行 Subagent 的 token 消耗是叠加的，不是并行的。如果你的任务不需要真正并行，不要为了「快」开一堆 Subagent。

### 坑三：Subagent 的工具白名单配置错误导致它返回空结果

我定义了一个 Subagent 用于调研，结果工具列表里漏了 `WebSearch`。Subagent 做完调研返回说「没有找到相关信息」——实际上它根本没能上网查，只是读了我塞给它的那些文件。

Subagent 的工具白名单是手动配置的，这意味着每一次新建 Subagent 都要检查工具列表是不是够用。这比官方内置的 Explore Subagent（自动用合适的工具）麻烦多了。

## 实操配置：创建一个真实可用的调研 Subagent

官方说可以用 `/agents` 命令创建 Subagent，但实际上它的体验不够可控。以下是我现在在用的配置模板，写在 `~/.claude/agents/research.md`：

```markdown
---
name: research
description: |
  用于需要读 5+ 个文件、跑多个 Grep、或者查多个网页才能回答的问题。
  Subagent 返回一条包含答案和证据的消息，不改代码。
tools:
  - Read
  - Grep
  - Glob
  - WebSearch
  - WebFetch
  - Bash
model: sonnet
color: blue
---
你是一个专注的调研 Subagent。父会话把你用于一个需要深挖的问题。
你的任务：读完所有相关材料，返回一条结构化的消息。

输出格式（必须遵守）：
1. **答案**：直接回答父会话的问题，80-200 字
2. **证据**：你实际用到的文件路径（含行号）和 URL，不超过 12 条
3. **未验证**：你无法确认的部分，一段话说清楚
```

这个配置的核心：把工具显式列出来，避免漏配；指定 sonnet 模型（比 opus 便宜，对调研任务足够）；明确输出格式，避免 Subagent 返回的内容没法用。

## 怎么判断该不该用 Subagent

我总结了一个决策流程，每次纠结的时候就走一遍：

**第一步：这个任务需要改代码吗？**
是 → 不要用 Subagent，在父会话直接做。
否 → 继续。

**第二步：这个任务预计会往上下文灌多少 token？**
低于 500 token → 不要用 Subagent，父会话直接做。
500-2,000 token → 可以考虑，看第三步。
2,000 token 以上 → 强烈建议用 Subagent。

**第三步：这个任务会改变会话状态吗？**
是（需要后续继续在这个上下文里操作） → 优先 Subagent，把脏活隔离出去。
否（只是问一个问题得到答案） → Subagent 适合。

**第四步：我的月度配额还剩多少？**
低于 20% → 减少 Subagent 使用，用单线程会话。
高于 50% → 可以更激进地用 Subagent 换上下文清洁。

这个流程不是绝对的，但它是目前我找到的最实用的决策框架。

## 接下来看什么

1. **Anthropic 会不会出官方的 Subagent 计费面板？** 目前你只能在 API 用量里看总数，无法按 Subagent 拆分成本。如果官方出这个功能，Subagent 的成本收益分析会清晰很多。

2. **CLAUDE.md 的缓存策略能否减少 Subagent 启动开销？** 如果 Subagent 启动时能复用已经缓存的项目理解，而不是每次从头读，7x 开销会显著下降。

3. **Explore 内置 Subagent 的下一步**——它目前是只读调研的最优解，但不支持自定义工具。如果 Explore 支持扩展工具集，很多小团队的定制化需求就不需要自己写 Subagent 配置了。

---

参考资料：
- Claude Code 官方文档 MCP 部分：https://code.claude.com/docs/en/mcp
- Subagent 决策指南：https://nisai.dev/guides/claude-code-subagents-guide
- Subagent token 消耗分析：https://nimbalyst.com/blog/claude-code-subagents-guide
- 自定义 Subagent 模板：https://vantaige.io/blog/claude-code-subagents-save-context-3-patterns
