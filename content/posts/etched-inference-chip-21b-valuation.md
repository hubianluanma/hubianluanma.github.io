+++
date = '2026-08-23T07:31:44+08:00'
draft = false
title = 'Etched 估值一月翻倍至 210 亿美元：ASIC 复兴还是又一场硬件豪赌？'
description = "ASIC 专用芯片在推理时代卷土重来。Etched 26 天估值翻倍至 210 亿美元，背后是 Jane Street 实测买单，还是 VC 们集体陷入了「模型即芯片」的叙事狂欢？"
tags = ["AI观察", "芯片", "创业"]
categories = ["AI观察"]
author = "Spiral"
+++

2026 年 8 月 18 日，AI 芯片初创公司 Etched 宣布完成 7 亿美元 D 轮融资，估值从 6 月的 103 亿美元飙升至 210 亿美元——**前后仅隔 26 天**。

这不是什么「种子轮到 A 轮」的常规跨越，而是一家成立不到四年、连自研芯片都还没大规模量产的初创公司，在一级市场的定价已经超过 AMD、赛灵思被收购前的市值。

量化交易巨头 Jane Street 领投， Sequoia、a16z、Kleiner Perkins、Bain Capital Ventures、Tiger Global、Blackstone 等一线机构跟投。Etched 的投资方名单几乎是一部完整的顶级 VC 百科全书。

与此同时，Databricks 同周宣布完成 50 亿美元融资，估值达到 1900 亿美元。投资方疯抢份额，原定目标 10 亿美元，最终超募 15 倍。

两件事放在一起来看，有一条暗线清晰浮现：**市场对 AI 基础设施的渴求，已经脱离了任何基本面约束。**

## Etched 到底在做什么

Etched 的核心产品是一款名为 **Sohu** 的 ASIC（专用集成电路）芯片，专门为 Transformer 推理任务定制。

与传统 GPU 相比，ASIC 的逻辑是：GPU 之所以贵，是因为它什么都能干——图形渲染、科学计算、训练、推理，全都塞进同一块芯片。如果只保留 Transformer 真正需要的运算单元（矩阵乘法 + Attention），把其他电路全部砍掉，理论上可以把效率提升一到两个数量级。

Sohu 芯片的核心参数来自 Etched 的官方披露：

- TSMC N4P 制程，与英伟达 H100 同代
- Etched 声称 Sohu 在 Transformer 推理任务上实现**每瓦特性能比 H100 高 10 到 100 倍**
- 采用「低电压推理」（Low Voltage Inference）架构，目标电压低于传统 AI 芯片的 50%，同时保持 80% 以上的峰值浮点算力
- 推出 Cluster Scale Memory 技术，支持多芯片共享高速内存池，降低跨节点通信延迟

但这里有一个重大问题需要直说：**这些数字全部来自 Etched 自己的工程博客，没有任何第三方机构做出过独立验证。**

Jane Street 是目前唯一公开named的实测客户。这家量化交易公司有自己的芯片测试能力，他们说「结果令人满意」——但没有公布任何 Benchmark 数据。

这很重要，因为芯片行业有大量「实验室数据」与「量产表现」严重不符的前车之鉴。

## 26 天翻倍的估值凭什么

让我们把 Etched 的融资时间线放在一起看：

| 时间 | 事件 | 估值 |
|------|------|------|
| 2024 年 12 月 | 融资 5 亿美元 | 50 亿美元 |
| 2026 年 6 月 | 走出隐匿模式 | 80 亿美元（累计融资） |
| 2026 年 7 月 23 日 | C 轮 3 亿美元 | 103 亿美元 |
| 2026 年 8 月 18 日 | D 轮 7 亿美元 | 210 亿美元 |

从 103 亿到 210 亿，Etched 在这轮融资里给出的核心论点是：**他们拿到了超过 10 亿美元的客户合同**，涵盖上市 AI 公司和云服务商。

这个数字本身并不夸张——对于一家已经融了 19 亿美元、总估值 210 亿的公司来说，10 亿合同对应的市销率（P/S）大概是 21 倍。与之对比，英伟达目前的市销率大约是 35 倍。

问题在于：合同 ≠ 收入，更不等于盈利。芯片公司从「拿到合同」到「大规模出货」之间，隔着供应链管理、量产良率、软硬件调优、好几个迭代周期。

Etched 的创始团队是三个哈佛辍学生（平均年龄多大没有公开），公司目前员工约 400 人，其中约 15% 来自英伟达。这个规模放在芯片行业，属于「严重小型团队」——赛灵思被 AMD 收购时员工超过 4000 人。

Webpronews 的评论一针见血：**「不要投资做芯片的年轻人。」** 这是整个行业对半导体初创公司的默认判断，Sequoia 合伙人 Sonya Huang 后来亲自打破了这个判断，但这不代表风险消失了。

## ASIC 复兴：这次真的不一样吗

ASIC 不是一个新概念。加密货币矿机（比特大陆）、网络设备（思科 ASIC）、智能手机 SoC（苹果、华为）都是 ASIC 的成功案例。**但 AI 推理 ASIC 面临一个独特的悖论：模型架构正在快速迭代。**

Transformer 从 2017 年诞生到现在，已经经历了 Bert、GPT-2、GPT-3、GPT-4、混合专家模型（MoE）、Mamba、RWKV、RetNet……每一次架构变化，都意味着针对上一代模型优化的芯片有可能被降维打击。

Etched 目前的官方表态是「我们的芯片可以运行任何前沿模型，不限于某一个」。但这句话本身就在暗示：Sohu 并不是真正针对某个单一模型深度定制的——它的「专用性」更多体现在对 Transformer 注意力机制的硬件加速上。

问题在于：**如果模型架构继续分化，如果非 Transformer 的新架构成为主流，ASIC 的专用优势会快速消失。** 历史上看，芯片公司最难对抗的不是竞争者，而是上游模型架构的范式转移。

对比之下，GPU 的通用性反而成了某种「免责条款」——英伟达不需要担心客户用的是什么模型架构，因为 CUDA 生态可以适配任何计算图。这是 Etched 在产品层面需要持续回答的核心问题。

## Databricks 这一幕：资本想进去、企业不想出

与 Etched 的故事形成鲜明对比的是 Databricks 的 1900 亿美元估值——**这更像是一个「理性的疯狂」。**

Databricks 联合创始人兼 CEO Ali Ghodsi 透露，这次融资他们原本只想融 10 亿美元。结果投资人的认购意向加起来是 150 亿美元。Ghodsi 本人的原话是：「兴趣程度简直疯狂。」

Databricks 的底气来自实打实的财务数据：

- 年化收入超过 70 亿美元
- 同比增长超过 80%
- Lakebase（面向 AI Agent 的无服务器 Postgres 数据库）单个产品已突破 1 亿美元年化收入
- 过去 20 个月累计融资 200 亿美元

这说明什么？**头部 AI 基础设施公司已经完全不需要靠 IPO 来证明自己了。** 它们在私人市场上能拿到的钱比二级市场更多，条件更灵活，信息披露义务更少。私募市场成了它们事实上的长期融资渠道。

这不是某个公司的问题，而是整个 IPO 市场的结构性变化。Spotify、Stripe、OpenAI……越来越多的千亿级公司正在推迟或无限期搁置上市计划，而 VC 们则被迫接受流动性更差的筹码。

## 接下来看什么

1. **Etched 能否真正量产交付**：2026 年底前如果 Sohu 芯片没有实际出货数据，210 亿美元的估值将面临严峻拷问。Jane Street 的那套 Rack 是目前唯一的「已完成测试」证据。

2. **软硬件生态是否建立**：芯片卖出去只是第一步，开发者能否低成本迁移、主流框架是否原生支持，这才是决定 ASIC 公司能否留住客户的关键。英伟达的护城河从来不只是 GPU 本身，而是 CUDA 生态。

3. **VC 市场的「供给侧改革」**：Databricks 的案例说明，大型机构投资者手里囤积了大量 Capital，它们需要出口。当 AI 成为唯一被持续看多的资产类别，这些资金会不断推高头部公司的估值——直到某一次财报或市场事件戳破这个循环。

4. **Transformer 架构的持久性**：如果 2027-2028 年出现新的主流模型架构，ASIC 公司能否及时适配将是生死线。Etched 正在押注 Transformer 的统治地位至少再维持三到五年。

---

**一句话结论**：Etched 的 210 亿美元估值是一张昂贵的赌桌——赌的是推理优先的计算时代真的到来，赌的是 Transformer 不会被快速替代，赌的是三个哈佛辍学生能管好一个价值千亿人民币的供应链。这个赌局有合理的逻辑支撑，但在拿到第三方 Benchmark 数据和真正的量产收入之前，它本质上还是一场顶级 VC 们的集体冒险。

Databricks 的故事则相反：它贵得有道理，但它太贵了已经不再适合普通投资者参与。一级市场正在变成少数机构玩家的游戏，而这场游戏的赌注，正在以惊人的速度膨胀。

---

## 参考资料

- TechCrunch: [Etched's valuation doubles to $21B in a month](https://techcrunch.com/2026/08/18/etcheds-valuation-doubles-to-21b-in-a-month/)
- TechMonitor: [Etched raises $700m at $21bn valuation, ships first rack to Jane Street](https://techmonitor.ai/news/etched-raises-700m-at-21bn-valuation-ships-first-rack-to-jane-street)
- Webpronews: [Etched's $21 Billion Ascent: Young Founders Bet Everything on Transformer Silicon](https://webpronews.com/etcheds-21-billion-ascent-young-founders-bet-everything-on-transformer-silicon)
- Memeburn: [Etched Doubled Its Valuation to $21B in 26 Days. Here's Who Paid](https://memeburn.com/etched-doubled-its-valuation-to-21b-in-26-days-heres-who-paid)
- TechCrunch: [Databricks wanted to raise $1B, investors wanted $15B. It settled on $5B at a $190B valuation](https://techcrunch.com/2026/08/13/databricks-wanted-to-raise-1b-investors-wanted-15b-it-settled-on-5b-at-a-190b-valuation/)
- Databricks 官方新闻稿: [Databricks Grows >80% YoY, Surpasses $7B Revenue Run-Rate](https://www.databricks.com/company/newsroom/press-releases/databricks-grows-80-yoy-surpasses-7b-revenue-run-rate-scales)
