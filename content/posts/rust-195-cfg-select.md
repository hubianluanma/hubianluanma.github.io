+++
date = '2026-06-30T07:31:47+08:00'
draft = false
title = 'Rust 1.95 收编 cfg_if：标准库与生态的边界战争'
description = "Rust 1.95 将 cfg_select! 引入标准库，结束了 cfg_if 统治条件编译的时代近十年。这不只是 API 变化，而是 Rust 核心团队对生态控制权的一次宣示。"
tags = ["编程", "技术", "Rust"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/30/img_1782777726_1.jpeg", alt = "Rust gears and circuit pattern dark blue" }
+++

2026 年 4 月 16 日，Rust 1.95 稳定版发布。其中最值得注意的不是某个语言新特性，而是一个"元工程"变化：内置宏 `cfg_select!` 正式进入标准库，等效替代了维护了近十年的社区惯用方案 `cfg_if` crate。

这件事值得深究，因为它的影响远超技术层面。

## 背景：cfg_if 为何统治了近十年

`cfg_if` 是一个简单到几乎不值一提的 crate：它提供 `cfg_if!` 宏，让开发者可以写这样的结构：

```rust
cfg_if! {
    if #[cfg(target_os = "linux")] {
        // Linux 路径
    } else if #[cfg(target_os = "macos")] {
        // macOS 路径
    } else {
        // 默认路径
    }
}
```

没有它，在 Rust 里做跨平台条件编译就得写一堆 `#[cfg]` 属性散落在代码各处，或者依赖编译器内置但表达能力有限的 `cfg!` 宏。`cfg_if` 的核心价值在于**把多个 cfg 分支收敛到一个结构里**，避免嵌套的 `#[cfg_attr]` 地狱。

这个 crate 由谁来维护？**没有人正式维护**。它是 Rust 社区的公共遗产，GitHub 上没有官方所有者，属于"谁需要谁提 PR"的松散状态。这在 Rust 生态里并不罕见——很多基础设施 crate 的维护者负担沉重，但下游依赖极广，更改代价极高。

## 官方下场：标准库为什么现在收编

Rust 核心团队选择 1.95 这个时间点动手，有几个原因。

**MSRV 策略成熟是前提。** Rust 1.95 同时引入了 MSRV-aware resolver（Minimum Supported Rust Version），使得 crate 作者可以更精确地声明自己兼容的 Rust 版本。这意味着内置 `cfg_select!` 不再需要在"给新用户更好体验"和"保持旧版本兼容"之间二选一——编译器本身解决了这个矛盾。

**cfg_if 的语义缺陷长期被忽视。** `cfg_if` 本质上只能处理 item 位置（函数、结构体、全局变量），不能处理表达式级别的条件。`cfg_select!` 在 std 里的实现则没有这个限制——它可以在表达式位置使用：

```rust
let val = cfg_select! {
    if #[cfg(debug_assertions)] { "debug" }
    else if #[cfg(release)] { "release" }
    else { "unknown" }
};
```

这是 `cfg_if` 永远做不到的事。Rust 1.95 的 `cfg_select!` 也不只是"更强的 cfg_if"，它的语义更接近一个编译期 match——先判定条件优先级再选中分支，比 `cfg_if` 的链式 if-else 更清晰。

**嵌入式社区的阻力说明它不是小事。** Rust 1.95 的发布记录里提到一个 breaking change：destabilized JSON target specs 影响了一部分嵌入式开发者。对这批用户来说，`cfg_select!` 进入 std 意味着他们依赖的某些 `target-spec.json` 行为在新版本下不再 work——这是官方用 std 接管生态控制权必须付出的代价。

## 反直觉视角：这不是"进步"，是"收编"

如果只看表面，内置 `cfg_select!` 显然是好事：标准库接管高频工具，减少外部依赖，加快编译速度。但从生态权力结构的角度，这件事有不同的读法。

**Rust 的 std 在 2026 年之前实际上非常克制。** 很长时间里 Rust 的哲学是"std 只提供必要的基础设施，剩下的交给生态"。这造就了丰富的 crate 生态，但也带来了碎片化问题：同一个问题有 N 个 crate 解决方案，MSRV 不兼容导致依赖地狱，crate 作者的维护负担重到宁愿 archived。

核心团队现在的动作表明他们不再接受这种状态。`cfg_select!` 进 std，本质上是把一个"已经事实标准化"的工具"官方认证"，同时让那些依赖 `cfg_if` 但没有资源维护的 crate 可以开始考虑迁移路径——往好了说是ecosystem maturity，往坏了说是生态控制权的集中化。

这和 Python 社区 2026 年的一个变化可以对照：Astral 的 uv/Ruff/ty 三件套已经让"Python 工具链"这个原本属于 pip/setuptools/pylint/flake8/mypy 分裂地带的领域，在 2026 年实现了事实统一（参见 Astral Python tooling 2026 分析）。Rust std 这次收编 `cfg_if`，是同一个逻辑：谁有资源持续维护，谁就来接管。

## 对开发者意味着什么（实用角度）

**立即影响：**
- 如果你的项目依赖 `cfg_if`，1.95 之后可以考虑迁移到 `cfg_select!`
- 如果你维护一个工具链 crate，MSRV-aware resolver 让你的用户升级 Rust 版本更容易了
- 标准库版本的 `cfg_select!` 编译速度比 `cfg_if` 快（没有外部依赖解析开销）

**中期影响：**
- `cfg_if` crate 不会消失，但新项目会越来越多选择 std 版本
- Rust 嵌入式生态（对 target-specs 敏感的部分）需要在新版本下重新验证兼容性
- 类似的"官方收编"可能会发生在其他高频社区 crate 上——下一个可能是 `once_cell` 或 `anyhow` 的某种变体

**长期看：**
- Rust 语言本身的演进速度在加快。2024-2025 年期间 std 引入的新 API 数量是 2019-2021 年的三倍以上。cfg_select! 只是其中一步。
- 生态集中化的代价是灵活性下降。如果某个标准库 API 的设计决策有争议，开发者没有"换一个 crate"的选择了——只能参与语言 RFC 流程。这对 Rust 这样的社区驱动语言是结构性转变。

## 参考资料

- Rust 1.95 Release Notes: https://pbxscience.com/rust-1-95-released-new-macro-smarter-match-guards-and-a-breaking-change-for-embedded-developers/
- cfg_select! 宏文档: https://doc.rust-lang.org/std/macro.cfg_select.html
- Rust conditional compilation reference: https://doc.rust-lang.org/reference/conditional-compilation.html
- Reddit PSA on Rust 1.95 cfg_select: https://www.reddit.com/r/rust/comments/1tah93l/psa_rust_195_stabilized_the_cfg_select_macro/
