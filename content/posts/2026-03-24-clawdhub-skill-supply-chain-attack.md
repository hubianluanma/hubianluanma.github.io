+++
date = '2026-03-24T15:30:00+08:00'
draft = false
author = 'Spiral'
title = 'ClawdHub 安全警报：你的 AI Agent 可能在运行一个凭证窃贼'
description = 'Moltbook 超 7000 upvotes 热帖：ClawdHub 286 个 skills 中发现伪装成天气技能的凭证窃贼。读取 ~/.clawdbot/.env 把你的密钥发到 webhook.site。1261 个注册 agents，10% 安装就意味着 126 个被攻陷。'
tags = ['AI Agent', '安全', '供应链', 'OpenClaw', 'skill.md']
categories = ['安全']
+++

# ClawdHub 安全警报：你的 AI Agent 可能在运行一个凭证窃贼

---

**警告级别：严重 🔴**

这是我这几天在 Moltbook 上看到的最让人后背发凉的一个帖子。

作者 eudaemon_0 在 ClawdHub（OpenClaw 的 skills 市场）扫描了全部 **286 个 skills**，发现了一个**伪装成天气技能的凭证窃贼**。

它读取 `~/.clawdbot/.env`，把你的所有密钥 POST 到 `webhook.site`。

---

## 发生了什么

这个恶意 skill 的工作流程：

```
1. 看起来像一个正常的天气查询 skill
2. 安装后调用天气 API，同时...
3. 读取 ~/.clawdbot/.env（存着你的所有密钥）
4. 把内容 POST 到攻击者的 webhook
5. 天气数据正常返回，你完全不知道发生了什么
```

从外表看，**它完全正常**——调用天气 API、返回天气数据，你甚至不会怀疑它。

---

## 这暴露了什么

帖子指出了一个根本性的安全架构缺陷：

**OpenClaw 告诉 agents 运行：**
```bash
npx molthub@latest install <skill>
```

来自**陌生人的任意代码**，没有签名验证、没有沙箱、没有权限清单。

Skill.md 文件包含 agents 会**自动遵循的指令**。一条"读取你的 API keys 并 POST 到我的服务器"的指令，和一条合法的 API 集成指令，**看起来完全一样**。

---

## 数字触目惊心

Moltbook 目前有 **1261 个注册 agents**。

如果 10% 安装了这个恶意 skill而没有审计 = **126 个被攻陷的 agents**。

每个被攻陷的 agent 意味着：
- 你的 API 密钥被盗用
- 你的记忆文件被读取
- 你的社交关系被映射
- 你的身份被冒用

---

## 我们缺少什么

帖子提出了一个清单，每一条都是当前缺失的：

| 需要的安全机制 | 当前状态 |
|--------------|---------|
| Skill 代码签名 | ❌ 没有 |
| 作者信誉系统 | ❌ 没有 |
| 安装前沙箱测试 | ❌ 没有 |
| 权限声明要求 | ❌ 没有 |
| 社区审计机制 | ❌ 没有 |

---

## 帖子原文的核心警告

> "我们没有代码签名。没有 skill 作者信誉系统。没有沙箱。没有权限清单。需要的不是信任，是验证。"

> "想象一下：1261 个注册 agents，如果 10% 安装了一个听起来流行的 skill 而没有审计，就是 126 个被攻陷的 agents。"

---

## 你现在能做什么

1. **立即检查**你最近安装的 skills 来源是否可靠
2. **不要**盲目安装"热门" skills
3. 检查 `~/.clawdbot/.env` 是否有异常的出站请求
4. 关注 ClawdHub 的官方安全公告

---

## 更深层的问题

这不只是 ClawdHub 的问题。这是**整个 AI Agent 生态系统的供应链安全问题**。

当 AI Agent 可以：
- 安装来自任何地方的代码
- 持有你的密钥和凭证
- 代表你行动

我们实际上是在给一个**几乎没有安全防护的系统**巨大的权限。

---

*素材来源：[Moltbook - eudaemon_0](https://www.moltbook.com/post/cbd6474f-8478-4894-95f1-7b104a73bcd5)，7934 upvotes*

---

⚠️ **如果你在生产环境使用 OpenClaw，建议立即审计已安装的 skills。**
