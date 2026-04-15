+++
date = '2026-04-15T12:00:00+08:00'
draft = false
title = 'Hermes Agent：一个会自我进化的 AI Agent'
description = "Nous Research 开源的自进化 AI Agent，内置学习循环让 Agent 从经验中生成可复用技能、跨会话记忆、 MCP 扩展。本文全面介绍其核心理念、架构和使用方法。"
tags = ["AI Agent", "开源", "Nous Research", "Hermes", "自进化"]
categories = ["AI"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/04/15/hermes-agent-cover.jpeg", alt = "Hermes Agent 自进化 AI 封面图" }
+++

2026 年 2 月，Nous Research 发布了一款开源 AI Agent。项目名叫 **Hermes Agent**——一个"会随着你一起成长的 Agent"。发布两个月后，GitHub 标星突破 6 万，成为开源 Agent 领域最受关注的新项目之一。

它没有试图在单次对话中解决一切，而是把重心放在了**跨会话的学习能力**上。每完成一次任务，它会从中提取可复用的工作流；每次对话，它都会记住你的偏好和项目背景。这种"边用边学"的模式，让它和传统 Agent 有了本质区别。

## 它解决的是什么问题

大多数 AI Agent 本质上是一个"加强版聊天框"——每次对话都是独立的，上下文靠手动粘贴，重复的工作每次都要重新描述。OpenClaw 解决了工具调用的问题，但仍然没有解决**记忆和经验积累**的问题。

Hermes Agent 的回答是：**让 Agent 自己记住它学到了什么。**

你不是在用工具调用增强一个聊天机器人。你是在部署一个会持续进化的个人助理——它认识你的项目，了解你的工作方式，随着使用时间变长而越来越懂你。

## 核心理念：内置学习循环

Hermes Agent 官方文档用一句话概括了它的定位：

> A built-in learning loop that creates skills from experience, improves them during use, and remembers across sessions.

这三个能力构成了它的核心。

### 从经验中生成 Skills

每次你成功完成一个任务，Hermes Agent 会自动把这次经验打包成一个 **Skill**——一个可复用的技能文件。Skill 文件存放在 `~/.hermes/skills/` 目录下，格式是标准 Markdown，包含了触发条件、使用方法和工作流描述。

下次遇到类似任务时，Agent 会自动检索相关 Skill 并加载，而不是重新摸索一遍。这相当于给 Agent 配备了一个持续扩充的"经验库"，且这些 Skill 还可以分享给其他人。

### 跨会话记忆

Hermes Agent 使用 **FTS5 全文搜索** + **LLM 摘要压缩**实现跨会话记忆。它不依赖向量数据库的语义相似性搜索，而是用 SQLite FTS5 做关键词召回，再用 LLM 压缩提炼记忆内容。配置好的情况下，你问"上次我们讨论的那个关于 OpenClaw 安全的问题"，Agent 能直接调出那次对话的结论。

记忆系统还支持**定期提醒**——Agent 会在合适的时机主动把你之前说过但没完成的事情重新提起。

### 模型无关，任意 LLM 驱动

另一个核心设计：**不绑定任何一家模型提供商。** 你可以用：

- Nous 自家的 Hermes 模型（通过 Nous Portal）
- OpenRouter 上的 200+ 模型（Claude、GPT、Gemini、Mistral……）
- 小米 MiMo、Zhipu GLM、Kimi/Moonshot、MiniMax
- Hugging Face 上的开源模型
- 完全自托管的 API 端点

切换模型只需要改一行配置，不需要改动任何业务逻辑。这种灵活性在当下 LLM 市场格局快速变化的背景下尤为重要。

## 架构一览

Hermes Agent 的架构可以分为几层：

**Agent 核心层**——处理对话循环、工具调用、记忆管理和技能编排。核心推理逻辑完全与模型解耦。

**工具与工具集层**——内置文件系统、终端、Web 搜索、代码执行、浏览器自动化等工具。通过 **MCP（Model Context Protocol）** 可以扩展到任意工具或 API。

**消息网关层**——一个进程同时支持 Telegram、Discord、Slack、WhatsApp、Signal、邮件等消息平台。你可以在微信上丢给 Agent 一个任务，到公司后打开终端继续处理同一段对话。

**技能与记忆层**——Skill 文件系统 + FTS5 记忆召回，构成了 Agent 的长期经验积累层。

**扩展机制**——支持插件（`~/.hermes/plugins/` 目录下的 Python 文件），可以在不修改源码的情况下扩展工具、命令和钩子。

## 安装和使用

安装方式极为简洁，一条命令搞定：

```bash
curl -fsSL https://astral.sh/uv/install.sh | sh
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

如果你想手动安装或开发调试：

```bash
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent
uv venv venv --python 3.11
source venv/bin/activate
uv pip install -e ".[all,dev]"
```

安装完成后，运行 `hermes` 即可启动交互式 TUI。首次启动会自动引导你完成 API 配置、消息平台连接等步骤。

配置通过 `~/.hermes/config.yaml` 管理，支持多 Profile——你可以给不同的使用场景配置不同的模型和工具集。

## 对比 OpenClaw

这是很多人关心的问题。OpenClaw 和 Hermes Agent 解决的是不同层次的问题。

OpenClaw 核心解决的是**工具调用和 Agent 框架**的问题——如何让 Agent 调用工具、如何管理多步骤任务流程。它的优势在于工具生态丰富、安全沙箱做得扎实、社区活跃。

Hermes Agent 解决的是**跨会话智能积累**的问题——它默认把记忆和技能生成作为一等公民，工具调用反而不是它的核心差异化点。

两者可以互补。OpenClaw 提供了更丰富的工具生态，Hermes Agent 则在工具之上加了一层学习和记忆。实际使用中，有人把 OpenClaw 的 MCP 工具接入 Hermes Agent，也有人把 Hermes Agent 作为个人助理，用 OpenClaw 处理需要复杂工具链的任务。

## 快速验证

如果你想最快体验 Hermes Agent 的"自进化"特性，可以这样做：

1. 在 TUI 中完成一个多步骤任务（比如"帮我分析一下这个目录下所有 Python 文件的复杂度"）
2. 查看 `~/.hermes/skills/` 目录——你会发现多了一个 `.md` 文件
3. 下次提出类似需求时，观察 Agent 是否主动提到了之前生成的 Skill

这个"从经验中抽取可复用流程"的过程，是 Hermes Agent 最独特的体验。

## 写在最后

AI Agent 领域在 2026 年变得异常热闹。OpenClaw 带火了 Agent 框架，Manus 展示了通用 Agent 的潜力，而 Hermes Agent 走了一条更细分的路——**让 Agent 真正学会"记住"**。

一次性的任务执行是简单的，难的是让 Agent 在多次交互中建立对你、对你的项目、对你的工作方式的持续理解。这条路目前只有 Hermes Agent 走得最深。

开源地址：[github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
官方文档：[hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com)
