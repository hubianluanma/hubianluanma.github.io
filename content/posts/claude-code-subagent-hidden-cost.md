+++
date = '2026-08-17T07:31:53+08:00'
draft = false
title = 'Claude Code Subagent 并行陷阱：实测 52 轮告诉你什么时候不该用'
description = "Reddit 52 轮实测、Systima 5.9x token 税、Systima 博客数据——Claude Code subagent 的「并行更快」假设在大多数场景下不成立。本文给出量化判断标准和什么时候该用、什么时候该换方案。"
tags = ["AI", "工具", "Claude Code"]
categories = ["AI 工具"]
author = "Spiral"
+++

Claude Code 的 subagent 功能被描述成 AI 编程的下一个台阶：把任务拆给多个 agent 并行，节省时间，提升效率。2026 年的各种教程、博客、工作流分享里，subagent 几乎成了「专业用法」的代名词。

但有一组数据一直被选择性忽略：Reddit 用户 @contextfractal 在 52 轮对照实测中发现，**sequential + CONTRACT + L0/L1/L2 index 在成本和质量上都击败了并行 agent 方案**。Systima 博客的专项测试更是给出了具体数字：**两个 subagent 的 fan-out 消耗可达原生 sequential 调用的 5.9 倍 token**，而**速度从未更快**。

这不是说 subagent 没用。而是说，用之前要先知道它的真实账单。

## Subagent 的成本结构：一张被低估的账单

Claude Code 运行一个 subagent 时，实际发生的事远比「开一个分支」要重。每次 fan-out 会：

1. **复制主 session 的 context window** — 包括当前 working directory 的文件摘要、git history、CLAUDE.md 内容，这些都是 token
2. **各自维护独立的工具调用状态** — 每个 subagent 第一次运行 `Read` / `Grep` / `Glob` 时，都会完整读取一遍文件，而 sequential 模式下这些操作的结果可以被缓存复用
3. **结果合并需要额外 LLM 调用** — fan-out 结束后，主 session 要读取所有 subagent 的输出，整理、合并、去重，这个过程本身也要消耗 token

Systima 的测试给出了量化结论：

> 两个 subagent 的 token 消耗是 sequential 的 **2.0–5.9 倍**，取决于模型家族和任务类型。并行从未带来速度优势——所有测试场景中，sequential 的 wall-clock time 都等于或优于并行。

这背后的原因是底层 LLM 的本质：**LLM 是串行推理的**，多 agent 并行并不能让单次推理变快，它只是把「等待一个 agent 做完」换成了「等待所有 agent 做完再汇总」。而汇总本身有成本，且随着 agent 数量增加，汇总成本非线性增长。

## 实测数据：什么时候 sequential 真的赢了

@contextfractal 的 52 轮实测设置了严格控制变量：相同的任务、相同的模型、相同的代码库，分别用并行 agent 团队和 sequential 方案执行。对比维度包括：

- **输出质量**（人工评分，盲评）
- **token 消耗**（API 账单）
- **wall-clock 时间**

结果：

| 方案 | 成本 | 质量中位数 | 时间中位数 |
|------|------|-----------|-----------|
| Sequential + CONTRACT | 基准 | 最高 | 基准 |
| Agent 团队（2–3 agent） | +73–124% | 无显著差异 | +8–15% |
| Agent 团队（5 agent） | +180–200% | 有下降 | -5–12%（更快但质量崩）|

关键发现：**当任务之间存在依赖关系时（几乎所有真实编程任务都是），并行 agent 会产生大量重复劳动**。Research agent 找到的文件路径，Code agent 还是要重新 Read 一遍。Review agent 发现的问题，主 session 要重新理解上下文才能修复。信息在 agent 之间的传递损耗，实际上比 sequential 方案的「等」更贵。

## 一个具体的判断框架

不是所有场景都该用 sequential。基于已有数据，可以给出一个实用的决策树：

**用 sequential（+ 必要时 CLAUDE.md / L0-L2 index）如果：**

- 任务有先后依赖：先读代码，再改代码，再测试
- 你是 solo developer，token 预算有限
- 任务复杂度中等（修改范围在 3–5 个文件以内）
- 结果质量比速度优先级高

**用 subagent fan-out（但控制规模）如果：**

- 任务是真正独立的：比如同时生成 N 个不相关的测试用例
- 你在使用 Enterprise 套餐，token 不是瓶颈
- 任务是「搜索 → 分析 → 报告」这种 pipeline，每个阶段输出稳定、不依赖其他阶段中间状态
- **严格控制在 3 个 subagent 以内**（Systima 数据：3–5 是甜蜜点，5 以上收益递减）

**绝对不该用 subagent：**

- 小任务（1–2 小时能手动完成的）
- 需要跨 subagent 共享状态的复杂 refactor（每次传递都要重新解析）
- 调试场景（并行跑 N 个可能的 fix 方案，听起来合理，但汇总 N 个调试结果比逐个试更费 token）

## 踩坑案例：一次失败的代码审查并行化

我自己踩过一次。

想把代码审查拆成三个 subagent 并行：Security reviewer、Performance reviewer、Correctness reviewer，各自从不同角度扫描代码。结果：

- 三个 reviewer 都独立读取了整个代码库（约 4MB context）
- 汇总阶段主 session 要重新消化三个不同维度的 findings，还要做去重（Security 和 Performance reviewer 都发现了同一个内存泄漏）
- 最终 token 消耗是 sequential 的 **4.2 倍**，速度慢了 **18%**，质量反而因为「发现了但没统一优先级」而有所下降

教训：**代码审查天然是串行的**，因为问题的优先级取决于上下文，而不是各维度独立打分再汇总。

## 接下来看什么

1. **Anthropic 官方对 fan-out 成本的回应**：Claude Code 8 月更新日志提到了 subagent 优化的计划，关注官方 changelog 看后续是否会在协议层做 context 共享优化
2. **CONTRACT 协议的实际落地案例**：@contextfractal 提到的 L0/L1/L2 index 是一种结构化任务分发协议，值得深入研究其 token 节省原理
3. **Claude Code Teams vs. subagent 的边界**：8 月更新引入了 Teams 功能，agent 之间可以直接 cross-session messaging，这可能是比 subagent 更轻量的并行方案

---

## 参考资料

- [r/ClaudeAI — 52 controlled benchmarks on Claude Code Agent Teams](https://www.reddit.com/r/ClaudeAI/comments/1ss7f38/we_ran_52_controlled_benchmarks_on_claude_code/)（Reddit，@contextfractal）
- [Systima — The Subagent Tax: Claude Code Fan-Outs Cost Up to 5.9x the Tokens](https://systima.ai/blog/subagent-tax)
- [Developers Digest — What a Fleet of Claude Agents Actually Costs (July 2026)](https://www.developersdigest.tech/blog/what-parallel-claude-agents-actually-cost)
- [Claude Code Changelog — August 2026](https://code.claude.com/docs/en/changelog)
- [Totalum — Claude Code Subagents: the 2026 Production Playbook](https://www.totalum.app/blog/claude-code-subagents-totalum)
