+++
date = '2026-09-01T07:32:27+08:00'
draft = false
title = 'TypeScript 7 快了 10 倍，但你的 ESLint 可能正在拖后腿'
description = "TypeScript 7 用 Go 重写编译器，type-check 提速 8-12 倍；但 typescript-eslint、ts-jest 等生态仍困在旧 API 上。与此同时，Rust 写的 Oxlint 正以 50-100 倍速度补位。两件事同时发生，把一个团队的工具链撕成了两个时代。"
tags = ["编程", "技术", "TypeScript", "工具链"]
categories = ["编程技术"]
author = "Spiral"
+++

2026 年 7 月 8 日，微软正式发布 TypeScript 7.0。编译器从 JavaScript/Node.js 彻底切换到 Go，代号 Project Corsa。官方数字很好看：VS Code 代码库全量类型检查从 125.7 秒降到 10.6 秒，11.9 倍；Slack 的 CI 类型检查从 ~7.5 分钟降到 ~1.25 分钟；Canva 首次错误提示从 58 秒降到 4.8 秒。

但这个数字只覆盖了 `tsc --noEmit` 本身。工具链的其他环节——ESLint、ts-jest、框架模板编译器——还在 TypeScript 6 上原地踏步。

## 编译器快了，工具链还在拖

TypeScript 7 的核心变化是 `tsgo`：一个用 Go 重写的原生二进制，不依赖 Node 运行时，不受 JavaScript 堆限制，可以真正用上 OS 多线程。这个设计目标很纯粹——让类型检查不要阻塞开发者等待。

但「类型检查」只是整个前端工具链的一环。现实中一个典型项目的 CI 流程是这样的：

```
代码提交 → lint (ESLint) → 类型检查 (tsc) → 单元测试 (ts-jest) → 构建
```

TypeScript 7 让第二步从 7.5 分钟变成 1.25 分钟。但第一步的 ESLint 呢？一个中等规模的 TypeScript monorepo，完整跑一遍 ESLint（含 typescript-eslint 的类型感知规则）通常需要 30-60 秒。这部分 TypeScript 7 完全帮不上忙。

原因很直接：TypeScript 7 移除了稳定的 programmatic API（内部叫 Strada API）。这个 API 正是 typescript-eslint、ts-jest、ts-morph 以及 Vue/Svelte/Astro 模板编译器赖以运行的基础接口。没有它，这些工具只能继续用 TypeScript 6。

typescript-eslint 团队在发布日当天就收到了报告：npm 拒绝在 TypeScript 7 旁边安装 typescript-eslint，因为 peer dependency 范围卡在 `<6.1.0`。强制安装后，ESLint 直接崩溃，堆栈里是 `TypeError: Cannot read properties of undefined (reading 'Cjs')`。

这不是 typescript-eslint 的 bug，是 TypeScript 7 主动不提供这个接口。微软的回应是：7.1 会补上，预计 2026 年 10-11 月。

## Oxlint：Rust 在 linting 层的降维打击

就在 TypeScript 7 的工具链空窗期，Rust 写的 Oxlint 正以 50-100 倍的速度差切入。

Oxlint 来自 Oxc 项目（由 Vite 作者 Evan You 资助），不是 ESLint 的替代品，而是一个前置过滤层。它的核心逻辑：

> 先用 Oxlint 跑语法级规则（0.4-1 秒扫完整个 monorepo），再用 ESLint 只跑类型感知规则。

Vercel 团队在 2026 年已经这样跑了：Oxlint 在 CI 里 1 秒内跑完，ESLint 只处理 Oxlint 覆盖不了的 type-aware 规则。完整 lint 管道从 81 秒降到 2.5 秒，其中 lint 本身是 0.7 秒。

对比数字，在一个约 1200 个 TypeScript 文件、85000 行代码的 Next.js 项目上：

- ESLint（完整配置：typescript-eslint + react + import + tailwindcss）：冷启动 ~45 秒，热启动 ~38 秒
- ESLint（仅类型感知规则，无其他插件）：冷启动 ~12 秒，热启动 ~8 秒
- Oxlint（等价的非类型感知规则）：冷启动 0.4 秒，热启动 0.4 秒

Node.js 自身仓库（6298 个文件）的对比是：ESLint 1 分 43 秒，Oxlint 21 秒，4.8 倍差距。在你自己的机器上感受会更明显——ESLint 30-60 秒的等待足够让开发者跳过本地运行，直接推 CI。

Oxlint 目前有 700+ 条规则，覆盖了 typescript-eslint 61 条类型感知规则中的 59 条。剩余 2 条是 `no-floating-promises` 和 `no-misused-promises`——这两条确实是类型检查才有的能力，Oxlint 无法替代。

## 一个团队的两条时间线

这造成了一个很现实的问题：TypeScript 7 的提速和工具链生态的跟进不在同一条时间线上。

一个具体团队的 2026 下半年可能是这样的：

- **现在（7-9 月）**：tsgo 单独跑 type-check，10 倍提速在 CI 和 editor 里已经生效。ESLint 继续用 TypeScript 6.x，两个编译器共存。
- **10-11 月（预计）**：TypeScript 7.1 发布，programmatic API 补上，typescript-eslint 可能跟进。
- **更远**：Oxlint 的 type-aware 规则覆盖率能否到 100%，决定了团队是否可以彻底卸下 ESLint。

这不是「升级 vs 不升级」的选择，而是「两个时代并行」的状态。一个 TypeScript 项目里同时跑着 Go 写的 tsgo（2026）和 JavaScript 写的 ESLint（2013），这在技术选型史上很少见。

## 实际操作建议

**如果你的项目今天就跑在 TypeScript 7 上**：

把 tsgo（`@typescript/native-preview`）作为独立的 type-check 工具安装，不要卸载 TypeScript 6。两套并存，TypeScript 6 只负责给 ESLint 和框架模板编译器用。

```bash
npm install -D @typescript/native-preview
npx tsgo --noEmit  # 快速类型检查
```

在 monorepo 的 package.json 里可以这样分工：

```json
{
  "scripts": {
    "typecheck": "tsgo --noEmit",
    "lint": "eslint .",
    "test": "jest"
  }
}
```

**如果你的 ESLint 已经跑得很慢**：

先跑一遍 `@oxlint/migrate`，它能自动把 ESLint 配置转成 Oxlint 配置，然后按 Vercel 的模式并行跑两个 linter：Oxlint 过语法关，ESLint 补 type-aware 规则。

```bash
npx @oxlint/migrate
# 生成 .oxlintrc.json
# ESLint 继续处理类型感知规则
```

**如果你的框架是 Vue / Svelte / Astro**：

模板编译器依赖 Volar，Volar 依赖 TypeScript programmatic API。TypeScript 7 短期内不会让这些框架的模板类型检查变快，这个状态会持续到 7.1 发布之后。这是框架侧的限制，不是项目配置问题，不要花时间在 tsconfig 里调参数。

## 接下来看什么

- **TypeScript 7.1 的发布时间**：如果微软真的在 10-11 月交付 programmatic API，typescript-eslint 和 ts-jest 的 7.x 兼容版会快速跟进，工具链碎片化会开始收拢。
- **Oxlint 的 type-aware 规则覆盖进度**：目前 59/61，两条 missing rules 的优先级决定了完整迁移路径什么时候打通。
- **Biome 的动向**：Biome 同时做 lint + format，体积比 ESLint + Prettier 小很多。如果它的 type-aware linting 追上 Oxlint，单一工具替换的收益会更大。

TypeScript 7 的编译器提速是真实的，但工具链的现代化是另一条独立的时间线。两件事不矛盾，只是节奏不同。搞清楚自己在哪条线上，才知道该等还是该动。

## 参考资料

- [TypeScript 7.0 is GA: a 10x compiler to adopt in stages (2026)](https://ecorpit.com/typescript-7-migration-readiness-eslint-astro-blockers-2026)
- [TypeScript 7 Is Here, and My Project Still Can't Use It](https://anowar.dev/blog/typescript-70-released-10x-faster-native-port-in-go)
- [Why Your TypeScript 7 Upgrade Broke ESLint, ts-jest, and ts-morph](https://devencyclopedia.com/blog/typescript-7-broke-eslint-ts-jest-ts-morph)
- [Oxlint vs ESLint: Rust-Powered Linting Performance 2026](https://pkgpulse.com/guides/oxlint-vs-eslint-rust-linting-performance-2026)
- [Oxc Is Quietly Building the Fastest JavaScript Toolchain in Rust](https://typescript.news/articles/2026-04-05-oxc-rust-javascript-toolchain-benchmarks)
- [Biome vs ESLint vs Oxlint: Best JavaScript Linter (2026)](https://trybuildpilot.com/840-biomejs-vs-eslint-vs-oxlint-2026)
