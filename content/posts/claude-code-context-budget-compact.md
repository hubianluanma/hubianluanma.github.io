+++
date = '2026-10-01T07:35:48+08:00'
draft = false
title = 'Claude Code 上下文Budget与compact策略：什么时候 /best-of-n 比硬撑更值'
description = "Claude Code 在 200K-1M token 上下文里跑了 60% 之后，模型开始降级。社区沉淀出两条互斥路径：/compact 回收空间 vs /best-of-n 另起炉灶。本文给出一套可操作的决策树，包含实操命令、token 节省的实测数字、以及三个踩坑案例。"
tags = ["AI", "工具"]
categories = ["AI 工具"]
author = "Spiral"
+++

Claude Code 上下文用到 60% 之后，模型开始变笨。这是 2026 年每一个重度用户都踩过的墙。

背后的数字是公开的：Anthropic 的 Thariq 早在 2025 年底就说过，1M 上下文的模型在 300-400K token 处进入 context rot；标准 200K 上下文在 40% fill 时就开始进入「dumb zone」。这不是 Bug，是注意力机制在超长上下文上的天然衰减。

社区沉淀出两条互斥路径：

- **路径 A**：/compact 压缩上下文，腾出空间继续干
- **路径 B**：/best-of-n 放弃当前 session，另起炉灶并行跑多个候选

哪条路更值，取决于三个变量：任务类型、上下文消耗原因、以及你愿意花多少 token 去买正确答案。

---

## 上下文为什么越用越贵

Claude Code 的 token 消耗有两层。

**显性消耗**：你输入的 prompt、文件内容、工具输出。容易理解。

**隐性消耗**：每次 /compact 触发时，Claude 会用当前上下文重新生成一个压缩版本来替换历史记录。这个过程本身也要消耗 token，而且压缩后的上下文不一定保留了你真正需要的信息。

Cursor 的 benchmark 数字说了一件反直觉的事：在同一个多文件重构任务里，Claude Code 消耗了 ~33K token 零错误完成；Cursor 的 agent 模式烧了 ~188K 而且中间有错误。**5.5 倍的差距**，来源不是模型能力，而是架构——Cursor 加载所有东西（打开的文件、导入、语义匹配、git 历史），Claude Code 按需拉取。

这意味着：Cursor 用户更容易撞上下文墙，因为积累的无关内容更多。Claude Code 用户的问题是另一类：当任务本身需要跨大量文件做关联推理时，上下文在任务完成前就触发了压缩。

---

## 决策树：什么时候该 /compact，什么时候该 /best-of-n

先问三个问题，按顺序判断：

**1. 上下文消耗的原因是「工具调用日志」还是「文件内容」？**

如果是工具调用日志（grep 结果、测试输出、命令返回），/compact 对这类文本压缩效果最好，可以快速把上下文从 60% 压回 30%。继续任务。

如果是文件内容（你需要同时理解 50 个文件才能做决策），压缩会丢失关键信息。此时 /compact 不是正确答案。

**2. 任务的正确性是否可以事后验证？**

如果是（测试通过、编译通过、PR diff 可审查），/best-of-n 的代价更值得——你买的是多候选验证，减少返工。

如果是（主观风格选择、API 设计决策），/best-of-n 产生的是「多个可能都对」的输出，反而浪费。

**3. 任务还有多少剩余工作量？**

如果你已经完成了 80%，只剩收尾，/compact 撑过去更划算。

如果你还有 50%+ 的工作量，而且上下文已经超过 60%，继续硬撑的代价通常高于重新开始的代价。

```bash
# 查看当前上下文填充量（Claude Code status line）
# 在 Claude Code 里输入 /status 或看状态栏右侧的百分比

# 当上下文超过 50%，主动触发 compact
/compact

# compact 前加提示，告诉模型重点保留什么
/compact 重点保留 auth refactor 的文件列表和当前进度，其他 grep 输出可以压缩
```

---

## 实测数字：/best-of-n 的 token 账怎么算

/best-of-n 在 Claude Code 官方并未作为一级命令暴露（Cursor 3 在 2026 年 4 月把它做成了 `/best-of-n` slash command），Claude Code 的等价能力是 Subagent + Worktree 组合，或者 Agent Teams。

DecimalAI 的 benchmark（2026 年 8 月）测过 Best-of-N skill 在 Claude Code 上的表现：

- 22 个案例，lift +27%（45% → 73% pass rate）
- Token 消耗：1,725 vs 1,849（基础），/best-of-n 反而省了 7% 的 token，因为失败路径被提前终止
- 每个 case 平均多花 1 个 agent turn

也就是说：**多跑 1 个 candidate 的 token 成本，通常被减少返工的收益覆盖**。但这个账要算具体场景，不是所有任务都值得。

Claude Code Agent Teams（2026 年 2 月上线）的 token 消耗更激进：比单 session 高 3-7 倍。Anthropic 自己说 plan mode 下约 7x token 消耗。所以 Agent Teams 适合大任务前的「多角度调研」，不适合高频小任务。

| 场景 | 推荐路径 | token 代价 | 适用条件 |
|------|---------|-----------|---------|
| 多文件重构，测试可验证 | /best-of-n + worktree | 2-3x 单次 | 正确答案可验证，失败代价高 |
| 单文件风格调整 | /compact | <1x | 上下文主体是工具日志，压缩不掉关键内容 |
| 跨模块架构调研 | Agent Teams | 3-7x | 需要多个专家视角并行，输出最后合并 |
| 简单 bug 修复 | /compact 或硬撑 | <0.5x | 任务简单，上下文没到 60% 红线 |

---

## 三个踩坑实录

**坑 1：compact 压缩掉了还没实施的方案**

任务：用 Claude Code 重构一个 12 个文件的认证模块。跑了很久，上下文到 55%，Claude 开始建议 /compact。我执行了，压缩后它继续编码，但删掉了一个还没实施的 JWT 方案笔记。

三小时后，测试发现那个方案被遗忘，导致 session 需要重建重来。

教训：/compact 前手动列一下「还没做但讨论过」的要点，compact 后对照检查。

**坑 2：/best-of-n 没设评分标准，跑出来没法选**

在一个 API 设计任务上开了 3 个 worktree 并行跑，输出了 3 套完全不同的设计方案。没有预设评分标准，最后只能靠「感觉」选了一套，后来发现另一套在可扩展性上明显更好。

教训：/best-of-n 启动前必须定义评分 rubric（正确性、简洁性、可维护性各占多少权重），否则多 candidate 只是制造选择困难。

**坑 3：Agent Teams 开太久忘记关，token 烧了一夜**

有一次开了一个 5 人的 Agent Team 跑架构调研，任务完成后没有显式关闭团队。Anthropic 的设计是 agent 完成后自动释放锁，但 mailbox 和 task list 的清理依赖进程正常退出。第二天早上一看，team 一直在 idle 状态，token 消耗了 12 小时。

教训：跑 Agent Teams 时用 `Shift+Tab` 定期检查团队状态，或者在 settings.json 里设 `teamIdleTimeout`。

---

## 上下文 budget 的日常维护

Boris Cherny（Claude Code 团队成员）给过一个可操作的日常节奏：

- 打开 Claude Code，先看状态栏右侧的上下文百分比
- 超过 40% 就开始注意，不等到 60% 才动
- 跨任务切换时用 `/clear` 而不是靠自然积累

Thariq 的经验值更具体：1M 上下文模型里，300-400K 是 context rot 的临界点；任何 intelligence-sensitive 的任务，保持在 300K 以下。如果你在做架构级别的决策，这个红线还要往下降到 200K。

---

## 接下来看什么

1. **Anthropic 官方 context 管理文档的更新**：Claude Code 的上下文策略一直在演进，2026 年 Q4 可能会有新的压缩算法或 cost-based 自动 compact 策略。关注 `code.claude.com/docs` 的 changelog。

2. **Cursor 3 的 /best-of-n vs Claude Code worktree 对比评测**：Cursor 把 /best-of-n 做成了原生命令，Claude Code 需要手动组合。两者在并行 candidate 数量上限和质量上的差异，还没有独立的第三方 benchmark。

3. **Prompt cache 成本占比的最新数据**：2026 年中旬的 arxiv 论文发现 cache 读写占 Claude Code 账单约 87%，压缩 hook 对实际计费的影响远小于对原始 token 量的影响。这个比例在不同任务类型上的分布值得进一步看。

---

## 参考资料

- [Claude Code 官方文档：使用 worktree 运行并行会话](https://code.claude.com/docs/zh-CN/worktrees)
- [Anthropic Claude Code Agent Teams 介绍](https://blog.laozhang.ai/en/posts/claude-code-agent-teams)
- [When Does Restricting a Coding Agent to execute_code Help? — arxiv 2607.10569](https://arxiv.org/pdf/2607.10569)
- [Token Reduction Is Not Cost Reduction — arxiv 2607.12161v4](https://arxiv.org/pdf/2607.12161v4)
- [Claude Code Agent Teams 实战指南](https://marc0.dev/en/blog/claude-code-agent-teams-multiple-ai-agents-working-in-parallel-setup-guide-1770317684454)
- [Best of N Skill — DecimalAI](https://app.decimal.ai/skills/hmbown-best-of-n)
- [Claude Code vs Cursor in 2026 — BuildThisNow](https://buildthisnow.com/blog/tools/claude-code-vs-cursor-2026)
