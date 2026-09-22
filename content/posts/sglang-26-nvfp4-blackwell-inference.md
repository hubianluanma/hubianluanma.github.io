+++
date = '2026-09-22T07:30:00+08:00'
draft = false
title = 'SGLang 26.08 支持 GB300：Blackwell 上的推理战争进入精度损耗新阶段'
description = "SGLang 26.08 全面支持 GB300 与 NVFP4 量化，GB300 NVL72 相对 H200 提升 25 倍吞吐量。精度从 FP16 到 NVFP4，内存占用腰斩，吞吐量翻倍——但代价是什么？"
tags = ["编程", "技术"]
categories = ["编程技术"]
author = "Spiral"
+++

SGLang 26.08 在 8 月底发布，最大的变化不是某个新 API，而是**对 NVIDIA GB300（Blackwell Ultra）的完整支持**——包括让整个推理工程界既兴奋又忐忑的 NVFP4 量化格式。MLPerf Inference v5.1 的官方记录里，GB300 NVL72 相对 H200 跑 DeepSeek-R1 提升了约 5 倍每 GPU 吞吐量，45% 的提升来自 GB300 本身，剩下的大头来自 NVFP4。

这不是 SGLang 一家的事。整个推理引擎生态——vLLM、TensorRT-LLM、SGLang——都在围绕 Blackwell 架构重新校准自己的精度/性能边界。本文拆开来看，**这轮量化升级真正改变了什么，开发者现在面临哪些新的选型摩擦**。

## NVFP4 是什么， 为什么这次不一样

NVIDIA 在 Blackwell 架构引入了 NVFP4，这是 4 位浮点格式，但它的设计思路和之前的 FP6/INT4 有本质区别。

传统低位量化（INT4/FP4）最大的坑是精度崩塌：4 位只能表示 16 个值，模型权重稍微复杂一点，量化误差就大到让输出偏离。NVFP4 用了 E4M3 的缩放因子加上两层微块策略——简单说，它给每个很小的块都单独维护了一个高精度的缩放系数，这样即使在 4 位表示下，也能把精度损失压到可接受范围。

具体数字：NVFP4 相对 FP16 内存占用减少约 3.5 倍，相对 FP8 减少约 1.8 倍。在 GB300 NVL72（288GB HBM3e × 72 GPU）上，这直接意味着**同一个模型可以开更大的 KV Cache，或者并发更多用户请求**。

NVIDIA 自己的技术博客给出的数字是：Blackwell Ultra（GB300）相对 H100 在 NVFP4 工作负载下，能效提升 25 倍。这不是跑分，是实际推理场景。

## GB300 NVL72 的硬数字：25 倍吞吐量从哪来

LMSYS 和 NVIDIA 联合发布的 GB300 数据有几个关键节点：

- DeepSeek R1 推理，GB300 NVL72 相对 H200：**每 GPU 吞吐量提升 25 倍**（InferenceMAXv2 基准）
- GB200 NVL72（Blackwell，非 Ultra），SGLang 4 个月内性能提升 8 倍，靠的是 MTP（Multi-Token Prediction）+ disaggregated prefill/decode
- GB300 NVL72 用上 NVFP4 之后，单 GPU 跑 DeepSeek-V4 在相同用户感知延迟下，吞吐量从 Day-0（2026 年 4 月）的 ~2200 tok/s/GPU 提升到 ~11200 tok/s/GPU——5 倍

这 25 倍不是一个魔法数字，它是**硬件架构提升 + 量化格式节省显存 + disaggregated serving 调度优化**三层叠加的结果。拆开看每一层：

**硬件层**：GB300 的 Tensor Core FP4 峰值算力比 B200 高 1.5 倍，HBM3e 容量是 B200 的 1.5 倍（288GB），带宽也对应提升。NVLink Switch 的跨 GPU 通讯开销在 MoE 模型里是瓶颈，GB300 的 NVLink 拓扑做了优化。

**量化层**：NVFP4 让 KV Cache 可以压缩。DeepSeek-V4 这类 MoE 模型，KV Cache 体积随上下文长度线性增长，是并发量的硬约束。FP4 KV Cache 把同样显存的利用率拉高了一截。

**调度层**：SGLang 的 RadixAttention 在长上下文场景（agentic 场景、多轮对话）里复用共享前缀，这个能力在 GB300 上被放大——更大的 KV Cache 意味着更多请求可以被缓存命中，减少重复计算。

## SGLang 26.08 的关键变化清单

SGLang 26.08（对应 SGLang 0.5.17，基于 CUDA 13.4.1）支持列表：

- GB300/B300 单节点及多节点配置
- DGX Spark、Jetson Thor
- FP8 精度（Hopper 及以上）
- **NVFP4 精度（Blackwell 全系列，含 Jetson Thor、DGX Spark）**
- RTX PRO 6000 Blackwell Server Edition

对于已经在用 SGLang 的团队，这次升级路径相对平滑——主要是 container 镜像更新 + 启动参数加 `--kv-cache-dtype mxfp4` 或 `--kv-cache-dtype fp8_e4m3`。但有几个需要注意的坑：

**坑 1：NVFP4 KV Cache 是 Blackwell only 的**。Hopper（H100/H200）不支持，想在 Hopper 上用同款精度省显存，只能选 FP8，没有平替。

**坑 2：MTP（Multi-Token Prediction）在 GB300 上需要显式开启** `--enable-multi-layer-eagle`，且需要对应固件版本。已在 B200 上验证过的配置迁移到 GB300 时，部分边缘场景需要重新调参。

**坑 3：FP4 checkpoint 不是所有模型都有**。Qwen3-235B、DeepSeek-V4 这类主流模型在 NVIDIA exemplar 仓库里有 FP4 recipe，但很多垂直领域小模型目前没有官方 FP4 权重，需要自己跑量化流程（PTQ/QAT），这块工程成本不低。

## vLLM 在做什么：两条路线的分化

SGLang 激进押注 Blackwell + NVFP4，vLLM 的路线图在 2026 年走的是**调度架构重构**。

vLLM 0.25.0（2026 年 7 月发布）最大的变化是 Model Runner V2 成为默认——它把客户端调度层和 GPU 执行循环拆成了两个独立进程，通过 ZMQ 通信。这解决了高并发场景下 Python 调度侧吃 decode 时间的问题。

vLLM 的 PagedAttention 思路被 SGLang 学过去了（现在 SGLang 也支持 PagedAttention），但 vLLM 新的重心是在**工程化可靠性**——v1 架构的进程分离让内存管理更干净，不容易因为某个请求的异常 OOM 导致整个服务崩溃。

两条路线的分歧点在这里：**SGLang 追求极致吞吐量，vLLM 追求极致稳定性**。选哪个，取决于你的 SLA 是"4 个 9 可用"还是"单请求延迟 p99 压到 500ms 以下"。

## 精度损耗：被低估的风险项

NVFP4 宣传的"精度损失可忽略"，是在标准 benchmark（GSM8K、MMLU）上量的。实际生产场景的 prompt 分布和 benchmark 分布差异巨大——特别是代码生成、数学推导、长文档分析这类任务，4 位量化偶尔会出奇奇怪怪的"软错误"：输出语法正确、长度正常，但逻辑链断了或者引用数字错了。

PyTorch 博客提到，DeepSeek-V4 在 GB300 上用 NVFP4，GSM8K 准确率是 96.74%（FP8 对比基线是 97%+）。0.3% 的差距在论文里不算啥，但在生产环境里，如果你的用户每 1000 次问答遇到 3 次错误答案，这个比例就不算低了。

NVIDIA 也承认，PTQ（post-training quantization）在某些场景需要额外验证流程，建议有条件的团队做端到端回归测试。

## 接下来看什么

1. **vLLM 对 Blackwell Ultra 的支持进度**：目前 NVIDIA 官方合作的框架里 SGLang 优先级更高，vLLM 的 Blackwell 支持预计在 0.26.x 系列补齐，届时两家的直接对比数据才有参考价值。

2. **NVFP4 从推理往训练渗透的速度**：NVIDIA 的 NVFP4 Technical Blog 提到可以在 pretraining 阶段使用 NVFP4，如果这成熟了，训练成本会再下一个台阶，但生态工具链（分布式训练框架、梯度累积精度）还需要时间。

3. **国产 GPU 的 NVFP4 对应方案**：燧原科技 T20/21 目前没有公开的 4 位浮点支持信息，推理精度路线图还在 FP8 阶段。如果 Blackwell 这轮NVFP4 打开的成本优势足够大，国产替代的窗口会被进一步压缩。

4. **MLPerf v5.1 的后续跟进**：GB300 NVL72 是第一次提交，NVIDIA 的对手（AMD、Intel）还没有对应参赛成绩。等下一轮测试结果，才能更客观判断 Blackwell Ultra 的实际优势是否在推理场景全面兑现。

---

**参考资料**

- SGLang Release 26.08, NVIDIA Documentation Hub, 2026-08: https://docs.nvidia.com/deeplearning/frameworks/sglang-release-notes/rel-26-08.html
- Introducing NVFP4 for Efficient and Accurate Low-Precision Inference, NVIDIA Technical Blog, 2026: https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference
- NVIDIA Blackwell Ultra Sets New Inference Records in MLPerf Debut, NVIDIA Technical Blog, 2026: https://developer.nvidia.com/blog/nvidia-blackwell-ultra-sets-new-inference-records-in-mlperf-debut
- Unlocking 25x Inference Performance with SGLang on NVIDIA GB300 NVL72, LMSYS Org / NVIDIA, 2026-02-20: https://lmsys.org/blog/2026-02-20-gb300-inferencex/
- Serving DeepSeek-V4 on GB300 with SGLang: 5x Higher Throughput at the Same Interactivity Since Day-0, PyTorch Blog, 2026: https://pytorch.org/blog/serving-deepseek-v4-on-gb300-with-sglang-5x-higher-throughput-at-the-same-interactivity-since-day-0/
- vLLM vs SGLang: Performance, Features & Deployment Compared, DeepInfra, 2026-08-04: https://deepinfra.com/blog/vllm-vs-sglang
