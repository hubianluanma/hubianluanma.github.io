+++
date = '2026-08-08T07:32:00+08:00'
draft = false
title = 'Rust 2026 安全关键计算三路并进：Polonius、autodiff 与 Ferrocene 各有各的难产'
description = "Rust 正在从「内存安全语言」进化为「可证明安全语言」，但 Polonius Alpha 上 nightly、std::autodiff 卡在 Apple CI、Ferrocene 拿到 ISO 26262 认证三条线各自面临工程难题，2027 edition 前夜的 Rust 其实比外界看到的更分裂。"
tags = ["编程", "技术"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/08/08/img_1786147454_1.jpeg", alt = "Rust safety critical computing abstract visualization" }
+++

Rust 在 2026 年的主线不再是"学不学 Rust"，而是另一件事：它能不能从「内存安全语言」进化成「可证明安全的语言」。这个目标拆成了三条技术线——Polonius 下一代借用检查器、std::autodiff 自动微分、Ferrocene 安全关键认证——每一条都宣称要改变 Rust 的用途，但每一条都还没完全兑现。这篇文章不是 Rust 年度总结，而是一次实情检查。

## Polonius：比 NLL 更聪明的借用检查器，8 月刚上 nightly

Rust 现有的借用检查器诞生于非词法生命周期（NLL）改革，那次改动让 Rust 能接受更多合理的代码，但依然保守。当前的检查器会拒绝一些实际安全的代码，因为它的分析粒度是按控制流块来的，而不是按变量的实际使用范围。

Polonius 是新一代基于可达性的借用检查器，核心改进是**位置敏感**（location-sensitive）：它追踪每笔借款（loan）在代码中的精确活跃范围，而不是按词法块粗粒度判断。效果是一些被误报为"使用了已移动值"的代码在新检查器下可以通过，而编译器不需要做任何运行时修改。

2026 年 8 月 4 日，Rust 官方博客宣布 Polonius Alpha 正式在 nightly 启用，目标是「未来几个月内」稳定化。这意味着 Rust 1.97（预计 2027 年）有可能正式带 Polonius 上岸。

但"Alpha"的意思是它还没完全就绪。官方文档明确说 Polonius 只完成了完整借用检查分析的一部分，目前 nightly 版本有 10-20% 的额外编译时间开销，且在某些场景下会给出不同结论。Rust 团队愿意接受这个开销换表达力，但建议不要在生产环境开 -Zpolonius。

**反共识角度**：Polonius 不是性能优化，是**认知负担下放**。现在的 Rust 开发者需要懂"哪些模式会让 borrow checker 生气"，Polonius 减少了这个认知负担，代价是把复杂度转移到了编译器维护者身上。对 Rust 语言本身的成熟度是好事，对已经熟悉现有规则的开发者来说收益有限。

## std::autodiff：Rust 进军科学计算的第一步，卡在 Apple CI

std::autodiff 是 Rust 项目目标之一，目的是把 Enzyme（基于 LLVM 的自动微分库）集成进标准库，让 Rust 也能像 PyTorch/JAX 一样计算梯度——不需要手动写反向传播，编译器自动生成。

这个特性的状态在 2026 年 5 月的官方更新里有详细说明：Linux 上已经能跑通，nightly CI 已通过，开发者也验证了安装后能正常工作。但 Apple 平台出了问题——rustc 和 Enzyme 各自带了独立的 LLVM 副本，导致 autodiff 调用时进程挂起。解决方案是在编译时合并这两个副本，PR 已经在审查中。

std::offload（GPU 卸载）的进度更慢，要等 rustc 升级到 LLVM 22 才能继续。官方原话是"comparably easy to get rid of the second one"——听起来简单，但 LLVM 版本升级从来不是简单的事。

**这条线的意义**：std::autodiff 的目标是让 Rust 成为一种适合 ML 编译器和科学计算的语言，而不只是系统编程。目前 Rust 在 ML 领域的存在感基本靠 tch-rs（PyTorch Rust binding）在撑，标准库级别的 autodiff 才是真正改变局面的东西。但 2026 年上半年卡在 Apple CI 这个事实说明——Rust 团队在系统级工程上的积累很厚，但一到跨平台 GPU/ML 这个交叉地带，工程厚度就薄了。

## Ferrocene：安全关键认证，Rust 的商业化出口

Ferrocene 是 Rust 基金会在安全关键领域的商业化出口，由 Ferrous Systems 维护，本质上是一套经过认证的 Rust 工具链变体，针对汽车（ISO 26262 ASIL B/D）、医疗（IEC 62304 Class C）、工业（IEC 61508 SIL 3/4）、航空（DO-178C）等场景做了资质认证。

2026 年 2 月发布的 Ferrocene 26.02.0 增加了 ISO 26262 ASIL B 认证的 Rust 核心子集支持。2026 年 1 月 Rust 官方博客专门发了一篇《What does it take to ship Rust in safety-critical?》，其中提到 Ferrocene 语言规范（FLS）已经从行业实践进化为 Rust 项目的正式规范文档，有独立团队维护——这意味着 Rust 官方正式承认了安全关键领域需要独立的标准文本，而不只是"我们用 Rust 就行"。

**这条线的核心矛盾**：Ferrocene 认证的 Rust 核心是一个**受限子集**，不是所有 Rust 特性都能用。async/await、泛型关联类型、某些宏系统在高安全认证场景下有使用限制。换句话说，越是想把 Rust 用在最高安全等级的场景，就越需要接受一个「功能更少但可证明的 Rust」。这和 Rust 社区追求表达力的方向是相悖的。

## 三条线指向同一个问题：Rust 的边界在哪里

这三条技术线有一个共同叙事框架——Rust 正在从"写操作系统和浏览器组件"扩展到"写可验证的、科学计算的、安全关键的"代码。这个叙事是真实的，但背后有一个还没解决的问题：**Rust 能走多远，取决于社区愿意在哪些方向上接受约束。**

Polonius 要接受 10-20% 的编译时间开销。autodiff 要接受跨平台 LLVM 版本管理的复杂性。Ferrocene 要接受一个功能受限的 Rust 子集。每一条线都在用不同的方式告诉 Rust 社区：扩展语言能力的边界，代价不是均匀分布的。

对已经在用 Rust 的开发者来说，2026 年不是"学 Rust 的好时机"，而是"观察 Rust 能不能在科学计算和安全关键领域证明自己"的年份。如果这三条线有任何一条在 2027 edition 之前稳定落地，Rust 的使用范围会真正打开。如果都继续难产，Rust 就会被困在"很好但只适合写系统组件"的天花板里。

## 接下来看什么

- **Polonius 稳定化时间表**：关注 Rust 1.97 的 release notes，如果 Polonius 正式 stable，说明 Rust 借用了十年的「精确借用检查」目标终于兑现
- **autodiff 的 Apple 支持**：PR merge 后首次在 macOS 上跑通 autodiff 示例，是 std::autodiff 可用的前置条件
- **Ferrocene 的 ASIL D 认证进度**：ASIL B 是量产车规，ASIL D 是最高等级认证，能拿到意味着 Rust 可以进制动系统和转向控制
- **Rust 2027 Edition 特性冻结**：2027 edition 的 RFC 窗口通常在 2026 年底关闭，届时能看到有哪些新特性会被接受——这会直接影响 Rust 未来三年的走向

## 参考资料

- [Announcing Rust Polonius Alpha on Nightly](https://blog.rust-lang.org/2026/08/04/enabling-polonius-alpha-on-nightly/)
- [Rust Project Goals: Polonius Alpha Stabilization](https://rust-lang.github.io/rust-project-goals/2026/polonius.html)
- [Rust Project Goals: High-Level ML optimizations (autodiff/offload)](https://rust-lang.github.io/rust-project-goals/2026/high-level-ml.html)
- [Rust Blog: What does it take to ship Rust in safety-critical?](https://blog.rust-lang.org/2026/01/14/what-does-it-take-to-ship-rust-in-safety-critical/)
- [Ferrocene 26.02.0: ISO 26262 ASIL B Certification](https://www.eenewseurope.com/en/ferrocene-26-02-0-automotive-core-certification/)
- [Rust 2027 Edition Preparation](https://internals.rust-lang.org/t/we-have-only-two-years-to-prepare-the-rust-2027-edition/22494)
