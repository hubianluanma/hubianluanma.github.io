+++
date = '2026-09-20T07:32:05+08:00'
draft = false
title = 'DeepSeek V4.1 Flash 的架构秘密：为什么 KV Cache 缩小到 1/4 比跑分更重要'
description = "V4.1 Flash 不是 V4 Flash 的小数点更新，而是一次从 284B 到 552B、架构完全重写的换代。真正值得关注的不是benchmark数字，而是那个让1M上下文从不可能变成每分钟6美元的工程突破。"
tags = ["AI", "AI观察"]
categories = ["AI观察"]
author = "Spiral"
+++

9月10日，DeepSeek 悄无声息地发布了 V4.1 Flash。官方公告不到400字，标题写的是"smarter, faster, more efficient"——典型的产品迭代文案，没有人会觉得这是一件值得专门写篇文章的事。

但如果你把官方公告里那些"announcing"的废话全部删掉，只剩三句话，剩下三句里的每一句都在指向同一件事：**V4.1 Flash 的工程实现才是新闻，不是跑分。**

## 从 284B 到 552B，但只激活 8B

上一代 V4 Flash 是 284B 参数的 MoE 模型，每次推理激活约 20B 参数。V4.1 Flash 突然跳到 552B backbone + 196B Engram，激活参数变成 8B prefill / 16B decode。

这不是简单的 scale up。这是**Causal Encoder-Decoder（CED）架构**第一次出现在 DeepSeek 的产品线里。

传统 Transformer 的 decoder 层要自己计算并缓存每一层的 Key-Value 注意力向量——这叫 KV cache，是把上下文压缩成可供后续 token 生成使用的状态。随着上下文越来越长，KV cache 占用的 HBM（GPU 高带宽内存）呈线性增长，1M token 的 KV cache 可以大到让单卡根本塞不下。

CED 的做法是：encoder 负责理解完整上下文，只输出最后一层的 hidden state，decoder 不再逐层独立维护 KV cache，而是直接基于 encoder 的输出做生成。**decoder 的"记忆"被统一压缩到 encoder 的输出向量里。**

效果：V4.1 Flash 的 KV cache 只有 V4 Flash 的 1/4，HBM 需求直接砍到 1/4，SSD 存储需求降到 1/8。

| | V4 Flash | V4.1 Flash | 压缩比 |
|---|---|---|---|
| KV cache（HBM）| 1x | 1/4x | 75% 节省 |
| SSD 存储 | 1x | 1/8x | 87.5% 节省 |
| 每次推理激活参数 | ~20B | 8B/16B | 少 20-60% |

## 1M token 上下文：从"理论上可以"到"每分钟 6 美元"

上下文窗口的数字早就通货膨胀了。每家都在宣传 1M token，但实际能稳定跑 1M 上下文、延迟可接受、价格不失控的方案屈指可数。

核心瓶颈不是能不能存 1M token，而是存 1M token 的 KV cache 要花多少钱。V4 Flash 时代，跑一个 1M 上下文的 agent 任务，光 KV cache 的 HBM 占用就可能达到几十 GB，算上显存带宽成本，每 token 的服务成本让"百万上下文"只是一个营销词汇。

V4.1 Flash 把 KV cache 压缩到原来的 1/4 之后，同样的 HBM 可以服务 4 倍长度的上下文，或者用同样的成本服务更厚的 batch 并发。

具体价格：off-peak $0.15/M 输入，$0.60/M 输出；peak 时间翻倍。这不是最便宜的 API（有些量化版模型可以做到 $0.07/M），但考虑到这是一个 552B 参数、支持原生视觉、1M 上下文的 MIT 许可开源模型，这个价格区间其实相当有竞争力。

## Agentic Benchmark 才是真正的评测维度

媒体喜欢报 MMLU、HumanEval 这类静态评测，但这些分数对实际工作没有指导意义——这些题库早就被刷烂了，GPT-5 能在上面拿高分不代表它写代码比 Claude 更好用。

真正值得看的 agentic 评测，是让模型在真实工具调用环境里完成任务。

V4.1 Flash 的官方数据：

- **Terminal-Bench 2.1**: 90.6 Pass@1（DeepSeek Harness Minimal mode，1M context）
- **DeepSWE v1.1**: 74.2（dsh-minimal agent）
- **HLE with Tools**: 63.9（带工具调用的人类最后考试）

对比同样 1M 上下文跑这类任务的竞品，这些数字是当前开源模型里最好的，且领先幅度不小。

更重要的是：DeepSeek 自己说 V4.1 Flash 在"performance, cost, speed & total runtime"四个维度都领先 V4-Pro。于是他们做了一个商业决策——**从 9月14日起，deepseek-v4-pro 的所有请求自动路由到 V4.1 Flash，按 V4.1 Flash 的价格计费**。V4-Pro 这个型号，等于被下架了。

这不是降价促销，这是用产品迭代的方式宣布：Pro 级旗舰已经被新架构吃掉了。

## MIT 许可：开源世界的真实变量

552B 参数，MIT 许可，可以自托管，可以 fine-tune，可以在 vLLM 和 SGLang 里直接跑。

Hugging Face 上有完整的 technical report 和权重文件（deepseek-ai/DeepSeek-V4.1-Flash）。Transformers、vLLM、SGLang、Docker 都有官方支持路径。Ollama 也已经同步上线，22,500 次下载，更新时间是 6 天前。

MIT 许可在这里是一个被低估的细节。Apache 2.0 和 Llama Community License 都有使用限制，MIT 是目前最宽松的开源许可——允许任何人自由使用、修改、商业化，甚至不需要公开修改后的代码。对于企业用户来说，这意味着内部 fine-tune 不需要担心合规问题，不需要向任何人报备，也不需要担心模型能力迭代后许可条款发生变化。

Qwen3.7-Plus 是闭源的（只有 API），Claude 和 GPT 系列的权重不开放，Llama 的社区许可有合规摩擦——在开源大模型的选择空间里，V4.1 Flash 是目前许可证最干净的旗舰级选项。

## 接下来看什么

**1. V4.1-Pro 什么时候来**

DeepSeek 用 Flash 替换了 Pro，但 Pro 还会回来。按 DeepSeek 的节奏，V4.1-Pro 大概率会在未来 1-2 个月内发布，届时会是完整 552B 激活参数的满血版，而不是现在的 8B/16B 截断版。如果你在等更强的单任务推理能力，等等看。

**2. 自托管的实际成本**

552B 参数 + 196B Engram，即使只激活 8-16B，完整加载也需要多卡。vLLM 和 SGLang 已经支持，但 8xH100 的入门配置大概在 20-30 万人民币的硬件成本。适合有 GPU 集群、但不想要 API 成本和延迟的团队。

**3. CED 架构会不会被其他玩家跟进**

Causal Encoder-Decoder 不是新概念（T5 早就用过），但 DeepSeek 把 MoE + CED + Compressed Sparse Attention 做到生产级别，并且开源了完整的技术报告。如果这个架构在成本-效率上持续验证，2027 年可能会有更多开源项目跟进。

**4. 视觉能力的上限在哪里**

V4.1 Flash 是原生视觉模型，不是把视觉 encoder 拼接在文本模型后面。但 DeepSeek 的官方文档对视觉能力的描述非常保守——只说"images go in the content array as image_url parts"，没有公布任何视觉 benchmark。考虑到这个模型的主要定位是 coding 和 agentic 工作，视觉能力的上限（尤其是 GUI 截图操控、视频帧理解）还需要独立测试。

## 参考资料

- [DeepSeek V4.1 Flash 官方公告](https://www.deepseek.com/en/news/deepseek-v4-1-flash)
- [Hugging Face: deepseek-ai/DeepSeek-V4.1-Flash](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash)
- [LLM Reference: DeepSeek V4.1 Flash 完整规格](https://www.llmreference.com/model/deepseek-v4.1-flash)
- [Ollama Library: deepseek-v4.1-flash](https://ollama.com/library/deepseek-v4.1-flash)
- [阿里云开发者社区: Qwen3.7-Plus 完整解析](https://developer.aliyun.com/article/1760258)
- [AI/TLDR: DeepSeek V4.1 Flash — 552B open-weight rebuild](http://ai-tldr.dev/releases/deepseek-v4-1-flash)
