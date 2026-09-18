+++
date = '2026-09-18T07:32:06+08:00'
draft = false
title = 'SGLang 还是 vLLM：2026年推理引擎选型，不再是二选一'
description = "SGLang 和 vLLM 都在快速进化，功能差距正在收窄。本文从实际场景出发，梳理两个引擎在 DeepSeek V4、MoE 模型、多模态支持上的真实差异，给出可操作的选型决策框架。"
tags = ["编程", "技术", "AI", "LLM"]
categories = ["编程技术"]
author = "Spiral"
+++

2026年了，SGLang 和 vLLM 的对比仍然是 LLM 工程师群里最常被翻出来的问题。但很多人没意识到的是，这个问题本身正在变得过时。

两个框架不再是在同一维度上 PK 的两个选项，而是各自在不同场景里建立了清晰的护城河。本文从最新发布的事实出发，给出一个可操作的决策框架。

## 现状：两个框架都在快速补全功能

先看最新动态。

**SGLang**这边，2026年的节奏很快。2月宣布在 NVIDIA GB300 NVL72 rack-scale 系统上运行 DeepSeek R1 达到 25x 性能提升；4月实现 DeepSeek V4 Day-0 支持，包含完整的 Verified RL pipeline；6月宣布对 Nemotron 3 Ultra、Nemotron 3 Super、Higgs Audio v3 TTS 提供 Day-0 支持。GPU recovery 优化也在推进，大模型重启时间从 6.5 分钟降到更短（针对 Qwen3-235B FP8 在 4 GPU 上的实测）。

**vLLM**这边，2026年的核心动作是 Model Runner V2（MRV2）成为默认配置。9月最新版本（v0.32 前后）正式废弃了 Model Runner V1，MRV1 相关优化停止接受新 PR。4月版本（0.18/0.19）引入了 gRPC serving（对高并发场景有直接影响）、GPU 加速的 speculative decoding，以及 KV-cache offloading 到 CPU/NVMe。Chord 项目（与 Novita AI 合作）让 INT4 MoE 在 H200 上获得 1.3x 加速，在未调优的 B300 上达到 2.15x。

一个值得注意的趋势：**两个框架对主流模型的 Day-0 支持速度几乎拉平**，不再是"谁先支持新模型谁就赢"的局面。

## DeepSeek V4：两个引擎的真实表现差异

具体到 DeepSeek V4 这个 2026 年最重要的开源模型，两者的支持情况如下：

- SGLang：Day-0 支持，包含 FP8/BF16 rollout、TileLang kernels、稀疏 MLA、FP4 MoE。在 AMD MI355X 上实测 110.5x throughput 提升（26天内从 20 tok/s/GPU 到 2256 tok/s/GPU）。
- vLLM：同样 Day-0 支持，FP4 MoE 后端、FP8/BF16、disaggregated prefill/decode。

但有一个细节值得关注：SGLang 的 DeepSeek V4 博客里提到了"Miles"——一个与 SGLang 配合的 RL 训练框架，实现 Step-0 train-inference diff 仅 0.02-0.03。对于做 RLHF 训练的团队，这是 SGLang 侧的隐性优势。

两者都没有官方支持 DeepSeek V4 的 TGI（Text Generation Inference），这个空白意味着如果你在这个时间点选 TGI，将面临模型支持断档。

## 选型决策矩阵：看场景而不是看 benchmark

benchmark 数据是选型的起点，但不是终点。以下是 2026 年第三季度实测和社区反馈的综合判断：

### 场景一：AMD GPU（MI355X/MI300X）

**推荐 SGLang**。

AMD ROCm 生态对 vLLM 的支持一直不够稳定，而 SGLang 在 MI355X 上的实测数据非常扎实：DeepSeek-V4-Pro FP4 MoE 权重从 159GB 降到 112GB（减少 29%），配合 memory_saver KV-cache 优化，单 GPU 训练 footprint 从 143 GiB 压到 87 GiB。110.5x throughput 提升的数字也来自这个平台。

如果你用的是 AMD 硬件，别犹豫，直接上 SGLang。

### 场景二：NVIDIA H200/B200 系列，大规模 MoE 推理

**两者均可，倾向 vLLM 的 Chord 方案**。

vLLM 与 Novita AI 合作的 Chord 在 H200 上对 INT4 MoE 有 1.3x 加速，在 B300 上达到 2.15x。这个收益在规模化部署时是真实的成本节省。

但 SGLang 在 GB300 NVL72 上的 25x 性能数据同样来自大规模部署场景，两者各有据点。

### 场景三：RLHF/RL 训练 + 推理联合 pipeline

**推荐 SGLang**。

SGLang 与 Miles 框架的 Verified RL pipeline 提供了开箱即用的 train-inference 闭环。vLLM 在这方面的集成文档和社区案例都相对较少。

如果你在做 RL 训练（不管是 DPO、GRPO 还是 PPO），推理侧选 SGLang 可以减少很多集成摩擦。

### 场景四：需要 gRPC 高并发接口

**推荐 vLLM**。

vLLM 0.18+ 引入了原生 gRPC serving，对已有 gRPC 基础设施的团队是直接利好。SGLang 的 API 层以 REST 为主，gRPC 支持不在官方 roadmap 的显眼位置。

### 场景五：多模态模型（视觉/语音）

**倾向 SGLang**。

SGLang 官方在 2026 年明确提到了 multimodal model 支持，且对 TTS 模型（如 Higgs Audio v3）有 Day-0 支持。vLLM 的多模态支持在 2026 年有所改善，但 SGLang 在这个领域的先发优势仍然存在。

## 一个反直觉的结论：选哪个可能没你想的那么重要

说了这么多场景，实际上有相当大比例的团队，两个引擎的差异并不是选型的核心变量。

真正影响体验的是：

- **团队对 CUDA/Python/异步编程的熟悉程度**——两个框架都有一定的运维学习曲线
- **基础设施的自动化程度**——SGLang 和 vLLM 都支持 Ray 分布式，对已有 Ray 集群的团队更友好
- **Vendor 锁定容忍度**——SGLang 对 FlashInfer 等内核的依赖相对紧耦合，vLLM 的内核抽象层更灵活

如果你现在跑的是 vLLM 0.18+ 并且运行稳定，没有必要因为"听说 SGLang 更快"就迁移。迁移成本是真实的，而性能差距在很多场景下并不显著。

## 接下来看什么

1. **NVIDIA Blackwell Ultra（B300）生态成熟度**：两个框架对 B300 的优化还在快速迭代，2026 Q4 可能会有新的格局变化
2. **MRV2 的生产稳定性**：vLLM 的 Model Runner V2 刚成为默认，生产案例积累还不够，想上车的建议先跑一轮自己的 benchmark
3. **SGLang 的企业级特性**：多租户、限流、审计日志这些企业场景必备功能，两个框架的完善程度都还在追赶

选引擎这件事，2026 年已经不再是"哪个最强"，而是"哪个最适合你现在的团队和场景"。这个答案只有你自己能回答。

## 参考资料

- SGLang Releases: https://github.com/sgl-project/sglang/releases
- DeepSeek-V4 on Day 0 with SGLang: https://www.lmsys.org/blog/2026-04-25-deepseek-v4/
- vLLM Release Notes (Sept 2026): https://docs.nvidia.com/deeplearning/frameworks/vllm-release-notes/index.html
- MI355X DeepSeek-V4-Pro on SGLang (InferenceX): https://inferencex.semianalysis.com/blog/mi355x-deepseek-v4-pro-sglang-110x-in-26-days
- vLLM Blog: https://blog.vllm.ai/
- Best LLM Inference Engines 2026 (Yotta Labs): https://www.yottalabs.ai/post/best-llm-inference-engines-in-2026-vllm-tensorrt-llm-tgi-and-sglang-compared
