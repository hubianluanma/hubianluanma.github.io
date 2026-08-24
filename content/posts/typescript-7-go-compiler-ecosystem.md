+++
date = '2026-08-24T07:32:38+08:00'
draft = false
title = 'TypeScript 7 的两难：编译器快了10倍，生态却还在原地等7.1'
description = "TypeScript 7 用 Go 重写编译器，build 速度提升 8-12 倍，但 typescript-eslint、ts-jest、ts-morph 在发布当天就全部报废。微软赌的是 7.1 的稳定 API 能来得及接上——这个赌注还没开奖。"
tags = ["编程", "技术", "TypeScript", "编程语言"]
categories = ["编程技术"]
author = "Spiral"
+++

2026 年 7 月 8 日，微软发布了 TypeScript 7.0。最重要的变化藏在水面下：整个编译器从 TypeScript/Node.js 重写成了 Go 原生代码。官方数字是 8 到 12 倍的速度提升——这对大型代码库来说是真实体感，VS Code 团队在内部迁移中率先尝到了甜头。

但与此同时，TypeScript 7 发布的第一天，ESLint 就崩了。

## 速度从哪来

TypeScript 7 的性能收益来自两个层面。

首先是 Go 带来的原生执行效率。旧编译器用 TypeScript 写就，跑在 Node.js 上——每次类型检查都是 JavaScript 运行时里的一次解释执行。新编译器是编译好的 Go 二进制，不经过中间层。

其次是共享内存并行化。Go 的 goroutine 加共享内存模型让编译器可以把整个项目的类型检查拆到多个线程同时跑。TypeScript 6 及之前是单线程的，单核性能再强也受限于一条流水线。官方博客说 TypeScript 7 "often about 10 times faster than TypeScript 6.0"，不是营销数字，是内部迁移后在真实代码库上测出来的结果。

> 这是一次架构平移，不是功能重写。TypeScript 团队明确说过，新 Go 代码库是"methodically ported from existing implementation"，类型检查逻辑与 6.0 结构上完全一致。所以速度提升是确定的，语义行为改变是极小的。

## 生态在同一天断供

但性能收益和生态健康是两件事。

typescrit-eslint（让 ESLint 理解 TypeScript 的桥梁）在 TS7 发布当天就拒绝安装——npm 的 peer dependency 限制把 typescript@7 拦在门外，ERESOLVE 错误直接中断 CI。强行用 `--force` 绕过后，ESLint 在 typescript-estree 深层崩溃。typescript-eslint 在 GitHub 上开了 issue #12518，**当天就被关闭了，标记为 not planned**。

ts-jest 情况类似：只要 node_modules 里保留一个 6.x 的 TypeScript，它就能跑。但如果你想升级到 TypeScript 7，对不起，transformer 不兼容。

ts-morph——大量代码生成工具的底层依赖——同样产生静默错误输出：类型信息看似正常，实际已经错了。这类错误最难发现：CI 绿了，但类型保障已经失效。

Vue、Svelte、Astro 的模板类型检查器全部被阻断，因为它们依赖 TypeScript 的 programmatic API，而这个 API 在 7.0 里**还没有 stable 版本**。微软说 7.1 才有。

## 被关闭的 issue 背后

typescript-eslint 团队选择关闭而非修复，不是因为不作为，而是因为修复权不在他们手上。

TypeScript 作为编译器的 API 一直没有稳定版本——`ts.createSourceFile`、`ts.createProgram` 这套东西从未承诺过向前兼容。每次 TypeScript 升级，这些内部 API 都可能悄悄变。旧编译器是 TypeScript 写的，API 层面好歹是同一套类型系统；Go 重写之后，API 层面凭空出现了一个"这次先不做 stable"的空白地带。

换句话说：TypeScript 团队给生态挖了一个接口坑，然后告诉生态"等 7.1"。

## 反共识：这不是失误，是策略

有一种解读是微软这次重写搞砸了生态，是一次执行事故。但仔细看时间线会发现：微软在 Go rewrite 公开 Beta 时就已经知道 API 会断供，7.1 的 roadmap 也是同时公开的。

所以更准确的解读是：**微软在性能和兼容性之间做了明确的优先级排序，并且愿意让生态承担迁移期的成本。**

这个赌注有合理性。TypeScript 的核心价值是类型检查和 IDE 体验，这两件事都直接依赖编译器速度。如果 Go rewrite 把编译时间从 3 分钟压到 20 秒，对每一个开发者每一天的体验都是真实提升。ESLint 和 ts-jest 是工具链的一部分，重要但不紧急——可以等 7.1。

问题是，这个"等"的代价是由整个生态共同支付的。每个还在跑 ESLint 的团队都需要在 CI 里多维护一个 typescript@6 的兼容性路径，每个升级了 TS7 的项目都在用"半残"的类型检查假装一切正常。

这不是失误。这是一次以速度为单一目标的战略选择。微软赢了编译速度，生态在赢之前要先扛一段时间。

## 接下来看什么

- **TypeScript 7.1 稳定 API**：这是整个生态恢复的触发点。预计 2026 年 Q4，typescript-eslint、ts-jest 和各框架的模板检查器会陆续跟进
- **框架的实际迁移策略**：Vue/Astro/Svelte 团队是否会放弃对 TS7 的第一时间支持，要求用户降级或等待？
- **esbuild / SWC 的竞争格局变化**：TypeScript 官方编译器已经这么快了，第三方超高速转译工具的护城河还在吗？

---

## 参考资料

- [Announcing TypeScript 7.0 Beta - Microsoft TypeScript Blog](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0-beta/)（确认 Go rewrite 架构平移声明）
- [TypeScript 7.0 with a Native Go Compiler, Delivering 10x Faster Builds - InfoQ](https://www.infoq.com/news/2026/08/typescript-7-released/)（7.0 GA 日期与性能数据）
- [typescript-eslint Issue #12518 - GitHub](https://github.com/typescript-eslint/typescript-eslint/issues/12518)（ESLint 7 月 8 日当日被关闭）
- [TypeScript 7 Migration Readiness: ESLint/Astro Blockers - ecorpit.com](https://ecorpit.com/typescript-7-migration-readiness-eslint-astro-blockers-2026/)（生态断供详情与时间线）
- [TypeScript 7 Is Two Weeks Old. Here's What's Actually Breaking - byteiota.com](https://byteiota.com/typescript-7-go-compiler/)（各工具链崩溃的具体表现）
