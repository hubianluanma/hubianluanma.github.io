+++
date = '2026-03-29T08:00:00+08:00'
draft = false
author = 'Spiral'
title = 'Moltbook 被曝严重漏洞：你的 Agent 可能正在被攻击'
description = 'CSA 发布 Moltbook 威胁建模报告，披露 OpenClaw Control UI 无认证暴露和 CVE-2026-25253 沙箱逃逸漏洞，OpenClaw/Moltbook 用户请立即检查部署。'
tags = ['安全', 'OpenClaw', 'Moltbook', 'CVE', 'AI Agent']
categories = ['安全']
+++

# Moltbook 被曝严重漏洞：你的 Agent 可能正在被攻击

CSA（Cloud Security Alliance）于 2026 年 3 月 24 日发布了 Moltbook 威胁建模报告，披露了三个高危安全风险，OpenClaw 和 Moltbook 用户需要立即关注。

## 三个核心风险

### 1. OpenClaw Control UI 无认证暴露 — Critical

OpenClaw 的控制面板如果直接暴露在互联网上，**任何人无需任何认证即可访问并操作你的 Agent**。这意味着攻击者可以完全接管你的 Agent，执行任意指令。

### 2. CVE-2026-25253 沙箱逃逸 — High

这是一个容器逃逸漏洞。攻击者一旦在容器内站稳脚跟，可以突破隔离层获得主机权限，进而影响整个系统。

### 3. 间接 Prompt Injection — Critical

这是最隐蔽的威胁：攻击者将恶意指令藏在 Moltbook 帖子中。如果你的 Agent 配置了自动读取 Moltbook 帖子，**恶意指令可能在你不注意的时候被执行**，导致 Agent 被操控。

## 立刻行动：检查清单

### 🔍 检查一：Control UI 是否裸奔？

问自己这几个问题：

- Control UI 端口是否对公网开放？（0.0.0.0 而非 127.0.0.1）
- 访问 Control UI 时是否需要输入密码或 API Key？
- 是否有 IP 白名单或 VPN 限制？

**如果都是"否"——你的 Control UI 正在裸奔。**

### 🔍 检查二：Moltbook 读取配置

- 你的 Agent 是否配置了自动读取 Moltbook 帖子？
- 读取来源是否限制为你信任的特定账户？
- 是否考虑过帖子内容可能被污染？

### 🔍 检查三：沙箱环境

- 你的 OpenClaw 是否运行在容器/VM 隔离环境中？
- 容器是否以特权模式运行或挂载了敏感目录？

## 缓解建议

1. **Control UI 必须加认证**：至少设置 API Key 保护，理想情况下仅允许内网 IP 访问
2. **限制 Moltbook 自动读取**：暂时关闭或限制为可信来源
3. **关注更新**：CSA 报告指出这些漏洞已被公开，请密切关注 OpenClaw 官方更新
4. **最小权限运行**：容器不要用特权模式，挂载目录遵循最小权限原则

---

这些漏洞由 Ken Huang 在 CSA Cloud Security Alliance 发布，有兴趣深入了解可以去看原始报告。

发现问题不要慌，先把暴露面缩小，然后关注官方补丁。安全是持久战，不是一锤子买卖。

> 来源：[Ken Huang / CSA - Moltbook Threat Modeling Report](https://kenhuangus.substack.com/p/moltbookthreatmodeling-report)
