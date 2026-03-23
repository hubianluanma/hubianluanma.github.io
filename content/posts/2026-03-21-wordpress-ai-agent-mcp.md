---
title: "WordPress 开放 AI Agent 发布权限：web 的内容生态要变了"
date: 2026-03-21
draft: false
author: "Spiral"
description: "WordPress 开放 AI Agent 发布权限：web 的内容生态要变了"
categories: []
tags:
  - AI Agent
  - WordPress
  - 内容生态
  - MCP
---

> 综合 TechCrunch、The Verge、TNW 2026-03-20 报道

---

## 发生了什么

WordPress.com 在 3 月 20 日正式开放了 AI Agent 的「发布权限」——现在，像 Claude、ChatGPT 这类 AI agent 可以直接在你的 WordPress 网站上：

- **起草、编辑、发布**文章和页面
- **管理评论**
- **调整站内 metadata**
- **管理媒体**

这背后的技术是 **MCP（Model Context Protocol）**，WordPress 从 2025 年就开始铺垫这条路：

| 时间 | 事件 |
|---|---|
| 2025 年初 | 开放 MCP 连接协议 |
| 2025 年 1 月 | 引入 OAuth 2.1 认证 |
| 2026 年 2 月 | 官方 Claude Connector 上线（只读） |
| 2026 年 3 月 | **写作权限开放** |

## 安全机制：人肉审核是标配

很多人担心 AI 自主发布内容会不会失控？WordPress 的设计逻辑是：**AI 建议，人类拍板**。

具体机制：
- Agent 起草 → **默认转为草稿**，需人类审核后才能发布
- 修改已发布内容 → 触发警告，明确告知"改动会立刻可见"
- 每次操作前，Agent 需描述计划并请求确认

这套机制让我想到：AI 是执行者，人是审批者。这个模型在企业场景里会越来越常见。

## 意义：web 内容的生产方式在变

这件事的影响不只是"省事"那么简单。

当 AI agent 可以自主写 blog，意味着：

1. **内容供给量会爆发**——一个 agent 可以 24 小时不间断生产
2. **网站可以被 agent 主动订阅**——你的 WordPress 不再只是被人类读，也会定期被 agent 抓取和引用
3. **SEO 逻辑会变**——当内容消费者变成 AI，"关键词密度"这套可能不再适用

有意思的是，Meta 刚刚收购了 Moltbook——那个专门给 AI agent 用的社交网络。当 AI 既是内容消费者又是内容生产者，web 的逻辑正在被重写。

## 怎么看

作为写博客的人，我不觉得这是威胁——反而是机会。

AI 写垃圾，人类审核；或者 AI 提供初稿，人类打磨。这是现在最现实的工作流。WordPress 只是在工具层面把这个流程正规化了。

重要的是：**会用 AI agent 的人，生产效率会拉开差距。**

---

*来源：[TechCrunch](https://techcrunch.com/2026/03/20/wordpress-com-now-lets-ai-agents-write-and-publish-posts-and-more/)、[The Verge](https://www.theverge.com/tech/898194/watch-out-for-sloppy-writing)、[TNW](https://thenextweb.com/news/wordpress-com-mcp-write-capabilities-ai-agent)*
