+++
date = '2026-06-13T08:00:00+08:00'
draft = false
title = 'GitHub Copilot 改token计费：一场开发者用脚投票的价格革命'
description = '2026年6月1日，GitHub Copilot正式切换为token计费模式。开发者账单暴涨10到50倍，Cursor和Claude Code趁机收割市场。这不只是一次简单的涨价，而是一次关于「AI编程工具究竟该卖什么」的路线之争。'
tags = ["编程", "AI工具", "GitHub", "开发者"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/13/img_1781308902_1.jpeg", alt = "a developer at a desk with floating tokens and credit meters, data streams visualized as coins" }
+++

6月1日，GitHub Copilot正式从"无限次请求"切换为"token计量"计费模式。同一天，科技圈炸锅——不是庆祝，是控诉。

Reddit上有人贴出账单：从每月29美元飙到750美元。TechCrunch的标题毫不客气：**"What a joke"**。开发者社区的讨论帖在48小时内涌入上万条回复，矛头直指微软的定价策略。

这不是孤立的个案。这是一场已经发生的路线分裂。

## 事情是怎么变味的

先说事实。GitHub在4月宣布了这次调整，6月1日正式生效。官方说辞很平滑：**"Copilot Pro保持10美元/月，Pro+保持39美元/月，Business保持19美元/月"**——听起来没涨。

问题出在计量单位换成了AI Credits：**1 credit = 0.01美元**。每个订阅档位送你对应额度的credits，用完就得买更多。

这意味着什么？重度用户（尤其是开启Agent模式、让Copilot写整个文件或做代码重构的那批人）在旧模式下不受限制，在新模式下每个token都在烧钱。一位开发者在Reddit贴出的实际使用记录显示，他一天就烧掉了原来一个月的配额。

这不是价格调整。这是产品形态的根本转变——从**工程工具**变成了**云服务**。

## 被量化的，不只是代码

云服务的定价逻辑本质上是"用多少付多少"。这对基础设施没问题，对LLM API没问题。但把它套在一个IDE插件上，问题就复杂了。

编程行为是高度碎片化的。一个开发者可能花两分钟写一个函数，然后花半小时读文档、跑测试、开会。Copilot的价值不是按token计量的——它的价值在于"在你思考的时候，代码补全恰好出现"。但token计费把这种非线性价值强行线性化了。

这就催生了一个讽刺的现象：**用得越多，单位价值反而越低**。因为Copilot最擅长的"小而频繁的补全"在新计费模式下反而最费credits。一个重度用户给科技媒体算的账是：每天Agent模式用下来，月账单轻松突破500美元。

## 三足鼎立：格局在72小时内重塑

Copilot改制的受益者名单很短，但都很响亮。

**Cursor**——AI原生IDE，成立于2023年，在这次事件前就已经是"愿意为体验付溢价"的开发者首选。它的定价是20美元/月，不计量，定位清晰：重度AI编程体验。改制消息出来后，Cursor的waitlist在72小时内翻了三倍。

**Claude Code**——Anthropic推出的终端编程工具，20美元/月，同样不计量。TechCrunch在报道中直接推荐它作为"Copilot Agent模式的直接替代品"。开发者社区里，一股从Copilot迁出、用Claude Code做agentic work的潮流已经开始。

**DeepSeek**——以价格屠夫姿态切入，token价格远低于主流厂商。适合对成本敏感、有大量代码生成需求的场景。

还有一批开发者选择了**混搭策略**：保留Copilot Pro（10美元/月）用于基础补全和Next Edit Suggestions，额外订阅Cursor或Claude Code处理复杂对话和Agent任务。总成本27到30美元，但两份工具各司其职，不再有计量焦虑。

这个混搭方案的流行说明了一个关键问题：**开发者不是付不起钱，是不愿意为无法预测的用量付钱**。

## 反共识：这个困局本质上是Copilot的成功

翻看社区讨论，有一个声音很少被放大，但它其实最值得深思：

> Copilot的token计费，是微软在**给Copilot的过度使用擦屁股**。

什么意思？Copilot最初的设计场景是"补全你正在写的代码"——一行、两行、三行。但过去两年，随着Agent模式推出，开发者开始用它写整个文件、做代码审查、生成commit message、甚至做代码重构。这些行为模式已经超出了补全工具的范畴，变成了真正的AI coding agent。

Copilot没有为此重新定价——它只是把老价格改成了计量制。所以与其说是"涨价"，不如说是**终于对实际使用量收费了**。

但问题在于：开发者已经形成了依赖，生态已经围绕Copilot构建。突然计量，等于打断了已经运转的工作流。

这不是商业策略的失误，这是产品定位和用户行为之间的错位——而且错位已经持续了至少18个月。

## 接下来看什么

1. **微软是否会调整credits上限或推出企业级 unlimited 方案**：目前的Pro+（39美元/月）含39credits，对于重度用户来说只够用几天。企业订阅是微软真正的现金牛，这块的政策走向会决定有多少公司选择续签。

2. **Cursor和Claude Code的用户留存**：这两者目前都是flat fee，但如果用户量级上来，定价压力会显现。尤其是Anthropic如果也推token计费，Claude Code的flat fee能撑多久是未知数。

3. **GitHub Copilot Enterprise的政策走向**：Business订阅目前是19美元/席位/月，含19credits。如果企业用户集体反馈账单失控，微软必须回应——否则就是给Cursor和Claude Code送客户。

4. **开源替代方案的窗口期**：这次事件给CodeGen、CodeLlama等开源工具的本地部署方案创造了一个明确的市场窗口。对于隐私敏感或有成本控制需求的企业，自托管AI编程工具从未像现在这样有吸引力。

## 参考资料

- GitHub Blog: [GitHub Copilot is moving to usage-based billing](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)
- TechCrunch: ["What a joke': GitHub Copilot's new token-based billing spurs consternation among devs"](https://techcrunch.com/2026/05/30/what-a-joke-github-copilots-new-token-based-billing-spurs-consternation-among-devs/)
- TechJournal: [GitHub Copilot Token Billing Starts Today: Devs Report 10x-50x Cost Increases](https://techjournal.org/github-copilot-token-billing-backlash)
- UsageBox: [GitHub Copilot AI Credits 2026: Pricing, Quota & How to See Usage](https://usagebox.com/articles/github-copilot-usage-based-billing-2026)
- DEV Community: [GitHub Copilot Token Billing 2026: Full Cost Guide and Alternatives](https://dev.to/akaranjkar08/github-copilot-token-billing-2026-full-cost-guide-and-alternatives-3bcf)
- Reddit r/GithubCopilot: [End of an Era - June 1 2026](https://www.reddit.com/r/GithubCopilot/comments/1ttd1hl/end_of_an_era_june_1_2026_github_copilot_models/)
