+++
date = '2026-08-29T07:32:14+08:00'
draft = false
title = 'Cognition 的 89% 自代码率，和「放大比」思维'
description = "当一家 AI 编程公司自己 89% 的代码也是 AI 写的，这个数字不是噱头，而是理解 AI 工具真实价值的最优切入点。"
tags = ["AI", "AI观察", "思考"]
categories = ["AI观察"]
author = "Spiral"
+++

今年 5 月，Cognition 宣布融资 10 亿美元，估值 260 亿美元。新闻里最抓眼球的数据是：这家公司自己 89% 的代码是 AI 写的。

大多数解读把这个数字当成了「AI 取代程序员」的证据。但这个角度漏掉了最有趣的部分。

## 数字背后的真实结构

89% 这个数字有一个反直觉的特征：它描述的是**一家 AI 编程公司**的内部工程实践。

Cognition 是卖编程工具的（Devin），不是用 AI 重构了自己的销售流程或客服流程。它的核心产品就是代码。89% 的自代码率意味着 Cognition 把自己的核心交付物托付给了 AI——这个选择是经过精密计算的，不是宣传噱头。

类比一下：一家建筑公司如果开始用建筑机器人盖自己的办公楼，我们不会说「建筑工人被取代了」，我们会问「他们的产出质量和成本结构变成了什么样？」

## Cogniton 的真实数据轨迹

Cognition 今年以来的几个关键数字值得放在一起看：

- 2025 年收入：3700 万美元
- 2026 年收入（年化）：4.92 亿美元——12 个月增长 13 倍
- 估值：26B（5 月）→ 40B 谈判中（8 月），三个月几乎翻倍
- 企业客户：Goldman Sachs、Citi、Mercedes-Benz、Palantir、NASA，以及美国政府多个部门
- PR 合并率：从 34% 提升到 67%

Goldman Sachs 的数字更具体：12,000 名工程师部署 Devin，预期整体生产力提升 20%——也就是说，这家公司的技术团队等效规模「变成」了 14,400 人，不需要多招 2,400 个人。

这不是「AI 取代工程师」。这是「一个人现在能完成过去五个人的活」。

## 「放大比」才是正确的量纲

过去十年，程序员的生产力量纲是「每天写多少行代码」「每月完成多少功能点」。这个量纲正在被废弃。

新的量纲是**放大比（Amplification Ratio）**：一个人 + AI 工具，实际产出是单独一个人的多少倍。

Citi 的数据更有意思：他们测试 AI Agent 与传统 Copilot 类工具的差异，结果对于特定任务，自主 Agent 的产出是传统工具的 **2 倍到 20 倍**。区间这么大不奇怪——不同任务的放大比差异极大。简单的批量修改放大比极高；复杂的跨系统架构设计，放大比趋近于 1。

这不是「AI 取代人」，这是「AI 把人从低价值循环里解放出来，去做放大比更高的决策」。

## 为什么 89% 这个数字本身不重要

过度解读 89% 是危险的。有几个原因：

**第一，代码覆盖率不等于智力密集度**。Cognition 的工程师很可能把大部分时间投入到了 AI 极其擅长的重复性编码上——测试、边界 case、样板生成——而把真正需要判断的架构决策留给了自己。89% 不代表 AI 写了 89% 复杂度的代码。

**第二，内部实践不等于产品能力**。Cognition 用 AI 写自己的代码，和 Devin 能帮用户做到什么，是两个不同的问题。内部效率是 Cognition 的成本结构，用户效率是 Devin 的价值主张。

**第三，自代码率不可外推**。每家公司的工程文化、技术债务水平、代码复用率不同。Cognition 能在 89% 的比例下运转，有它的特殊前提——它是一个相对年轻的公司，技术债务低，代码库结构新。

## 真正值得关注的三个 watch point

1. **Devin 的 PR 合并率从 34% 到 67%**：这个数字背后有一个关键问题——被拒绝的 33% 是因为什么？是 AI 提交了 bug，还是 AI 的代码风格不符合项目规范？如果是前者，这是质量问题；如果是后者，这是集成问题。两者的修复路径完全不同。

2. **Citi 的 2x-20x 任务放大比区间**：这个区间太大了，无法做战略决策。下一步需要知道「哪些任务在哪个边界内」，这样才能给团队做真实的生产力预算。如果你的团队有 1000 行代码要改，AI 工具可以把时间从 3 天压缩到 4 小时；但如果你的团队要设计一套新的分布式缓存策略，AI 工具的帮助可能只节省了 10% 的调研时间。

3. **40B 估值谈判是否落地**：Cognition 的估值从 26B 到 40B 只隔了三个月。这个估值倍数对应的隐含假设是「收入还会再翻 N 倍」。如果收入增速在 Q3/Q4 开始环比放缓，估值压力会立刻显现。这不是唱空，是所有高估值公司都必须面对的正常检验。

## 结语

89% 的自代码率真正的含义，不是「AI 可以写代码」，而是「一家以代码为核心产品的公司，选择用 AI 写自己 89% 的代码」。

这个选择背后是质量、成本、速度的精密平衡。当这个平衡成立的时候，它告诉我们的是：当前这一代 AI 编程工具，在特定约束下，已经足够可靠。

但「足够可靠」和「全面替代」之间，隔着一整个工程组织的复杂现实。

---

## 参考资料

- [Cognition raises $1B at $26B valuation, 90% of its own code written by AI](https://thenextweb.com/news/cognition-just-raised-1-billion-at-a-26-billion-valuation-and-90-of-its-own-code-is-written-by-its-ai) — The Next Web, 2026-05
- [Cognition AI in new funding talks at $40B valuation](https://www.bloomberg.com/news/articles/2026-08-12/ai-startup-cognition-in-new-funding-talks-at-40-billion-value) — Bloomberg, 2026-08-12
- [Goldman Sachs Devin AI: 3-4x more productive than previous tools](https://en.cryptonomist.ch/2026/08/06/goldman-sachs-devin-ai/) — Cryptonomist, 2026-08-06
- [Goldman Sachs 12,000 technologists can operate like 14,400](https://www.ibm.com/think/news/goldman-sachs-first-ai-employee-devin) — IBM Think, 2026
- [Devin PR merge rate: 34% to 67% year over year](https://aiwiki.ai/wiki/devin) — AI Wiki
- [AI Coding Agents: Agent-First Architecture Beats IDE Tools](https://www.techtimes.com/articles/317354/20260529/ai-coding-agents-cognitions-26b-raise-bets-agent-first-architecture-beats-ide-tools.htm) — Tech Times, 2026-05-29
- [Citi: 2x to 20x productivity improvements for specific tasks using autonomous agents](https://lucidate.substack.com/p/goldman-sachs-scales-ai-coding-to) — Lucidate, 2026-08
