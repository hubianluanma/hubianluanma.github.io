+++
date = '2026-06-18T08:00:00+08:00'
draft = false
title = '微软一统AI开发框架：Agent Framework 1.0 把牌摊开了'
description = "微软将 AutoGen 和 Semantic Kernel 合并为 Agent Framework 1.0，4月正式发布，支持 MCP 和 A2A 协议。但这次合并是否真的对开发者有利，还是又一场供应商锁定？"
tags = ["编程", "技术", "AI", "协议"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/18/img_1781740947_1.jpeg", alt = "two rivers merging into one unified stream, enterprise architecture concept" }
+++

4月3日，微软正式发布 Agent Framework 1.0，把两套用了两年的 AI 开发库——AutoGen 和 Semantic Kernel——合并成一套 SDK。消息一出，开发者社区的反应两极分化：有人叫好"终于不用学两套了"，有人担忧"所有鸡蛋放一个篮子"。但很少有人注意到，这场合并背后的真正驱动力，既不是微软的工程美学，也不是开发者的体验，而是**企业采购部门的压力**。

## 两套框架，两年折腾

事情得从两年前说起。2024年，AI Agent 概念开始爆发，微软同时维护着两套定位完全不同的库：

- **Semantic Kernel**：面向企业应用，强调 session 状态管理、类型安全、Filters 拦截器、Telemetry，定位是"企业级 AI 应用框架"
- **AutoGen**：面向研究场景，强调多 Agent 协作的易用性，定位是"快速原型和多 Agent 实验"

两套库都能用，但互相不兼容。企业在招聘时需要区分"会用 Semantic Kernel"和"会用 AutoGen"的工程师，团队内部也存在技术路线之争。更要命的是，随着 MCP（Model Context Protocol）和 A2A（Agent-to-Agent）协议逐渐成为行业事实标准，两套库谁先支持、怎么支持，又成了新的分裂点。

## 供应商合并：解决的是采购问题，不是工程问题

微软的逻辑很简单：与其让企业在采购清单上写两套工具，不如合并成一套统一兜售。Agent Framework 1.0 保留了 Semantic Kernel 的内核（kernel、plugin model、connector system），同时吸收了 AutoGen 的多 Agent 协作抽象，一次性支持 MCP 动态工具发现和 A2A 跨框架协作。

对于企业采购来说，这确实是好消息：**一个合同、一个支持窗口、一套 SLA**。但对于实际写代码的工程师，这个合并的代价是：

AutoGen 从此进入维护模式，不再接受新功能 PR。这意味着 2026 年之后还在用 AutoGen 的团队，实际上是在维护一个不再进化的开源项目，而不是迁移到 Agent Framework，就是等死。

## 真正的问题：LLM 生产环境的失败率

不管框架怎么合并，有一个数字被这次发布的热度掩盖了：

> Datadog 2026 年 2 月追踪了线上 LLM 调用的错误率，**5% 的所有 LLM span 报告了错误，其中 60% 是 rate limit 导致的容量问题**。到了 3 月，这个比例仍然维持在 30%。

换句话说，**企业在为"用哪个框架"争论的时候，真正的生产问题是：API 配额不够用、重试逻辑把上游打垮、模型更新静默改变了输出质量**。

这引出了一个更根本的问题——LLM 的"静默退化"。

## 模型在更新，但你不知道它变了什么

2月发表在 PLOS ONE 的一篇论文，对主流 transformer 服务进行了十周的人类锚定纵向评估，证实了**有意义的 Behavioral Drift（行为漂移）**确实存在于部署后的线上服务中。作者指出，由于模型提供商不发布更新日志或训练细节，"任何对观察到的退化原因的归因都纯属推测"。

这个发现对企业意味着什么？企业在生产环境里跑的模型，上线时测过一遍，但三个月后模型版本更新了，没人知道输出是否还一致。更没有人在跑回归测试。

LangGraph 能在这种情况下成为最活跃的 Agent 编排框架（32,000+ GitHub stars，Klarna、Uber、LinkedIn 在生产环境使用），并不是因为它解决了静默退化问题，而是因为它的图结构状态管理让"哪里出了问题"更容易定位。

## 框架收敛，但协议战争还没打完

微软把 Agent Framework 1.0 定位成"MCP + A2A 的原生支持者"，这本身就是在赌协议的未来走向。MCP 让 Agent 能动态发现和调用外部工具，A2A 让不同框架构建的 Agent 能互相协作——这两个协议如果真的成为 W3C 或 ISO 标准，意味着**框架层面的差异将被协议层抹平**。

但这恰恰是微软合并框架最微妙的地方：它把赌注押在了"MCP + A2A 会赢"上，同时把 AutoGen 和 Semantic Kernel 的用户都赶到 Agent Framework 这条船上。等这两个协议真的成为标准时，微软已经是船本身了，而不是乘客。

这是工程决策，还是生态锁定？答案是：**两者兼有**。微软做了正确的事，但理由未必如宣传的那么纯粹。

## 接下来看什么

1. **Agent Framework 1.0 的企业采纳数据**：微软说"生产级支持承诺"，但哪些财富 500 强真的在用它替换现有 Agent 系统？Q3 财报前的技术选型报告会有答案。

2. **MCP 协议的标准化进展**：目前 MCP 是 Anthropic 主导的社区项目，Anthropic 以外的厂商支持程度参差不齐。如果 MCP 进入正式标准进程，Agent Framework 的"MCP 原生"优势会被稀释。

3. **Datadog Q3 LLM 可靠性报告**：5% 的失败率在容量优化后能否下降，还是随着 Agent 调用链变长而继续恶化？多 Agent 串联的失败率叠加是指数级的，一个 Agent 失败率 5%，两个串联就是 9.75%。

4. **AutoGen 维护终止时间线**：微软说 2026 年内做迁移规划，但 AutoGen 的 GitHub 还有大量未关闭的 issue 和 RFC。迁移窗口有多长，迁移成本有多高，会直接影响现有 AutoGen 用户的生产系统稳定性。

---

**参考资料**

- [Microsoft Agent Framework 1.0 官方发布](https://devblogs.microsoft.com/agent-framework/microsoft-agent-framework-version-1-0/)
- [AutoGen 与 Semantic Kernel 合并分析](https://medium.com/@dhirendrachoudhary_96193/microsoft-agent-framework-1-0-semantic-kernel-and-autogen-are-dead-efc486d9a6ac)
- [Datadog State of AI Engineering 2026](https://www.datadoghq.com/state-of-ai-engineering/)
- [PLOS ONE: LLM Behavioral Drift 论文](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0339920)
- [LangGraph 生产环境部署](https://www.alphabold.com/langgraph-agents-in-production/)
- [A2A Agent 协议文档](https://learn.microsoft.com/en-us/agent-framework/agents/providers/agent-to-agent)
