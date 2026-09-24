+++
date = '2026-09-24T07:31:40+08:00'
draft = false
title = "第九巡回法院 Copilot 判决：DMCA 第1202条不护开源，但许可证争议还活着"
description = "2026年9月16日，第九巡回法院就 Doe vs GitHub 作出裁决，判定 AI 代码生成工具未触发 DMCA 反规避条款。但这个判决的真实含义比'Copilot 赢了'要复杂得多。"
tags = ["AI观察", "科技"]
categories = ["AI观察"]
author = "Spiral"
+++

2026年9月16日，美国第九巡回上诉法院就 **Doe vs GitHub** 一案作出裁决。这是全球首个大模型代码生成工具与开源许可证冲突的上诉判决，结果一出，媒体普遍以"GitHub 胜诉"为标题。但实际读判决书会发现：这个结果既不是开源运动的终点，也不是 AI 版权问题的定论。

## 判决说了什么

本案原告是一群匿名的开源开发者，他们指控 GitHub Copilot 和 OpenAI Codex 在训练过程中使用了他们在 GitHub 上公开发布的开源代码，却从未在生成的代码中保留作者署名、版权声明和许可证条款——这些恰恰是每一份开源许可证明确要求保留的信息。

原告援引的是 **DMCA（数字千年版权法）第1202条**，该条款禁止在作品中"移除或改动"版权管理信息（CUI）。他们的逻辑是：Copilot 生成了代码，但没有保留原始许可证要求的信息，这等于"移除了"受保护的信息。

法院最终驳回了这一论点。判决书由 **Eric Miller 法官** 撰写，其中最关键的一句话是：

> "一个新作品的创作者未能包含 CMI，并不能说这个人'移除'或'改动'了什么。"

这个区分看起来很技术化，但它把"生成新作品时没有附带信息"和"从已有作品中盗取信息后再移除信息"划分成了两件完全不同的事。法院拒绝将"普通版权侵权"转化为"DMCA 侵权"，理由是这会让 Section 1202 变成任何版权纠纷的万能入口。

## 反共识观点：这个判决反而给开源留了空间

主流解读把这件事说成"Big AI 赢了，开源输了"。但仔细看判决措辞，结论恰恰相反：

**法院并没有说开源许可证被尊重了。** 它只是说 Section 1202 不是处理这个问题的正确工具。

判决原文紧接着就写明：这个判决**不涉及**原告提出的"开源许可证是否被违反"这一核心争议——那部分主张仍在地区法院审理中。

换句话说，DMCA 这条路被堵死了，但开源许可证的合同请求权（contractual claims）还活着。几乎所有主流开源许可证——从 MIT 到 GPL——都明确要求保留版权声明和署名。GitHub 和 OpenAI 是不是有义务在 Copilot 生成代码时提示用户注意这些要求，这个诉讼还没有被判决。

开源促进会（OSI）新任执行总监 **Duane O'Brien** 的回应很到位：

> "第九巡回法院只回答了 DMCA 一个条款的狭义问题。它没有判定开源许可证是否得到了尊重，而那部分主张仍在地区法院审理。几乎每一种开源许可证，从 MIT 到 GPL，都明确要求保留版权声明和署名。开发者是在这些条件下向世界提供他们的工作的，任何在此基础上构建工作成果的人——包括构建 AI 工具的公司——都应该尊重这些条件。"

法院用技术区分救了 Copilot，却没有给 Copilot 一张"开源许可证免责卡"。

## 三个关键事实

**1. 估值跳跃：OpenRouter 五个月估值涨五倍**

Stripe 收购 OpenRouter 的价格超过 **70亿美元**，而 OpenRouter 在2026年5月刚完成 B 轮融资，估值据报道为 **13亿美元**。五个月内估值翻超五倍，背后是 AI 路由层作为"token 经济基础设施"的战略价值被重新定价。

**2. 模型许可证分裂：中、美两套规则正在形成**

截至2026年9月，状态分化明显：
- **西方实验室**：Google Gemma 4（Apache 2.0）、Meta Muse Glimmer（Apache 2.0）、NVIDIA Nemotron 3 Ultra（OpenMDW 1.1）全面倒向宽泛开源许可证
- **中国前沿实验室**：智谱 GLM-5.3 新增自定义许可证，**$100亿营收以上**的模型即服务提供商必须通过 Z.AI 安全审查，且"关联公司"定义模糊；Kimi K3 和 MiniMax M3 均要求商业协议

这条裂缝在2026年正在加深，而不是弥合。

**3. 许可证审查被绕过：没人真的在检查**

dev.to 评论文章《Is Open Source AI Still Open in 2026?》指出一个冷酷现实：即使许可证明确要求署名，几乎没有任何模型托管服务在实际执行。GLM-5.3 的 Z.AI 安全审查触发门槛是"过去12个月合并营收超过100亿美元"——这个数字筛掉了全球几乎所有潜在用户，实际上更像是一张延迟执行的空头支票。

## 接下来看什么

**1. Doe vs GitHub 地区法院阶段**：许可证合同争议怎么判，将决定 Copilot 和类 Copilot 工具以后要不要内置许可证检测和代码来源提示。这是真正有牙齿的部分。

**2. GLM-5.3 许可证的"关联公司"定义**：智谱在英文文本中留了空白，中文文本则引用了中国法律定义。国际托管服务商——尤其是想服务全球开发者的中国境外云厂商——会不会因此拒绝提供 GLM-5.3 的商业推理服务？这个漏洞要么被律师填补，要么引发新一轮许可证战争。

**3. Stripe 收购 OpenRouter 后的 neutrality 问题**：OpenRouter 一直以"provider-neutral"（供应商中立）为卖点，被 Stripe 收购后还能不能维持这个定位？Stripe 自己的支付业务和其他 AI 提供商存在竞争关系，企业用户把 OpenRouter 当作避免 vendor lock-in 的工具，这个信任基础会不会被动摇？

**4. 中国开源模型许可证会不会引发出口管制嵌套**：如果 GLM-5.3 的安全审查条款被美国实体触发，这算不算一种事实上的出口管制？这条线的法律定性目前是空白。

---

## 参考资料

- [Latest open artifacts (#24): Motif-3, GLM-5.3, Hy4-preview and open model licenses — Interconnects (Nathan Lambert)](https://theai.news/briefs/2026/09/latest-open-artifacts-24-motif-3-glm-5-3-hy4-preview-and-ope-bb2a664c)
- [US District Court Decision in AI's Favor Worries Open-Source Developers — DevOps.com](https://softmag.in/devops/us-district-court-decision-in-ais-favor-worries-open-source-developers-26)
- [State of Open Source AI 2026 — StateOfOpenSource.ai](https://stateofopensource.ai/state-of-open-source-ai-v1-1.pdf)
- [Is Open Source AI Still Open in 2026? — dev.to](https://dev.to/mithilesh_gaurihar/is-open-source-ai-still-open-in-2026-pfi)
- [Stripe agrees to acquire OpenRouter — Stripe Newsroom](https://stripe.com/en-at/newsroom/news/stripe-agrees-to-acquire-openrouter)
- [Stripe seals $7bn deal to buy AI startup OpenRouter — Yahoo Finance](https://uk.finance.yahoo.com/news/stripe-seals-7bn-deal-buy-071500047.html)
- [Nvidia targets Hugging Face in $13bn deal as open-weight AI draws major capital — Europe's Run](https://europes.run/article/91401/nvidia-targets-hugging-face-in-13bn-deal-as-open-weight-ai-draws-major-capital)
