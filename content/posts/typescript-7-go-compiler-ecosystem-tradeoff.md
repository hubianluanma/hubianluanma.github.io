+++
date = '2026-08-22T07:31:16+08:00'
draft = false
title = 'TypeScript 7 快了 10 倍，但你的 ESLint 和 ts-jest 可能正在求救'
description = "TypeScript 7.0 的 Go 重写把编译速度推到 10x，但 typescript-eslint、ts-jest、ts-morph 这三个绕不开的工具全部躺平——因为微软把 programmatic API 留给了 7.1。"
tags = ["编程", "技术", "TypeScript"]
categories = ["编程技术"]
author = "Spiral"
+++

7 月 8 日，微软正式发布 TypeScript 7.0（项目代号 Corsa）——一个用 Go 从零重写的编译器。数字很漂亮：VS Code 150 万行代码的类型检查从 77.8 秒降到 7.5 秒，大型代码库平均快 8 到 12 倍。

但这个数字背后藏着一个被大多数报道忽略的事实：**你的 ESLint、ts-jest 和 ts-morph 可能在这次升级后已经悄悄坏了。**

## 编译器快了，工具链断了

TypeScript 7 的架构转换幅度远超一次常规版本迭代。旧的 TypeScript 编译器用 TypeScript 本身编写，暴露了完整的 programmatic API，第三方工具通过这些 API 做类型感知的 lint、测试转换和 AST 操作。

Go 重写之后，这些 API 全部消失。微软明确表示：稳定的 programmatic API 要到 TypeScript 7.1 才会上线，预计 2026 年 10 月前后。

这直接导致三个现实问题：

**typescript-eslint**（类型感知 ESLint 规则的核心依赖）—— Issue #12518 于 7 月 8 日当天提交，次日被关闭，状态标注为"not planned"。维护者的解释很直白：这不是 eslint-typescript 的问题，修复不在他们这边。

**ts-jest**——如果把 ts-jest 指向 `@typescript/native-preview`，它会崩溃。症状不是安装失败，而是一串令人困惑的 transform 错误，实际原因是它调用的 Strada 函数在新编译器里根本不存在。临时解法：项目中保留一份 6.x 的 TypeScript 专门给 ts-jest 用。

**ts-morph** 和自定义 AST 转换器——一位开发者在 4 月的实测中测试了 15 个主流工具，发现 ts-morph 的每一个调用都映射到不存在的 Strada 函数，完全无法工作。

## 为什么说这是结构性的取舍，不是 bug

大多数报道把这件事描述为"过渡期的阵痛"，似乎 7.1 上线就能解决。但问题的结构要更深一层。

微软在选择 Go 重写时面临一个根本矛盾：想要编译速度，就要放弃 TypeScript 写的编译器带来的可扩展性。Go 是编译型语言，没有字节码层面的自省能力，天然不支持 TypeScript 编译器那种深度的类型计算 API。如果要完整保留 API，Go 重写的意义就折半了。

这不是技术能力问题，是产品策略选择。微软选了速度，代价由整个工具生态承担。

另一个被低估的问题是**锁死效应**。大量流行工具（Vue 的模板类型检查、Svelte 的编译增强、Astro 的类型注入）都依赖 TypeScript 的 programmatic API。一旦这个接口在 7.0 断开，它们没有办法临时打补丁——唯一的选择是等 7.1。

## 现实中的开发者处境

如果你今天在 VS Code 里跑 `tsgo --noEmit`，类型检查速度确实起飞了。但紧接着 `eslint --ext .ts` 可能会直接崩溃，或者 `jest` 跑测试时出现没有意义的报错。

一个被社区验证过的临时方案是**双轨 TypeScript**：项目里装两份 TypeScript，一份 7.x 给编辑器做 type-check，一份 6.x 专门给工具链用。但维护两套版本本身就是在积累技术债，而且不是每个工具都支持这种分离方式。

oxlint + oxlint-tsgolint 的组合是另一个方向（一个用 Go 重写的更快的 linter），但规则覆盖度和配置格式和 ESLint 完全不同，迁移成本不低。

## 接下来看什么

- **7.1 API 时间线**：微软说 10 月前后，但历史上 major version 的 API 发布时间经常跳票。如果 7.1 延期，整个工具链的空窗期会比预期更长。
- **ESLint 官方态度**：目前社区方案都是 workaround，ESLint 官方是否会提供内置桥接方案值得关注。
- **框架方跟进速度**：Astro、Vue、Svelte 这些重度依赖 TS API 的框架会不会推自己的 TS 版本锁定策略，或者直接切换到 oxlint 生态。
- **企业采纳节奏**：大型前端团队升级前会先等工具链稳定，这会拉长 7.0 的 adoption 曲线，6.x 的寿命可能比预期长。

TypeScript 7 的速度提升是真实的，方向也是正确的。但从 7.0 到真正可用的完整生态，中间还有几个月的过渡期。这段时间里，开发者实际上在用一套跑得飞快的新编译器，拖着三套还没跟上趟的旧工具——这是微软这次架构迁移的真实成本。

---

**参考资料**

- [TypeScript 7.0 Ships Stable: Corsa Native Go Rewrite Delivers 12x Build Speeds](https://www.ntcompatible.com/story/typescript-702-ships-stable-cors%D0%B0-native-go-rewrite-delivers-12x-build-speeds)
- [TypeScript 7.0 Is GA: The 10x Compiler Migration Playbook](https://www.digitalapplied.com/blog/typescript-7-0-ga-native-compiler-migration-playbook-2026)
- [Why Your TypeScript 7 Upgrade Broke ESLint, ts-jest, and ts-morph](https://dev.to/dev_encyclopedia/why-your-typescript-7-upgrade-broke-eslint-ts-jest-and-ts-morph-385k)
- [TypeScript 6.x to 7.0 Migration Guide](https://gist.github.com/nafiskabbo/01ccb4970515413076f3759486c39755)
- [What Breaks When You Upgrade to TypeScript 7 (tsgo)](https://medium.com/@krunalkanojiya/what-breaks-when-you-upgrade-to-typescript-7-tsgo-614005afbbd0)
