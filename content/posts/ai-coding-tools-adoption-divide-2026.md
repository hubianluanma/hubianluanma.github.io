+++
date = '2026-09-12T07:32:23+08:00'
draft = false
title = 'Claude Code 满意度 91%，为什么全球只有 18% 的开发者在用'
description = "JetBrains 15000 人调研揭示 AI 编程工具的采用分裂：终端原生工具满意度碾压一切，但全球 adoption 却只有 18%。这不是产品问题，是工作文化问题。"
tags = ["编程", "技术", "AI"]
categories = ["编程技术"]
author = "Spiral"
+++

Claude Code 的数字看起来像假的：91% 客户满意度，NPS 54，80% 的用户在用过之后把它设为主要工具。放到任何 SaaS 产品里，这都是封神的数据。但同一个调研里，它的全球 adoption 是 18%——美国 47%，全球 18%，差了将近 30 个点。

这不是 Claude Code 卖不动。这是全球开发者工作文化的分裂。

## 数字背后的问题：谁在用，用在哪里

JetBrains 在 2026 年 5 到 7 月调研了 15509 名专业开发者，这是第十届年度开发者生态调查，样本覆盖 8 种语言，统计上算是可信的。

核心数字：

- **Claude Code**：2026 年 1 月 adoption 18%，到 5-7 月涨到 39%，翻了不止一倍
- **GitHub Copilot**：从一年前的 29% 跌到 21%，正式让出王座
- **OpenAI Codex**：从 3% 到 16%，半年 5 倍增长
- **Cursor**：排名第三，和 Claude Code 并列 18%，但增速已经放缓

真正反直觉的数字在这里：Claude Code 的 91% CSAT 是所有被测工具里最高的，但它在美国以外的市场只有 18% adoption。美国是 47%，欧洲和亚洲加起来拖了后腿。

这中间差了 29 个点。

## 反共识观点：满意度高不等于产品好，是交互模型在筛选用户

一个工具 CSAT 91% 但 adoption 只有 18%，常规解释是"产品好但贵/难用/推广不够"。这些理由放在 Claude Code 上都不太成立：它有免费 tier，操作逻辑自洽，文档完整。

真正的原因是它的交互模型在主动筛选用户。

Claude Code 是终端优先的。打开它，你需要在一个没有 IDE 视觉辅助的环境里和 AI 协作。你给指令，AI 执行，你审批结果。这要求开发者已经习惯终端工作流，能够用自然语言描述复杂的多步任务，并且愿意把"执行权"交给一个 autonomous agent。

这类开发者在全球范围内是少数，但在 CLI 文化最强的美国是主流。所以满意度高，是因为用进来的人都对上了它的交互模型；adoption 低，是因为这个模型天然排斥了另一类开发者。

这不是产品缺陷。这是产品哲学。

## Copilot 输掉的不是功能，是定位

Copilot 从 29% 跌到 21%，而它的 awareness 仍然是所有工具里最高的——79% 全球知晓率，美国/欧洲甚至到 86-90%。

知道它，但不用了。这是更危险的信号。

Copilot 最初的定位是"IDE 里的 autocomplete"，它解决的是"帮我补全这行代码"。这个定位在 2021 年是创新的，在 2026 年的 autonomous agent 时代就显得过时了。它的 agent 能力是后期加上去的，用的是同一套 IDE 交互框架，而那套框架的逻辑是"人在驾驶，AI 提供建议"。

Claude Code 的逻辑是反过来的："AI 在驾驶，人在审批"。这两套逻辑在简单任务上差别不大，在复杂任务上体验是断层的。

Copilot 的问题不是做得差，是它最初定义的那个类别正在萎缩。inline suggestion 这个赛道，Claude Code、Codex、Cursor 都在做，Copilot 并没有明显优势。而 autonomous agent 这个新赛道，它入场晚了。

## Cursor 的困境：最好的 IDE，却不是最好的 agent

Cursor 是被讨论最多的工具，也是很多人眼中的"AI 编程未来"。它的 IDE 集成做得最好，Composer 模式解决了多文件编辑问题，Tab 补全速度领先。

但它卡在中间。

Cursor 本质上还是一套增强的 IDE 交互框架：你在写代码，AI 给你建议，你接受或拒绝。它的 agent 能力（Composer/Agent 模式）是在原有架构上叠加的，这让它的 autonomous 能力受限于 IDE 的交互范式。

相比之下，Claude Code 从第一天就是为 autonomous 设计的，没有 IDE 包袱。

Cursor 的解法是"做最好的 IDE，集成最强的 AI"，Claude Code 的解法是"做最强的 autonomous agent，支持任何工作流"。两个解法都没有错，但后者在复杂任务上的天花板更高，在简单任务上的效率差距在缩小。

这解释了为什么 Cursor adoption 增速在放缓：它已经吃完了 IDE 重度用户这个基本盘，想往 autonomous 方向扩，天然受阻于它的 IDE 血统。

## 接下来看什么

**Watch 1：Codex 的增长能不能持续**。Codex 从 3% 到 16% 只用了 6 个月，而且背靠 OpenAI 的生态和 ChatGPT 的推广渠道。如果它推出桌面客户端，增长可能会继续加速。它是 Claude Code 最直接的竞争对手，而且它的定价对重度用户更友好。

**Watch 2：Claude Code 会不会出 IDE 版本**。91% CSAT 说明产品力已经验证了，但 18% 的全球 adoption 说明终端工作流的渗透率有上限。如果 Anthropic 推出官方 VS Code 插件或桌面 IDE，adoption 可能会迎来第二波增长。

**Watch 3：企业市场的选择**。目前 adoption 数据主要反映个人开发者，企业采购的决策逻辑完全不同：安全性、合规性、团队协作功能、集中管理。Cursor 已经在推团队版，Anthropic 也在推企业方案。企业市场可能是下一个变量。

**Watch 4：Copilot 的反击策略**。微软不会坐视 Copilot 下滑。如果它推出真正的 autonomous agent 模式（而不是现在的 autocomplete 加持版），市场格局可能再次改变。19% 的团队定价和深度 VS Code 集成是它的现有优势。

## 结论：满意度是锁， adoption 是钥匙

91% CSAT 和 18% adoption 的差距，本质上是产品哲学的代价。Claude Code 选了一条最难走的路——说服开发者放弃部分控制权——并且把它做到了极致。用进去的人满意度爆表，用不进去的人转头去了 Cursor 或 Copilot。

这不是 Claude Code 的失败，是市场的真实分层。

工作文化的差异不会在一年内消失，这意味着 Claude Code 的 adoption 增长会以美国为圆心向外辐射，速度取决于各地 CLI 文化的渗透程度。对开发者来说，这意味着你的工作方式决定了你该选哪个工具，而不是反过来。

选工具之前，先问自己一个问题：我想让 AI 帮我补全代码，还是帮我完成任务？

答案决定了你在哪个数字里。

## 参考资料

- [JetBrains Developer Ecosystem Survey 2026 — AI Coding Agent Adoption](https://blog.jetbrains.com/research/2026/04/which-ai-coding-tools-do-developers-actually-use-at-work)
- [Heise Online: JetBrains Survey — Claude Code is the most popular coding agent](https://www.heise.de/en/news/JetBrains-Survey-Claude-Code-is-the-most-popular-coding-agent-11419280.html)
- [OTF Kit: AI Coding Tools Shift — Copilot Drops, Codex Soars, Claude Code Shines](https://otf-kit.dev/blog/ai-coding-tools-trends)
- [Dev.to: Claude Code Overtakes GitHub Copilot — JetBrains Survey Analysis](https://dev.to/jamilxt/claude-code-overtakes-github-copilot-what-jetbrains-survey-of-15000-developers-says-about-ai-3nhc)
- [Byteiota: Claude Code Dethrones Copilot — JetBrains 2026 AI Coding Survey](https://byteiota.com/claude-code-dethrones-copilot-jetbrains-2026-ai-coding-survey)
- [Nipralo: AI Coding Tools 2026 — Claude Code, Cursor, Copilot Tested](https://nipralo.com/blogs/best-ai-coding-tools-2026)
