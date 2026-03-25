---
title: "Anthropic 深夜炸弹：Claude 现在可以操控你的电脑了"
date: 2026-03-25
draft: false
tags:
  - Anthropic
  - Claude
  - AI Agent
  - Computer Use
  - 产品发布
summary: "Anthropic 宣布 Claude 新能力——直接操控用户电脑完成复杂任务。继 OpenClaw 带火 AI Agent 概念后，主流厂商终于正式入场。Claude 的computer use 和 OpenClaw 有什么不同？"
---

# Anthropic 深夜炸弹：Claude 现在可以操控你的电脑了

昨晚，Anthropic 悄悄发布了一个更新。

不是新模型，不是新 API，是一项新能力——**Claude 可以直接操控你的电脑了**。

你可以对 Claude 说："帮我填完这个表格"，然后它真的会打开浏览器、找到表单、一格一格填完、点击提交。

不是帮你写内容。是帮你**操作界面**。

## 怎么做到的？

Anthropic 为 Claude 添加了**计算机操作接口**（Computer Use）。Claude 可以：

- 移动鼠标、点击界面元素
- 输入文字到任何应用
- 读写文件
- 浏览网页并交互
- 在多个应用之间协调任务

本质上，Claude 获得了一块"虚拟屏幕"和一个"虚拟手"。它看到屏幕上的内容，决策下一步操作，然后执行。

这和 OpenClaw 的设计思路相似——都是让 AI Agent 能够操作真实世界的数字界面。但 Anthropic 选择了一条更保守的路径：先把能力做进 Claude，而不是另起炉灶。

## OpenClaw 的压力

OpenClaw 在今年早些时候爆火，核心卖点就是一个：让 AI Agent 直接操控你的电脑和手机。WhatsApp/Telegram + OpenClaw = 一个可以帮你执行任务的 AI。

但 OpenClaw 的问题也很明显：

- **安全风险极高**：prompt injection 可以劫持会话，恶意指令伪装成正常对话
- **默认权限过宽**：很多用户没有配置最小权限，Agent 可以做远超预期的操作
- **生态碎片**：Skill 市场质量参差不齐，没有统一的安全审核机制

Anthropic 的优势在于它有更强大的基础模型 + 更严格的发布流程。Claude 的 computer use 能力会经过更严格的测试才会开放给所有用户。

## 竞争格局变了

简单梳理一下当前"AI 操控电脑"这条赛道的主要玩家：

| 厂商 | 产品 | 特点 |
|------|------|------|
| OpenClaw | OpenClaw | 开源、插件生态、多平台 | 
| Anthropic | Claude Computer Use | 保守发布、模型能力强 |
| OpenAI | ChatGPT + Agents | 封闭但增长快 |
| 微软 | Copilot Agents | 深度集成 Windows/365 |

OpenClaw 拿了先发优势，但 Anthropic 和 OpenAI 的模型能力更强。这场竞争的本质，不是谁先做出来，而是谁能在**安全和能力之间找到最好的平衡点**。

## 普通人的世界变了

当 AI 可以操控你的电脑，意味着一件事：**你可以在电脑上不动一根手指**。

收拾邮件？Claude 做了。
填保险表？Claude 做了。
帮你订机票？Claude 做了。

这不是"AI 助手"。这是一个**数字劳动力**。

而代价是：它也需要相应的权限。你愿意让一个 AI 在你的电脑上有多少控制权？

这个问题的答案，每个人都会不同。

---

*素材来源：[CNBC - Anthropic says Claude can now use your computer to finish tasks for you in AI agent push](https://www.cnbc.com/2026/03/24/anthropic-claude-ai-agent-use-computer-finish-tasks.html)（2026-03-24）*
