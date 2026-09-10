+++
date = '2026-09-10T07:32:11+08:00'
draft = false
title = 'SGLang vs vLLM：选推理引擎，不是选谁更强'
description = "SGLang 在 DeepSeek V3 上比 vLLM 快 3.1 倍，但这个数字没有意义——除非你知道自己的 workload pattern。两者真正的差异在于 prefix caching 策略，而非峰值吞吐量。"
tags = ["编程", "技术", "AI"]
categories = ["编程技术"]
author = "Spiral"
+++

选推理引擎的时候，很多人会问「哪个更快」。但如果把 vLLM 和 SGLang 放在一起比，你会发现一个奇怪的现象：同一个模型，在不同的测试场景下，它们轮流胜出。SGLang 在 DeepSeek V3 上跑出 3.1 倍于 vLLM 的速度，但在高并发 unique-prompt 场景下，vLLM 的 C++ PagedAttention 反而扩展得更好。

这不是测试误差，是架构差异带来的固有特性。理解这一点，比记住任何一个 benchmark 数字都重要。

## 核心差异：PagedAttention 和 RadixAttention

vLLM 在 2023 年靠 PagedAttention 改变了游戏规则。它把 KV cache 切分成固定大小的「页」，GPU 内存利用率从此不再被预分配的完整序列长度束缚。这个设计优雅、通用，适合大多数场景。

SGLang 在这个基础上加了一层：RadixAttention。它不只是分页管理 KV cache，还在请求结束后**保留**这些 cache，构建成一棵前缀树（radix tree）。当新的请求进来时，SGLang 会先在这棵树里查找有没有可以复用的前缀——如果你的系统 prompt 是 2000 token，所有请求共享，SGLang 只需要计算一次，剩下的直接复用。

对于 agent 多轮对话和 RAG 场景，这个差异是决定性的。

## 具体数字是什么水平

在 H100 80GB 上跑 Llama 3.1 8B，AI Multiple 的测试结果是：SGLang 16,215 tok/s，vLLM 12,553 tok/s，SGLang 领先约 29%。这个数字是**单次长序列、无前缀复用**的测试，更多反映的是算子优化差异。

但在**前缀复用场景**下，SGLang 的优势会显著扩大。Spheron 的测试显示，当多个请求共享长 system prompt 或 RAG 文档时，SGLang 的 Time-to-First-Token（TTFT）比 vLLM 低 20-40%。原因是已经算过的 KV cache 直接复用，不需要重新处理那几千个 token。

高并发下的情况正好反过来。当并发请求数很高、GPU 需要充分饱和时，vLLM 的 C++ 实现绕过了 Python 层，扩展性更好。而 SGLang 的 radix tree 在这种场景下反而成了开销——管理、前缀匹配的成本开始超过收益。

## DeepSeek V3 为什么特别

DeepSeek V3 是一个 MoE（Mixture-of-Experts）模型，它的 Attention 机制用了 MLA（Multi-head Latent Attention）。SGLang 专门为 MLA 做了后端优化，集成了 FlashAttention3、FlashInfer、FlashMLA、CutlassMLA，这一套下来让 SGLang 在 DeepSeek V3 上的推理速度达到了 vLLM 的 3.1 倍。

这不是通用差距。vLLM 对 MLA 的支持在 DeepSeek V3.2 才开始完善，而 SGLang 已经在这个模型上做了大量针对性优化。如果你的主力模型是 DeepSeek V3 或其后续版本，SGLang 目前是更务实的选择。

## 决策框架：四个问题

**第一个问题：你的 workload 里有没有大量前缀复用？**

如果是——比如大量请求共享 system prompt，或者 RAG 文档会被反复引用——选 SGLang。在这个场景下，radix tree 的前缀缓存会直接变成延迟优势和成本优势。

**第二个问题：你的并发量有多高？**

如果并发请求数量很大，需要 GPU 持续饱和运转，vLLM 的 C++ 实现目前扩展性更好。如果并发量一般，SGLang 的优势区间更宽。

**第三个问题：你用的是什么模型？**

如果是 DeepSeek V3 或 GLM-5 系列，SGLang 的优化更成熟。如果是Llama、Qwen 通用场景，两者的差距在缩小，vLLM 的生态和文档优势可能更重要。

**第四个问题：你的团队熟悉哪个？**

两者都生产可用，但调试推理系统的时候，熟悉度会直接影响排查速度。如果团队已经熟练掌握 vLLM，换引擎的迁移成本可能大于性能收益。

## 接下来看什么

SGLang 最近的 release notes 显示已经支持 DeepSeek V4、Intern-S2-Preview、MinCPM-V 4.6 等新模型，FP4 低延迟路径也在持续优化。vLLM 侧对 MLA 的支持是下一步的关键变量——当 vLLM 完整支持 DeepSeek V3 的 MLA 后，两者在这个模型上的差距可能会显著缩小。

另一个值得关注的方向是 prefix caching 的标准化。SGLang 的 radix tree 目前是一个闭门实现，如果 PagedAttention 社区把类似的前缀缓存机制做进 vLLM，两个引擎的核心差异会从「架构选择」变成「参数调教」。

## 参考资料

- [SGLang vs vLLM in 2026: Benchmarks, Architecture, and Performance](https://particula.tech/blog/sglang-vs-vllm-inference-engine-comparison)
- [vLLM vs SGLang: RadixAttention vs PagedAttention Benchmarks](https://www.spheron.network/blog/vllm-vs-sglang-2026/)
- [vLLM vs SGLang: Performance, Features & Deployment Compared](https://deepinfra.com/blog/vllm-vs-sglang)
- [SGLang GitHub Releases](https://github.com/sgl-project/sglang/releases)
