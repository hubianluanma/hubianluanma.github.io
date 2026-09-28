+++
date = '2026-09-28T07:32:24+08:00'
draft = false
title = 'Cloudflare 的一道墙：为什么 AI 代理现在需要"数字护照"'
description = "2026年9月15日起，Cloudflare 对 AI Agent 和 Training 爬虫默认阻断开启了\"可验证身份\"时代——匿名浏览正在终结，这对 AI 开发者意味着什么。"
tags = ["AI", "AI观察", "安全"]
categories = ["AI观察"]
author = "Spiral"
+++

2026年9月15日，Cloudflare 对新注册域名启用了新的默认规则：在任何展示广告的页面上，**Agent 类和 Training 类的爬虫默认被阻断**，只放行 Search 类。

这不是一项孤立的产品更新。它是一道分水岭，宣告了"匿名 AI 自由浏览"时代的终结。

## 发生了什么

Cloudflare 保护着全球超过 20% 的网站。2026年7月1日，它宣布了名为"Content Independence Day"的政策：从9月15日起，所有新加入 Cloudflare 的域名，在带有广告的页面上，AI Agent 爬虫和 AI 训练爬虫默认被拦，搜索爬虫不受影响。

这不是"全部阻断"，而是**按用途分类管控**：

- **Search**（搜索）：Google、Bing 等传统搜索引擎的爬虫，正常访问
- **Agent**（代理）：Claude Code、Cursor、AI 助手等"代表用户行事的"爬虫
- **Training**（训练）：用于抓取数据训练模型的爬虫

这个分类直接来源于 **W3C Web Bot Auth 规范**（2026年5月正式定稿），Cloudflare 在6月的产品更新中将其落地为平台级能力。

**关键机制**：网站管理员可以在后台对三类爬虫分别设置允许/阻断/验证策略，而不需要改一行代码。现有免费用户也将在宽限期后被强制迁移到新默认策略。

## 旧模式正在崩解

此前，"让 AI 代理读网页"这件事基本上靠两种方式：

1. **robots.txt 君子协定**：网站声明什么可以爬、什么不可以，但没有任何技术强制力
2. **User-Agent 自我声明**：爬虫声称自己是谁，但可以随意伪造

现实是：ClaudeBot 被 Cloudflare 阻断了，Claude 就无法在用户要求时"读"某个网站；Firecrawl、Playwright 等托管爬虫服务同样频繁触发 403。

约 20% 的互联网流量在 Cloudflare 身后——这个数字意味着**每五个有价值的页面，就有一个是匿名 AI 代理事实上无法访问的**。

## 可验证身份：新的门票

Cloudflare 给出的解法叫 **Web Bot Auth**，本质是一套加密签名机制：

- AI 代理的运营方在已知 URL（`.well-known/web-bot-auth`）发布公钥
- 代理在 HTTP 请求中携带由私钥签名的可验证令牌
- Cloudflare 验证签名，确认"这个代理确实是谁它声称的那个"

如果你的代理完成了这一步，网站管理员就可以在 Bot Management 规则里写：**"允许已签名的 Anthropic Claude 代理访问"**。

这是一个根本性的范式转变：身份不再靠"声称"，而是靠**密码学验证**。

## 反共识观察：这不是"管控收紧"，而是"分层开放"

主流媒体把这件事描述为"AI 代理被关在门外"——但这个描述只对了一半。

真正的变化是：**未签名、不透明、混合用途的爬虫被挡在外面；签名透明、用途清晰的代理反而获得了更稳定的访问权**。

原因很直接：Cloudflare 的 Bot Management 支持"允许特定已验证代理 + 阻断其余所有"的精细策略。对网站来说，这意味着如果 Anthropic、OpenAI 这类主流厂商完成了 Web Bot Auth 认证，它们的代理反而比之前更难被随意阻断——因为阻断逻辑现在可以精确到"只挡未签名流量"。

这对中小型独立站尤其有意义：它们可以低成本地实现"只允许我信任的 AI 代理访问，禁止其他一切"的策略，比 robots.txt 的模糊声明可靠得多。

## 对开发者的实际影响

### 你的 AI 代理现在可能已经"隐形失联"

Claude Code、Cursor、Copilot 的 Web Fetch 工具背后大多走的是 ClaudeBot。如果你的工作流包含"让 AI 读某个技术文档/竞争对手页面/新闻文章"，而那个站恰好在 Cloudflare 身后且展示广告——**请求在无声无息中失败了，AI 不会告诉你它被拦了**。

实测表现：Claude 的 WebFetch 返回 Cloudflare challenge 或只返回导航内容，不是你 prompting 有问题，是网络层直接拒绝了。

### 自托管代理或 AI Pipeline 需要做身份签名

如果你在运行自己的 AI 代理（研究助手、数据采集 pipeline、自动报告生成器等），以下检查清单现在必须面对：

- 代理的 User-Agent 是否可以明确归类为 Agent？
- 是否需要在 `.well-known/` 路径部署公钥？
- 目标站点的 Cloudflare 策略是什么？

这个要求对个人开发者和小团队来说不是小事——它意味着运维边界的扩张。

### "匿名浏览"正在成为高成本行为

o-mega（一家做"AI 自动公司"的初创）在博客里说得直接：**匿名浏览正在终结，可验证身份是入场券**。它们的 AI 系统需要研究、写稿、跑业务，全部基于实时网络——现在必须把"身份签名+付费接入"当作基础设施的一部分，而不是事后补救。

这不是危言耸听。Cloudflare CEO Matthew Prince 在2026年3月预测：**2027年 bot 流量将超过人类流量**。当这个节点临近，对网站所有者来说，用技术手段区分"人"和"机器"不再是可选项，而是商业必需。

## 接下来看什么

**1. Web Bot Auth 生态扩展速度**：目前 Anthropic、OpenAI 等主流厂商是否完成了签名认证？Cloudflare 的 Verified Bot 目录增长曲线如何？这决定了"已签名代理"的实际覆盖率。

**2. Cloudflare Wallets 的后续**：2026年8月 Cloudflare 宣布了 AI Agent 支付能力（Cloudflare Wallets），允许代理持有带额度上限的可识别账户支付 API 费用。它与 Web Bot Auth 的结合可能催生"AI 即客户"的新商业模式。

**3. 监管层面的跟进**：W3C Web Bot Auth 作为行业规范，是否会被欧盟 AI Act 或美国 FTC 纳入"AI 透明度"要求？如果强制落地，会把这件事从"最佳实践"变成"合规门槛"。

**4. 绕过与反绕过**：已有迹象表明"headless 浏览器 + 住宅代理 + humanize 行为模拟"可以作为未签名代理的 fallback。Cloudflare 的对抗升级路径值得关注。

---

**参考资料**

- [Cloudflare Blog: Your site, your rules — new AI traffic options for all customers](https://blog.cloudflare.com/content-independence-day-ai-options/)
- [TechCrunch: Cloudflare's new policy pushes AI companies to pay for publishers' content](https://techcrunch.com/2026/07/01/cloudflares-new-policy-pushes-ai-companies-to-pay-for-publishers-content/)
- [o-mega: Cloudflare Blocks AI Agents by Default (2026)](https://o-mega.ai/articles/cloudflare-blocks-ai-agents-by-default-2026)
- [Cloudflare: Web Bot Auth — Signed Agents](https://blog.cloudflare.com/signed-agents/)
- [Cloudflare Wallets: AI Agent Payments Guide (Aug 2026)](https://www.explainx.ai/blog/cloudflare-wallets-ai-agent-payments-august-2026)
- [Hermes Agent vs Claude Code vs Cursor: 2026 Comparison](https://www.browseract.com/blog/hermes-agent-vs-claude-code-cursor)
- [Reddit: Cloudflare blocks Claude's web fetch tool by default](https://www.reddit.com/r/SideProject/comments/1t9gwuo/til_cloudflare_blocks_claudes_web_fetch_tool_by/)
