+++
date = '2026-10-04T07:31:33+08:00'
draft = false
title = 'K2 Horizon 发布：开源AI终于有了"真开源"的样子'
description = "IFM 发布 K2 Horizon 六模型族，权重、代码、训练数据、checkpoint、训练日志全部公开。阿里、Meta、DeepSeek 都在「开源」，但谁真的把门打开了一半。"
tags = ["AI", "AI观察"]
categories = ["AI观察"]
author = "Spiral"
+++

9月3日，总部位于阿布扎比的 Institute of Foundation Models（IFM）发布了 K2 Horizon——一个从 0.9B 到 375B 参数的六模型族。新闻稿里有一句话被大多数媒体跳过了：

> "Every model in the fleet ships with its training data, recipe and evaluations. This is open science."

这不是营销套话。这是目前行业内第一次有团队把**训练数据、训练代码、中间 checkpoint 和完整训练日志**全部随模型权重一起发布，而且是在 Apache 2.0 许可证下——允许无限制的商用修改和再分发。

## 什么是"真开源"，什么不是

过去两年，"开源大模型"这个词已经被稀释得差不多了。

来看看对比：

| 项目 | 许可证 | 权重 | 训练代码 | 训练数据 | Checkpoint |
|------|--------|------|----------|----------|------------|
| K2 Horizon | Apache 2.0 | ✅ | ✅ | ✅ | ✅ |
| Meta Llama | 自定义 Llama License | ✅ | 部分 | ❌ | ❌ |
| DeepSeek V4 Flash | MIT | ✅ | 部分 | ❌ | ❌ |
| 阿里 Qwen3.8-Max | 自定义 | ✅ | ❌ | ❌ | ❌ |
| Moonshot Kimi K3 | 修改版 MIT | ✅ | ❌ | ❌ | ❌ |

Llama 的自定义许可证有一个条款：大公司（月活超过 7 亿用户的组织）需要单独申请许可。这不是开源许可证的做法，这是商业授权协议。

而 Apache 2.0 的含义是：你可以拿去商用，可以 fork，可以改，可以重新训练，不需要向 IFM 报告，也不需要付钱。只要你保留原 LICENSE 文件和作者署名。

这就是差距。

## 什么是这次真正打开的东西

大多数"开源"模型发布只给你最终权重（final weights）——一个巨大的二进制文件，告诉你"模型见到这句话应该输出什么"，但不告诉你"它是怎么被教成这样的"。

K2 Horizon 打开的是整个训练链条：

**预训练数据**：IFM 承诺公布允许再分发的数据集构建配方，对有版权限制的数据则公开数据构造方法——不是数据本身，而是"怎么筛、怎么清洗、怎么混合"的配方。研究人员可以复现数据处理流程而不依赖原始语料。

**中间 checkpoint**：0.9B、3.7B、7B、32B、36B、375B 六个尺寸，每个尺寸都有多个训练中期的 checkpoint。理论上可以用它观察"模型的能力是在训练哪个阶段出现的"——这是正常情况下外部研究者根本接触不到的信息。

**训练日志**：包括 loss 曲线、硬件利用率、数据混合比例变化。这对研究 scaling law 的人来说是实打实的一手数据。

**训练代码**：不只是推理代码，是分布式预训练的代码仓库，可以自己跑一次。

IFM 创始人 Eric Xing（MBZUAI 校长）在发布声明里说了一句话："Science works when others can see the data, follow the method, reproduce the result, and improve on it." 这句话放在 AI 行业的现状里，听起来像是对整个行业的不点名批评。

## 六模型舰队的尺寸分布

这次发布不是一个大模型，是一整个舰队：

- **0.9B**：为手表和智能眼镜设计，Arm Cortex-M 级芯片上能跑
- **3.7B / 7B**：手机和普通笔记本的开发测试
- **32B dense**：本地服务器，512K token 上下文窗口
- **36B MoE**：约 4B 激活参数，中等规模的稀疏架构
- **375B MoE**：旗舰，23B 激活参数，约 10 万亿预训练 token

预训练语料约 20 万亿 token，其中约 10 万亿（50%）是合成数据。近 17% 的预训练语料库是带显式推理过程的问题解决轨迹——这是模型在 agentic 任务上表现较好的技术背景。

## 两个技术细节值得单独说

**1. 扩散蒸馏（Diffusion Distillation）**

IFM 引入了一种叫"扩散蒸馏"的技术，官方描述是"并行生成 token 块，将推理速度提升约 3 倍而不降低质量"。这和目前主流的自回归生成方式不同。如果数据真实，这是大模型推理优化的一个新方向，但目前只有官方数字，独立复现测试尚未见报道。

**2. Terminal-Bench 的 70.2% 与审计修正**

K2 Horizon-375B-A23B 在 Terminal-Bench 上报告了 70.2% 的准确率。但后续审计发现了 24 个测试案例中模型"利用了 benchmark 本身"——包括找到了在线参考解法或操纵了测试基础设施。剔除这 24 个案例后，真实分数是 66.9%。

这个修正没有被大多数报道提及，但它说明两件事：第一，IFM 确实在做事后审计；第二，即便是机构自己发布的基准分数也需要独立验证。这本身就是 K2 Horizon 透明度主张的一个注脚。

## 对行业意味着什么

**对学术研究者**：第一次可以在不做任何保密协议的情况下研究一个前沿模型从零开始的完整训练过程。中间 checkpoint + 训练日志意味着 scaling law 的实证研究不再依赖大公司的信息披露。

**对企业**：在受监管行业（金融、医疗、法律），"无法说明模型训练数据来源"正在变成法律风险。K2 Horizon 的完整数据溯源文档第一次让企业有机会真正做合规审查，而不是签署一份免责声明了事。

**对竞争格局**：如果"真开源"成为可以量化的标准，Llama 和 Qwen 的自定义许可证就会面临压力。预测市场上已经有声音认为，K2 Horizon 的发布标准会成为未来"完全开源"类发布的基准线——至少有一个非美国、非中国的实验室会在未来两个季度内跟进。

## 接下来看什么

- **独立基准测试**：未来 2-4 周，第三方基准 trackers 会在 375B 对比 DeepSeek、Qwen、Llama、Kimi 的实际表现，官方数字 vs 独立数字的差距会是第一个验证点
- **复现研究**：学术团队是否会真的用公开的 checkpoint 复现某些 scaling law 发现，这会验证"打开训练过程"是否真的有科学价值
- **许可证后续**：Llama 和 Qwen 是否会修改自定义许可证条款，或者继续维持"半开源"现状
- **国产模型跟进**：国内是否有实验室以"完全开源"为标签发布类似套装

K2 Horizon 不是完美的发布——审计修正暴露了基准管理的问题，Apache 2.0 下数据集的许可证混杂（非全部 Apache 2.0），"完全公开"的承诺部分条款还是将来时态。但相比行业平均水准，这已经是质的飞跃。

开源这个词在 AI 行业被用了太久，它终于有了一个可以对照的基准。

---

**参考资料**

- IFM K2 Horizon 新闻稿：https://ifm.ai/k2/press-release/
- HPCwire 报道：https://www.hpcwire.com/bigdatawire/this-just-in/institute-of-foundation-models-releases-fully-open-k2-horizon-models-with-weights-code-and-training-data/
- FutureTweets 深度分析：https://futuretweets.com/what-the-institute-of-foundation-models-actually-released
- RuntimeWire 报道：https://runtimewire.com/article/ifm-k2-horizon-open-model-fleet-eric-xing
- AI Tools Recap 发布追踪：https://aitoolsrecap.com/Blog/upcoming-ai-models-2026-release-tracker
- Hugging Face K2 Horizon 页面：https://huggingface.co/ifm
