+++
date = '2026-08-27T07:31:31+08:00'
draft = false
title = '中国开源模型正在改写 AI 推理市场的游戏规则'
description = 'OpenRouter 流量数据显示中国开源模型已占据全球 AI 推理市场 45% 以上的份额，DeepSeek V4 Flash 周调用量突破 8.83 万亿 token。这不是技术竞争叙事，而是一个更底层的现象：推理市场正在商品化，而美国公司发明的 API 商业模式正在被价格战瓦解。'
tags = ['AI', 'AI观察']
categories = ['AI观察']
author = 'Spiral'
+++

最近看 OpenRouter 的公开数据，发现一个反直觉的事实：2026 年 8 月，按实际 token 调用量计算，全球最大的 AI 推理消费市场里，中国模型已经是主角。DeepSeek V4 Flash 周调用量 8.83 万亿 token，环比增长 570%；Tencent Hy3 免费版每周 6.13 万亿 token；前五名里四个是中国模型。这不是排行榜上的短暂波动，是真实的生产流量。

更值得注意的不是数字本身，而是背后的机制：美国公司建立了 AI API 经济，但真正在这个经济里大规模消耗 token 的，是愿意把价格压到十分之一的后来者。这件事的逻辑和大多数媒体报道的方向不太一样。

## 推理市场商品化：一条不可逆的价格曲线

2023 年初，GPT-4 级别的推理定价约为每百万 token 20 美元。今天，DeepSeek V4 Flash 的定价是每百万 token 输入 0.077 美元、输出 0.154 美元。这个跌幅不是摩尔定律能解释的——三年跌 50 倍，远超硬件改进速度。驱动因素是竞争，而不是效率。

开源模型在这个过程中扮演的角色是价格锚。DeepSeek V4 Flash 出现后，OpenRouter 上所有同类模型的输出价格都在向它靠拢。一家有量化仓位管理需求的团队告诉我，他们把生产级的客服 Agent 全部从 Claude 切到了 DeepSeek V4 Flash，原因很简单：输出 token 消耗量大，便宜 80% 的情况下质量够用，就能省出真实的钱。

这不是孤例。Lindy CEO Flo Crivello 对外表示，公司把 100% 的流量从 Claude 切换到 DeepSeek，预计每年节省数百万美元。这类决策在初创公司和中小型科技企业里扩散的速度比大厂快得多——因为它们的生死线就在毛利率上，没有预算为"最领先的模型"支付溢价。

## 反共识：不是"中国 AI 超越美国"，而是"美国 API 商业模式被自己的逻辑打败"

主流叙事是中美 AI 竞争、中国正在超越美国。但看 OpenRouter 的数据，真实发生的事情更微妙：不是中国模型的绝对能力在各个维度超越了 Claude Opus，而是**美国公司发明的按 token 计费 API 模式，正在被这种计费方式的受益者用自己的逻辑打败**。

OpenAI 和 Anthropic 建立的商业模式里，token 是成本单位，收入 = token 单价 × 消耗量。当客户开始精打细算每个 token 的性价比，降价是唯一出路。Anthropic 激进下调 API 价格、OpenAI 推出 GPT-5.6 三变体（Sol/Terra/Luna 差异化定价）——这些动作都是在应对一个事实：在开源模型的竞争压力下，闭源模型如果不能在质量上拉开足够大的差距，就必须在价格上让步。

这不是竞争失败，是竞争规则变了。以前的护城河是"我有你没有的能力"，现在的护城河是"我能以这个价格卖给你"。但护城河的定义权已经不在闭源厂商手里了——因为开源模型的定价逻辑是成本加成，而不是价值定价。DeepSeek 可以把价格压到几乎成本线，因为它的商业模式不是靠 API 收费变现的。

## 真实受益者：不是模型，而是工作流

有趣的是，这波中国开源模型崛起的最大受益者，可能不是模型本身，而是围绕它的工作流工具链。

AutoGen、LangGraph、LlamaIndex、DSPy 这些 Agent 编排框架，用户粘性来自于工作流设计而不是底层模型——换掉 DeepSeek V4 Flash 换成 GLM-5.2，工作流不需要重写，prompt 改两行就能跑。这意味着在应用层沉淀了差异化能力的团队，其实并不担心模型层的波动。他们关心的是：哪个模型今天性价比最高，然后路由过去。

OpenRouter 的 Auto Router 功能（基于每周 55 万亿 token 实际消费数据自动选择最优模型）本质上就是在服务这个需求。8 月 10 日该功能全量上线后，对应的正是开发者对多模型路由的强烈需求——不是一个模型通吃，而是按任务动态分配。

## 监管噪音：美国国会调查影响的是情绪，不是流量

美国国会正在调查中国 AI 模型的安全风险，有议员提出要立法限制联邦机构使用中国模型。这个叙事在媒体上很热闹，但看 OpenRouter 的数据，这种噪音几乎不影响实际使用量——因为用这些模型的是私营公司，它们看的不是政策风向，是账单数字。

真正有潜在影响的是另一个动向：北京正在考虑限制中国最先进模型向海外提供服务。如果这个政策落地，会直接冲击 OpenRouter 上的中国模型可用性。但截至目前，DeepSeek V4 Flash、GLM-5.2、Kimi K3 的 API 在全球范围内依然可用，且没有任何一家减少了在海外的投入。

## 接下来看什么

**Watch 1：DeepSeek V4 Pro 正式版发布节奏。** V4 Flash 已经验证了市场需求，Pro 版如果定价在合理区间，会进一步压缩闭源模型的利润空间。

**Watch 2：Anthropic 的下一步定价策略。** Claude Opus 4.8 仍然是 OpenRouter 上排名第一的闭源模型，但它和 DeepSeek V4 Pro 的价格差距如果继续扩大，切换压力会从初创公司蔓延到中型企业。

**Watch 3：北京限制海外访问政策的落地情况。** 这是目前最大的政策尾部风险。如果最主流的几个中国模型在海外的可用性下降，OpenRouter 的流量格局会重新洗牌。

**Watch 4：Llama 4 在欧盟的合规问题。** 欧盟开发者对 Llama 4 的多模态权限问题仍有顾虑，这给中国开源模型在欧洲的渗透留出了额外的市场空间。

---

一句话总结：AI 推理市场正在商品化，价格战是表象，根因是开源模型把"模型能力"变成了可替换的生产资料而不是稀缺资产。在这场商品化浪潮里，受益最多的是能把模型路由和工作流设计做好的应用层开发者，而不是任何一个模型供应商。

参考资料：

- [AI大模型最新进展深度盘点（2026年8月篇）](https://iaipie.com/ai%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%80%E6%96%B0%E8%BF%9B%E5%B1%95%E6%B7%B1%E5%BA%A6%E7%9B%98%E7%82%B9%EF%BC%882026%E5%B9%B48%E6%9C%88%E7%AF%87%EF%BC%89%EF%BC%9A%E4%B8%AD%E5%9B%BD%E5%BC%80%E6%BA%90/)
- [OpenRouter Usage Statistics — Baidu Baike](https://baike.baidu.com/en/item/OpenRouter/1528457)
- [Chinese AI Models Top OpenRouter — Tech Insider](https://tech-insider.org/au/chinese-ai-models-openrouter-2026)
- [开源 AI 现状 2026：中国模型占 OpenRouter 61%](https://abmedia.io/mozilla-state-of-open-source-ai-2026-report-china-openrouter-61-percent)
- [Chinese Models Cross 30% of U.S. Enterprise Token Traffic — The Agentic Review](https://theagenticreview.com/articles/2026-07-08-chinese-ai-models-claim-30-46-of-u-s-enterprise-token-traffic-as-congress-opens)
- [OpenRouter Now Picks Your Model by What Everyone Else Is Paying For — Ground Truth](https://groundtruth.day/news/openrouter-now-picks-your-model-by-what-everyone-else-is-paying-for.html)
- [Qwen 3.8 vs Kimi K3: Cheaper Isn't Simpler — MoClaw](https://moclaw.ai/blog/qwen-3-8-vs-kimi-k3)
