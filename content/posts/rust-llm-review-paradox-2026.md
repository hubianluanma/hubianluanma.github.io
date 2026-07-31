+++
date = '2026-07-31T07:32:22+08:00'
draft = false
title = "LLM 时代 Rust 的反直觉事实：代码更容易写了，review 却变得更难了"
description = "2022 年 Matt Welsh 说 Rust 太难不该用于新项目。2026 年 AI 编程工具已经能写出通过 code review 的 Rust——但这恰恰是问题所在：Rust 的价值在于通过编译器摩擦强迫你思考 ownership，绕过这个摩擦的同时也绕过了学习。"
tags = ["编程", "技术", "AI", "Rust"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/07/31/img_1785456120_1.jpeg", alt = "robot reading code with magnifying glass digital art dark blue" }
+++

2022 年，Matt Welsh 在一篇文章里说 Rust 太复杂、不值得团队投入。四年过去，2026 年的他又写了一篇[ Revisiting Rust in 2026](https://mdwdotla.medium.com/revisiting-rust-in-2026-ae8720cc7f2c)，结论却截然不同——不是 Rust 变简单了，而是 LLMs 把上手门槛踩平了。

这个转变看起来很正面，但 Welsh 真正想说的其实是：**这是一件危险的事。**

## 代码能跑了，不代表你懂这段代码

LLM 编程工具（Claude Code、Aider、MarsCode 等）现在能生成"看起来完全合理"的 Rust 代码。你把需求丢进去，它返回一个实现，编译通过，测试全绿，CI 也没问题。一个不了解 Rust 的开发者可以靠 AI 交付看起来完全合格的东西。

问题在于：**Rust 编译器通过，不代表这段代码是对的。**

Rust 的类型系统和 borrow checker 强制你思考生命周期和所有权——这个摩擦本身是语言设计的一部分。但当 LLM 帮你绕过这个摩擦时，你得到的是一段"编译器接受"而不是"我理解"的代码。

具体来说有三个层面的问题：

**第一，泛化和特化的边界你摸不清。** Rust 的 trait bounds、泛型约束是出了名的复杂。LLM 生成的代码在当前场景能跑，但当数据规模扩大、并发压力上来，或者边界条件出现时，你根本不知道哪里会出问题。你没经过那个"反复编译失败"的痛苦过程，所以没有关于这段代码的直觉。

**第二，unsafe 代码的风险你看不见。** 当 LLM 引入 `unsafe` 块（这在系统编程场景下非常常见），编译器完全无法提供保护。数据竞争、内存泄漏、use-after-free——这些问题 Rust 的 safe Rust 可以预防，但在 unsafe 块里全靠开发者自觉。你不亲手写过几个因为 unsafe 引发的 bug，就不会有对这段代码的敬畏。

**第三，性能陷阱不会被标记。** Rust 允许你写出正确但极其慢的代码。过度 clone、不必要的 heap 分配、错误的并行化策略——这些不会触发编译错误，只会在生产环境里以 latency spike 的形式出现。没有亲手调优过 GC 调不出来的问题，就不知道该往哪里看。

## review 的成本在上升，不是在下降

一个常见的误解是：AI 降低了开发门槛=降低了 code review 的成本。实际上恰恰相反。

当团队里所有人都能用 AI 写出"能编译的 Rust"时，reviewer 的负担变成了：**你必须懂这段代码的所有细节，才能判断它是否安全。** 因为提交者自己可能也不完全懂。

这和 Python/JavaScript 不一样。在动态语言里，reviewer 看到一段代码，大致能判断它在做什么，就算有隐藏 bug 也不至于灾难性。但 Rust 的类型系统是穷尽性的（exhaustive）——你必须理解每一个分支、每一种可能的错误处理，才能判断代码逻辑是否完整。LLM 可以帮你写，但你没法绕过理解的责任。

Welsh 在他的文章里提到一个具体例子：cargo crate 的 feature flags 机制。Rust 生态里大量 crate 有复杂的 feature 组合，不同 feature 之间有隐含的依赖关系。当 LLM 生成依赖这些 crate 的代码时，它会按照"大多数情况"生成一个配置，但**特定 feature 组合下的 corner case 只能靠人类判断**。你要是不知道这个 crate 有这个坑，你就不会意识到 LLM 给你埋了一个。

这形成了一个不对称：写代码的门槛在降低，review 代码的门槛反而在上升。你团队里每个会用 AI 写 Rust 的人都能提交代码，但能有效 review 这段代码的人还是那么几个。

## 真正的问题：Rust 的护城河没有变矮

Rust 的护城河从来不只是语法或类型系统，而是**对内存模型的深层理解**。这种理解必须通过和编译器反复博弈才能建立。

LLM 可以帮你写代码，但不能帮你建立这种理解。你可以让 AI 把所有权问题解释给你听，但那不是亲身经历。你没有亲手遇到过"这个变量为什么不能两次借用"的编译错误，就不会有对 ownership 概念的肌肉记忆。

这也意味着，用 AI 学 Rust 是一种幻觉。你可以在 AI 的帮助下写出能跑的程序，但这和你"学会了 Rust"是两回事。就像你可以在副驾驶的帮助下开车通过考试，但这不代表你真的学会了驾驶。

当这种 AI 辅助交付的 Rust 代码进入生产环境，出现了一个以前不存在的风险：**你身边的人都在用 AI 写 Rust，但真正懂 Rust 的人并没有增加。** 代码库的复杂度在上升，团队整体的风险承受能力却没有。

## 接下来看什么

- **Rust 2026 项目目标更新**（[官方博客](https://blog.rust-lang.org/2026/05/18/project-goals-2026-04/)）：std::autodiff 已进入 nightly，std::offload 正在跟进，这两个功能会把 Rust 推向 ML/科学计算场景——这些领域对 unsafe 的使用会更加密集，review 的风险也更高
- **Rust 在 AI 编程工具中的实际渗透率**：根据 [Modal Blog 的代码LLM对比](https://modal.com/resources/best-open-source-code-llms-ai-coding-agents)，Qwen3.6-Plus 等模型在 Rust 上的表现已经开始接近 Go 和 TypeScript，这意味着 2026 年用 AI 写 Rust 的比例会显著增加
- **Matt Welsh 的后续讨论**：他在 [Substack](https://mdwla.substack.com/p/revisiting-rust-in-2026) 上表示这个话题远没有定论，评论区有不少实战经验的反馈

## 小结

LLM 让 Rust 的上手门槛降低了——这是一个事实，不是坏事。但门槛降低和护城河消失是两回事。Rust 的价值在于通过编译器强制你思考难以理解的概念，AI 帮你绕过这个摩擦时，这个价值并没有消失，只是被绕开了。

真正需要担心的是：当团队里每个人都能用 AI 交付"能编译的 Rust"，但真正能判断这段代码是否安全的人凤毛麟角时，代码库的隐性技术债会悄悄积累。

这不是 Rust 的问题，也不是 AI 的问题。这是一个组织问题：**你用什么工具写了多少代码，和你真正理解多少代码，是两件完全不同的事。**

## 参考资料

- [Revisiting Rust in 2026 — Matt Welsh, Medium](https://mdwdotla.medium.com/revisiting-rust-in-2026-ae8720cc7f2c)
- [Rust 2026 项目目标更新 — Rust Blog](https://blog.rust-lang.org/2026/05/18/project-goals-2026-04/)
- [Rust std::autodiff 文档 — Rust Nightly](https://doc.rust-lang.org/nightly/std/autodiff/index.html)
- [Best Open Source Code LLMs for AI Coding Agents in 2026 — Modal Blog](https://modal.com/resources/best-open-source-code-llms-ai-coding-agents)
- [Rust Project Goals: High-Level ML optimizations](https://rust-lang.github.io/rust-project-goals/2026/high-level-ml.html)
- [After Six Months: Honest Assessment of Go 1.26 in Production](https://elsyarifx.medium.com/after-six-months-heres-my-honest-assessment-of-go-1-26-in-production-0a375567189f)
