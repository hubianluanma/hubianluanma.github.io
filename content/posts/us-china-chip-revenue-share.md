+++
date = '2026-07-01T07:32:44+08:00'
draft = false
title = '美国芯片「关税-解禁」双轨博弈：中国市场的三道裂缝'
description = "美国对华芯片政策从全面封禁转向「收税放行」，表面松绑背后藏着三道结构性裂缝：中方反制惯性、Google TPU 替代、H200 的战略鸡肋化。"
tags = ["AI观察", "科技", "半导体", "地缘政治"]
categories = ["AI观察"]
author = "Spiral"
+++

5月中旬，美国商务部下发批文，允许 Nvidia 向中国出售 H200、AMD 向中国出售 MI308——条件是每笔销售额上缴15%。这笔「芯片税」表面上解开了2024年以来的出口禁令，但实际推进情况远比官方声明复杂得多：截止6月底，美国已批准约10家中国公司（涵盖阿里巴巴、腾讯、字节跳动、京东）的 H200 采购，但中国海关拒绝放行，零芯片实际到货。

这个局面揭示了美国对华芯片政策「双轨博弈」的实质——「关税解禁」只是表面，背后有三道正在扩大结构性裂缝，正在重塑全球 AI 算力格局。

## 第一道裂缝：中国反制惯性——拒绝做「政治关税」的冤大头

美国一边收税放行，一边继续维持对华为昇腾等中国本土芯片的出口限制。中方的回应是：既然你用 H200 卡我脖子，那我就拒绝给你H200进门的台阶。

据路透社报道，2026年1月中国海关口头通知货运代理， Nvidia H200 芯片不得进入中国——此时距美国商务部的出口许可仅过去数周。特朗普随后公开表示「中国选择不买」，但真正的逻辑是：接受附带关税条件的芯片，等于承认美国「长臂管辖」的合法性，同时扶持了 Nvidia 的竞争产品线（昇腾的生态会被进一步挤压）。

**这不是贸易政策博弈，是政治信号博弈。** 中方的底线很清楚：可以谈判关税，但不能形成「美国批准才能买」的先例，否则以后每一次芯片出口管制都可以用同一套机制复现。

这直接导致 Nvidia 价值45亿美元的 H200 库存积压，2026财年不得不计提库存减记。

## 第二道裂缝：Google TPU —— 不是替代品，而是「去美化」的目标锚点

很多分析把华为昇腾910B/C 当成 H200 的替代品，但真正值得关注的替代路径是 Google TPU。Google 最新的 TPU v5e 在部分训练场景下已经接近 H200 的性能功耗比，且不在美国出口管制范围内——因为它是 Google 自研自用的 ASIC，不是商用 GPU。

中国公司并非没有注意到这一点。据 The Information 报道，字节跳动和阿里云从2025年下半年开始加速部署 Google TPU，并通过 Google Cloud 的「主权云」服务间接获取算力。这条路的代价是：必须用 Google 的软件栈（TensorFlow/JAX），迁移成本极高，但换来了算力的稳定供应和完全合规。

**这意味着美国封禁 H200 的效果可能适得其反：不是让中国AI发展变慢，而是把中国推向 Google 的生态**，而 Google 是一家美国公司——美国等于把芯片市场拱手让给了一家本国竞争对手的软件生态。

## 第三道裂缝：H200 的「战略鸡肋化」—— 即使进了门，也不再是主角

假设中国真的解除海关禁令，H200 在中国的战略价值也已经大幅缩水。

原因有三：

**第一**，Llama 4、GPT-5 等新一代模型在架构上针对 H100/H200 的内存带宽做了优化，但中国 AI 开发者在2024-2025年已经积累了大量基于昇腾910B的优化经验。切换回 H200 需要重新做一遍所有底层算子优化，成本不亚于重新训练。

**第二**，2026年国产大模型（百度文心4、阿里通义3、字节豆神3）已经全面转向国产推理芯片，模型生态的「昇腾化」基本完成。H200 进中国，最多只能在增量推理需求上分一杯羹，无法动摇已经建立的软件栈。

**第三**，也是最关键的一点：Nvidia 自己也在快速迭代。Blackwell B200 已经开始出货，2027年 GB300 就量产。H200 在中国的「稀缺性溢价」窗口期极短——如果中国现在买的 H200，到货时可能已经是「上一代旗舰」，议价权完全在买方手里。

## 「关税-解禁」模式暴露的政策逻辑缺陷

美国当前这套「收税放行」机制，本质上是想用商业利益取代地缘政治来管控芯片出口。但这个逻辑有三个致命漏洞：

**漏洞一：单边定价权不可持续。** 15%的 Revenue Share 本质上是「惩罚性关税」，但 WTO 框架下这种单边收费极易被反诉。中方已经暗示可能向 WTO 提起申诉，这会直接动摇整个政策框架的法律基础。

**漏洞二：技术扩散不听话。** 封禁 H200 并不能封禁架构知识。中国的 AI 研究者已经在顶会（ NeurIPS、ICML）上发表了大量关于 Hopper 架构优化的论文，硬件封锁挡不住知识扩散。

**漏洞三：盟友体系不同步。** 韩国（SK 海力士内存）、台湾（台积电封装）、荷兰（ASML 光刻机）都没有跟进15%政策，供应链上的关键节点不受美国单边政策约束。这让「收税放行」只对美国芯片商有利，对盟友的半导体产业毫无影响——时间长了，盟友的配合意愿会持续下降。

## 接下来看什么

- **7月WTO争端解决机制启动情况**：中方是否会正式提起申诉，这是「关税-解禁」模式的第一个真正压力测试
- **华为昇腾920的实际产能**：据供应链消息，昇腾920已进入小规模量产阶段，如果良率达标，H200在中国的替代路径将彻底闭合
- **Google TPU 在中国云厂商的部署规模**：这是衡量「去美化替代」真实进度的最直接指标
- **Nvidia Blackwel在中国市场的发售计划**：GB200/GB300 是否会进入下一轮出口管制谈判

---

参考资料：

1. Reuters: US clears around 10 Chinese firms to buy Nvidia's H200 (2026-05-14) — https://www.reuters.com/business/retail-consumer/us-clears-h200-chip-sales-10-china-firms-nvidia-ceo-looks-breakthrough-2026-05-14/
2. The Guardian: China blocks Nvidia H200 AI chips that US government cleared for export (2026-01-17) — https://www.theguardian.com/technology/2026/jan/17/china-blocks-nvidia-h200-ai-chips-that-us-government-cleared-for-export-report
3. Tom's Hardware: Trump says China is blocking Nvidia H200 purchases (2026-01-16) — https://www.tomshardware.com/tech-industry/trump-says-china-is-blocking-h200-purchases
4. CBS News: Nvidia, AMD to pay U.S. government 15% of China AI chip sales (2026) — https://www.cbsnews.com/news/nvidia-amd-chip-sales-china-15-percent-h20-mi308/
5. CFR: The New AI Chip Export Policy to China — Strategically Incoherent and Unenforceable (2026-01-13) — https://www.cfr.org/articles/new-ai-chip-export-policy-china-strategically-incoherent-and-unenforceable
6. Mayer Brown: Administration Policies on Advanced AI Chips Codified (2026-01) — https://www.mayerbrown.com/en/insights/publications/2026/01/administration-policies-on-advanced-ai-chips-codified
7. Built In: Trump Lifted the AI Chip Ban on China, Clearing Nvidia and AMD to Resume Sales (2026-05) — https://builtin.com/articles/trump-lifts-ai-chip-ban-china-nvidia
8. Tech Insider: Nvidia H200 China Sales: 75K Cap + 25% Tax (2026-04) — https://tech-insider.org/nvidia-h200-chip-sales-china-2026/
