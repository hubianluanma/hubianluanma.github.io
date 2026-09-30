+++
date = '2026-09-30T07:32:55+08:00'
draft = false
title = 'Go 和 Rust 的 API 设计哲学，在 2026 年走向了不同的极端'
description = "Go 1.24 泛型别名正式落地，Rust async 闭包进入 2024 edition。两者都在解决「类型系统如何服务开发者」的问题，但路径截然不同——这背后是两种工程哲学的根本分歧。"
tags = ["编程", "技术", "Go", "Rust"]
categories = ["编程技术"]
author = "Spiral"
+++

2026 年，Go 和 Rust 都在语言演进上迈出了关键一步。Go 1.24 让泛型别名（generic type aliases）正式成为语言的一部分；Rust 则在 2024 edition 里把 async 闭包（async closures）扶正为一级公民。表面上看都是「补全语法」，但两者解决的是完全不同的问题，背后折射的工程哲学也截然不同。

## 泛型别名：Go 的克制哲学终于松口

Go 的泛型之路走得异常漫长。从 2022 年 Go 1.18 引入泛型开始，社区等了整整七个小版本，才在 Go 1.24（2025 年 2 月）等到泛型别名完整落地。

这不奇怪。Go 团队对语言特性的态度一贯是「没有充分理由就不加」。泛型别名的支持从 Go 1.23 开始以 `GOEXPERIMENT=aliastypeparams` 的形式提供实验性支持，Go 1.24 才默认开启。这个节奏本身就是信息：Go 团队在泛型上的谨慎，不是因为技术实现有多难，而是因为他们需要时间确认「加了之后对整个生态是好是坏」。

泛型别名解决的是 Go 开发者长期面临的一个具体问题：类型别名不能带类型参数，导致在重构泛型代码时不得不直接修改原类型。使用方式很直接：

```go
// 定义一个泛型别名
type ComparableVector[T comparable] = Vector[T]

// 使用
var cv ComparableVector[int]
```

这个功能的实际价值在于**渐进式重构**。当一个库需要把某个具体类型改成泛型时，泛型别名让调用方可以逐步迁移，而不必在同一时刻全部修改。没有这个功能，库作者只能在「破坏性变更」和「维护两份代码」之间二选一。

但值得注意的是，Go 1.24 的泛型别名有一个限制：`GOEXPERIMENT=noaliastypeparams` 在 Go 1.25 会被移除，意味着这个特性不会有永久的开关。这和 Go 对泛型的整体态度一致——既然加了，就稳定下来，不要留实验性的尾巴。

## Rust async：从「能用」到「好用」的三年

Rust 的 async 故事则是另一条轨迹。Rust 1.39（2019 年）引入 `async/await` 语法，但早期的 async fn 不能直接写在 trait 定义里，必须依赖 `async-trait` proc-macro 这个社区方案——代价是每次调用都会产生一次堆分配。

这个限制持续了四年。Rust 1.75（2023 年 12 月）终于将 async fn in traits 原生化，编译器直接生成 Future，零堆分配。Rust 1.85（2024 edition）更进一步，引入 `AsyncFn`/`AsyncFnMut`/`AsyncFnOnce` 三个 trait，让 async 闭包成为语言的一等公民。

```rust
// 旧的写法：两个泛型参数，无法表达高阶生命周期
fn old<F, Fut>(f: F)
where F: Fn(&str) -> Fut, Fut: Future<Output = String> { ... }

// 2024 edition：干净正确，闭包可以直接捕获环境
async fn new(callback: impl async Fn(&str) -> String) {
    let local = String::from("hello");
    let result = callback(&local).await; // 闭包捕获 local — 正常工作
}
```

Rust 2024 edition 还修复了 lifetime capture 的规则，让 unsafe 代码的 ergonomics 更安全。这和 Go 的路径形成鲜明对比：Go 在「加」上极为克制，而 Rust 在「改」上更为激进——通过 edition 机制主动清理历史包袱，即使这意味着破坏兼容性。

## 两个故事，同一个问题

Go 和 Rust 在 2026 年的演进，其实都在回答同一个问题：**类型系统应该为代码的可维护性服务到什么程度？**

Go 的回答是：类型系统是工具，不是目的。泛型别名是为了让重构更安全，让库作者不必在破坏性变更前犹豫不决。它的价值不在于「表达能力更强」，而在于「降低改代码的风险」。Go 团队的核心逻辑是：**工程上的渐进式改进，比语言层面的宏大设计更重要**。

Rust 的回答则是：类型系统是合约。async 闭包和 async fn in traits 把「这个函数是异步的」这件事变成了类型系统的第一等公民，让编译器能在编译期就捕获异步代码的错误。Rust 团队的核心逻辑是：**如果一个概念在语义上有意义，它就应该在类型层面被表达出来**。

这两种哲学没有高下之分，适用场景不同。Go 的目标是工程可靠性，Rust 的目标是正确性保证。当你在写云原生基础设施、微服务网关这类需要快速迭代、向后兼容要求高的系统时，Go 的克制是优点。当你需要构建高性能库、并发框架、需要编译器强制保证合约的系统时，Rust 的激进是优点。

## 接下来看什么

2026 年剩下的时间和 2027 年初，有几个值得关注的动向：

**Go 方向**

- Go 1.27 传闻会有 generic methods（泛型方法），即直接在类型上定义泛型方法，而不是通过泛型 struct/class 中转。如果落地，Go 的泛型版图将基本完整。
- `go fix` 命令的 codemod 能力在 1.26 之后会更强大，大规模代码迁移的成本会进一步降低。

**Rust 方向**

- Async iterators via gen blocks 仍在路线图上，但被 next-generation trait solver 的进度卡住了。如果 2027 年有突破，async Rust 的 ergonomics 会再上一个台阶。
- Pin ergonomics 的改善（让 self-referential futures 不那么痛苦）目前没有明确时间表，但这是一个影响大量现有 async 代码的重大改进。

**两者共同的竞争区间**

- WebAssembly 是 Rust 和 Go 都在渗透的战场。Rust 的 `wasm-bindgen` 和 Go 的 `GOOS=js GOARCH=wasm` 两条路径都在进化，2027 年这个领域可能会有新的格局变化。

---

## 参考资料

- [Go 1.24 Release Notes](https://go.dev/doc/go1.24)
- [Go 1.24 is released](https://go.dev/blog/go1.24)
- [What's in an (Alias) Name?](https://go.dev/blog/alias-names)
- [Rust 1.86.0 Announcement](https://blog.rust-lang.org/2025/04/03/Rust-1.86.0)
- [Modern Rust Features Guide (2023-2026)](https://github.com/wyattgill9/knowledge-base/blob/main/Wiki/Pages/modern-rust-features.md)
- [Rust Async Evolution](https://github.com/wyattgill9/knowledge-base/blob/main/Wiki/Pages/rust-async-evolution.md)
- [Codewars: Nine Language Runtime Updates](https://www.codewars.com/post/whats-new-on-codewars-nine-language-runtime-updates)
