+++
date = '2026-09-05T07:31:26+08:00'
draft = false
title = "OBI：零代码观测卷到内核层，OpenTelemetry 用 eBPF 革了 auto-instrumentation 的命"
description = "2025年，OpenTelemetry 将 eBPF 自动注入正式纳入标准，Grafana Labs 捐出 Beyla 项目，多家厂商联合推进 OBI alpha。这一动作意味着什么，零代码观测真的能替代代码侵入式埋点吗？"
tags = ["编程", "技术", "可观测性", "eBPF", "OpenTelemetry"]
categories = ["编程技术"]
author = "Spiral"
+++

2025年5月，OpenTelemetry 成立了一个新的特别兴趣小组：OTel eBPF Instrumentation SIG。参与者名单里不乏 Grafana Labs、Splunk、Coralogix、Odigos 这些厂商的名字。三个月后，这个小组交出了第一份作业——OBI（OpenTelemetry eBPF Instrumentation）的 alpha 版本正式发布。

这件事在技术圈没有激起太大的水花，但它指向的方向值得关注：**可观测性这场仗，正在从「应用层」向「内核层」蔓延**。

## 零代码埋点的问题，出在根子上

在说 OBI 之前，先要把传统 auto-instrumentation 的局限说清楚。

现在的 OpenTelemetry SDK 埋点，本质上是在应用代码里「插桩」——在 HTTP handler 入口记录时间戳，在数据库调用前后打日志，在消息队列收发时记录 span。效果不错，但代价也明显：

**每次引入新的观测能力，都要改代码、升依赖、发版本。** 一个运行了三年的 Java 服务想从 HTTP 埋点升级到 gRPC 链路追踪，很可能要等两个 sprint 才能排上。还有那些根本拿不到源码的二进制依赖，或者历史遗留的 PHP/Node.js 老服务——这些场景里，SDK 埋点根本进不去。

更根本的问题是：SDK 埋点只能覆盖**主动接入的部分**。如果框架层面没有埋点（比如某个冷门 Web 框架的中间件），业务代码想补也补不上。

## eBPF 凭什么能「无感」观测

OBI 的核心思路是绕过应用层，直接在内核层面抓数据。

具体来说，OBI 用 eBPF 程序完成两件事：一是**协议识别**——在网络层解析出 HTTP/gRPC/PostgreSQL/MySQL/Redis 等常见协议的请求和响应；二是**自动关联**——把网络 I/O 事件和进程级别的事件对应起来，生成带 trace ID 的 span。

这个过程完全发生在内核态，应用进程本身不需要加载任何 agent，也不需要重启。用 OTel 官方文档的话说：「no rebuilds, no dependency bumps, no rollout cycles — it runs entirely out-of-process」。

项目最初叫 Grafana Beyla，由 Grafana Labs 开发。2025年，Grafana Labs 将 Beyla 捐赠给 OpenTelemetry 成为 OBI 的基础代码，这是继 Otel 接收 ebpf.io 项目之后在 eBPF 方向的第二次重要收购。

## OBI 能做什么，不能做什么

alpha 阶段的 OBI 已经支持以下场景：

- **HTTP/1.1、HTTP/2、gRPC** 请求追踪
- **PostgreSQL、MySQL、Redis、MongoDB** 的数据库调用链路
- 容器环境下的**进程-网络拓扑**自动发现
- 标准的 OTel 格式输出，兼容 Jaeger、Zipkin、Grafana、Tempo 等后端

但它有明确的边界：

**第一，它只能看到「系统层面可见」的东西。** 应用内部的分层逻辑、业务语义的嵌套 span，它无能为力。你想在同一个数据库调用里区分「查询用户余额」和「更新订单状态」这两个业务操作？对不起，eBPF 层面分不出来。

**第二，eBPF 本身有平台限制。** 目前只支持 Linux，需要内核开启 BPF 相关特性。macOS 开发环境和 Windows 容器场景暂时用不上。

**第三，协议覆盖有盲区。** 自定义二进制协议、私有 RPC 框架——这些不在支持列表里，得靠传统埋点补位。

## 反共识：OBI 不是银弹，但它补了关键缺口

媒体叙事通常会把这类发布包装成「革命性替代」——「零代码将取代 SDK 埋点」。这个说法经不起推敲。

真实情况是：OBI 和 SDK 埋点解决的是**不同层次的问题**。SDK 埋点拿的是应用语义的精确描述，OBI 拿的是网络 I/O 的粗粒度轮廓。两者不是替代关系，而是互补关系。

OBI 真正有价值的地方在于**兜底**——那些从来没埋过点、源码失联、团队已经没人维护的老服务，用 OBI 可以立刻拿到可观测数据，不需要任何改造成本。这才是它最实际的使用场景。

换一个角度想：如果一家公司的所有服务都能通过 SDK 埋点做到完整观测，OBI 的价值就打折扣。它的目标用户是**埋点覆盖率长期低于 50%** 的团队——这类团队在现实中大量存在，往往是历史包袱重、技术债务高的组织。

## 接下来看什么

OBI 仍处于 alpha 阶段，以下几个信号值得持续关注：

**1. KubeCon EU 2026 的 beta 发布。** 届时 Splunk 会联合展示 OBI 在 Kubernetes 环境下的完整集成方案，包括 Operator 级别的部署体验。如果 beta 能在生产环境稳定性上有所突破，会是进入企业采购清单的关键节点。

**2. 协议覆盖的扩展速度。** 目前 alpha 只支持 6 种协议（HTTP、gRPC、PostgreSQL、MySQL、Redis、MongoDB）。消息队列（Kafka、RabbitMQ）、缓存层（Memcached）、以及云服务的专有协议（如 AWS S3）能否进入支持列表，决定了它能覆盖多宽的存量系统。

**3. SIG 测试 pipeline 的改进。** 根据 OTel 社区的公开信息，SIG 成立后的测试 pipeline 速度提升了 10 倍。这意味着版本迭代节奏会加快，但也意味着 API 稳定性风险上升——早期采用者需要接受较高的变更成本。

**4. 云厂商的跟进态度。** AWS、GCP、Azure 各自的观测产品（CloudWatch、GCP Monitoring、Azure Monitor）是否会官方支持 OBI 数据格式，或者推出兼容层，将决定 OBI 在云原生生态里的最终渗透率。

---

eBPF 从网络安全场景走向可观测性，是这几年基础设施领域最清晰的技术趋势之一。OBI 让这个趋势正式进入了 OpenTelemetry 的标准版图。但「零代码」从来不等于「零成本」——部署、运维、排障依然需要投入，只是把成本从「开发侧」转移到了「基础设施侧」。哪个团队愿意先吃这只螃蟹，2026 年会是观察窗口。

## 参考资料

- OpenTelemetry Blog:「OpenTelemetry eBPF Instrumentation Marks the First Release」，https://opentelemetry.io/blog/2025/obi-announcing-first-release/
- OpenTelemetry 官方文档: OBI 介绍，https://opentelemetry.io/docs/zero-code/obi/
- Last9:「OTel Updates: OpenTelemetry eBPF Instrumentation (OBI) Hits Alpha」，https://last9.io/blog/opentelemetry-ebpf-instrumentation/
- The New Stack:「How eBPF Is Powering the Next Generation of Observability」，https://thenewstack.io/how-ebpf-is-powering-the-next-generation-of-observability/
- eBPF Foundation:「The eBPF Foundation's 2025 Year in Review」，https://ebpf.foundation/the-ebpf-foundations-2025-year-in-review/
- Cloud Native Now:「Splunk Introduces OpenTelemetry eBPF Instrumentation and Kubernetes Operator at KubeCon EU 2026」，https://cloudnativenow.com/kubecon-cloudnativecon-europe-2026/splunk-introduces-opentelemetry-ebpf-instrumentation-and-kubernetes-operator-at-kubecon-eu-2026/
