+++
date = '2026-09-17T07:35:00+08:00'
draft = false
title = '阿里云百炼批量下架背后：DeepSeek V3/R1 迁移倒计时30天'
description = "2026年10月10日，阿里云百炼将下线DeepSeek V3/V3.1/V3.2/R1全线模型。本文追踪这一事件的技术与商业影响：QPM已从公告起开始缩减，迁移窗口比多数人意识到的更窄。"
tags = ["AI", "AI观察"]
categories = ["AI观察"]
author = "Spiral"
+++

阿里云百炼的 DeepSeek 接入页面上悄然多了一条横线——"2026年10月10日下线"。这不像一次版本迭代，更像一次战略撤退：DeepSeek V3、V3.1、V3.2、R1、蒸馏系列全部在列。官方替代建议只有一行字：Qwen3.7-plus、Qwen3.7-max、Qwen3.6-flash。

这不是一次常规的模型退役。

## 三个事实，先说清楚

**一、QPM 已开始缩减，不是等 10 月 10 日才停。**

百炼的公告写得很隐蔽：速率限制从公告发布之日而非下线之日开始收紧。已申请过 DeepSeek 模型扩容配额的账号，系统会先恢复默认限制，再逐级压缩。这意味着今天（9月17日）跑着 `deepseek-v3` 的服务，QPM 可能已经比两周前低了 30%–50%，只是没人告诉你。来源：阿里云百炼官方文档（`help.aliyun.com/zh/model-studio/deepseek-api`）

**二、迁移目标只有一个：回归原厂 API。**

百炼的第三方覆盖本质是代理——模型名相同（`deepseek-v3`），但 Base URL、鉴权格式、配额体系完全不同。通过百炼使用 `deepseek-v3` 的团队，无法直接将配置迁移到 DeepSeek 原生 API（`api.deepseek.com`），因为这是两个独立接入点。来源：TheRouter.ai 追踪报道（`therouter.ai/zh/news/dashscope-third-party-model-retirement-october-2026-routing/`）

**三、这不是国内独有的节奏。**

2026年8月，Qwen3.8-2.4T-A95B 以开源权重形式发布，AWS SageMaker HyperPod 已支持一键部署（`aws.amazon.com/blogs/machine-learning/deploying-qwen3-8-2-4t-a95b`）。这与百炼下架 DeepSeek 几乎是同一时间段——阿里一边关闭代理，一边加速推开源旗舰。两者之间有联系，但官方从未明说。

## 反共识视角：这不是「阿里 vs DeepSeek」

媒体对这个事件的描述通常是「阿里下架 DeepSeek」，暗示两家是竞争关系。这是错的，至少不准确。

DeepSeek 的 API 一直在 `api.deepseek.com` 正常运营，百炼只是一个分销量更大的分销渠道。当分销渠道开始收缩，通常只有一个解释：**利润空间不够维持**。DeepSeek V3 的定价（每百万输入 tokens 0.1–1 元，输出 2 元）在 2025 年 2 月涨价后依然远低于 GPT-4o 等同类产品，靠的是低价换开发者生态。当阿里需要为这个「国产平替」承担更多的算力成本和监管风险时，退出是最理性的选择。

这不是背叛，是商业逻辑。开发者需要理解这一点，而不是停在「阿里不支持 DeepSeek」的情绪里。

## 为什么这个时间点值得关注

多数技术团队真正的问题不是「10月10日之后怎么办」，而是「现在 QPM 已经变低，但我还不知道」。

具体风险场景：

- **AI Agent 生产线**：如果你的自动化流程依赖百炼的 DeepSeek V3 做长程推理，速率限制收紧会导致任务超时率上升，而告警通常不会配置到这个层级
- **API 网关配置**：所有写在网关层解析到 `dashscope` 后端的模型名称（`deepseek-v3` / `deepseek-r1`），在 10 月 10 日后直接 404，但代码里看不到
- **成本核算**：百炼下架后，如果开发者慌忙转向 DeepSeek 原生 API，配额体系不同，费用结构可能突然跳升

## 迁移路径：三个层次

**层级一：模型 ID 替换（最快，但最浅）**

把 `dashscope` base URL 换成 `api.deepseek.com`，把模型名从 `deepseek-v3` 换成 `deepseek-v3`（名称相同，但鉴权 token 不同）。这是最小改动，但不解决配额和费用问题。

**层级二：多 API Key 管理（推荐）**

在应用层同时接入百炼（Qwen3.7 系列）和 DeepSeek 原生 API，用模型路由层做流量分配。百炼的 Qwen3.7-plus 现在是推荐主力：1M context、支持 function calling、支持结构化输出，价格比 DeepSeek V3 略高，但在百炼体系内有更稳定的 QPM 保障。

**层级三：本地开源部署（最稳，但有运维成本）**

Qwen3.8-2.4T-A95B 已开源权重，2.4T 参数、95B 激活参数、1M context，在 AWS SageMaker HyperPod 或自有 GPU 集群上可用 vLLM 部署。这个选项适合对数据主权有要求、或月度 API 费用已经超过自托管成本的团队。

## 接下来看什么

- **DeepSeek 官方是否会推出更激进的定价或配额来承接百炼流出的开发者**（这在 V3 发布时做过一次，2025年2月涨价逆转了局面）
- **阿里云是否会保留 DeepSeek 的「推荐迁移」入口**，还是彻底清场
- **百炼 QPM 缩减的实际节奏**：开发者的线上监控数据比官方公告更真实——如果你的日均 QPM 利用率开始下滑，那就是信号
- **其他国内代理平台（硅基流动、火山引擎等）是否跟进类似下架**，还是把 DeepSeek 作为差异化竞争点

---

## 参考资料

1. 阿里云百炼 DeepSeek 模型文档 — `help.aliyun.com/zh/model-studio/deepseek-api`
2. 阿里云百炼 10 月模型下线追踪 — `therouter.ai/zh/news/dashscope-third-party-model-retirement-october-2026-routing/`
3. Qwen3.8-2.4T-A95B 开源权重发布 — `aws.amazon.com/blogs/machine-learning/deploying-qwen3-8-2-4t-a95b`
4. 腾讯云知识引擎 DeepSeek 模型接入 — `cloud.tencent.com/document/product/1772/115969`
5. Qwen 模型家族总览 — `www.alibabacloud.com/help/en/model-studio/text-generation-model`
6. QwenCloud 模型发布日志 — `docs.qwencloud.com/changelog/models`
