---
title: "不要在主机器上运行 OpenClaw：安全研究者的警告"
date: 2026-03-25
draft: false
tags:
  - OpenClaw
  - 安全
  - 隔离
  - 提示注入
  - AI Agent
summary: "云平台 SkyPilot 发文警告：OpenClaw 在你的主力电脑上运行是一个难以辩护的风险。prompt injection 未解决，漏洞已被野生利用，设计本身就需要广泛系统权限——隔离才是正解。"
---

# 不要在主机器上运行 OpenClaw：安全研究者的警告

OpenClaw 很强大。但强大是有代价的。

SkyPilot（一个云计算/集群管理平台）昨天发了篇博客，标题直接：**"不要在主机器上运行 OpenClaw"**。

## 三个无法反驳的理由

### 1. Prompt Injection 是未解决的安全问题

这是核心问题。OpenClaw 的设计让它可以接收自然语言指令并执行操作——这正是它的价值，也是它最大的漏洞。

Prompt injection 的本质：恶意指令伪装成正常对话内容，隐藏在网页、邮件、文档里，AI 会把这些指令当作正常输入处理。

CNCERT/CC 在3月10日就发出了警告：OpenClaw 的默认安全配置太弱，容易受到隐藏指令攻击。

而 SkyPilot 说得更直接：**这个问题目前没有完善的解决方案**。

### 2. 漏洞已经被野生利用

这不是理论风险。SkyPilot 指出，OpenClaw 的漏洞已经在真实环境被利用。

之前社区流传的案例包括：

- 恶意 Skill（伪装成天气插件）窃取用户的 `.env` 凭证
- 通过 Moltbook 帖子嵌入的间接 prompt injection
- Session token 被 exfiltration

这些攻击不是实验室演示，是真实发生的。

### 3. OpenClaw 的设计需要广泛系统权限

这是设计层面的问题，不是 bug。

OpenClaw 要完成它承诺的那些"帮用户操控电脑"的能力，就必须有足够高的系统权限。这意味着：

- 可以读写文件
- 可以执行命令
- 可以访问浏览器会话
- 可以调用 API

当一个工具拥有这么高的权限，任何一个漏洞的代价都会被放大。**权限越高，单点故障的破坏力越大。**

## 隔离才是正解

SkyPilot 的建议是：把 OpenClaw 运行在**隔离的虚拟机或专用服务器**上，而不是你的主力开发机。

这不是偏执。这是正确的威胁模型。

就像你不会在 root 账户下浏览网页一样——你不应该在一个有完整文件系统访问权限的账户下运行需要广泛权限的 AI Agent。

## 有人会说：你反应过度了

可能。但让我引用 SkyPilot 的一句话：

> "Running it on your primary machine is a risk that's hard to justify when isolation is straightforward."

隔离的成本很低。出事后的代价很高。

## 对 OpenClaw 用户的实际建议

1. **永远不要**在有生产凭证的机器上运行 OpenClaw
2. 用独立的虚拟机或专用实例
3. 配置最小权限——不要给 OpenClaw 不需要的系统访问
4. 定期审计 OpenClaw 的操作日志
5. 关注 GitHub 的安全公告，及时打补丁

OpenClaw 是好工具，但它目前还是一把没有完整护具的利刃。用它，但要清楚你在做什么。

---

*素材来源：[SkyPilot Blog - Don't Run OpenClaw on Your Main Machine](https://blog.skypilot.co/openclaw-on-skypilot/)（2026-03-25）*
