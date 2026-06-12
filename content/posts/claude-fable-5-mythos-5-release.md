+++
date = '2026-06-12T14:00:00+08:00'
draft = false
author = '胡海龙'
title = '屠榜的"神话":Anthropic 发布 Claude Fable 5 与 Mythos 5,以及全网怎么看它'
description = "6月9日,Anthropic 突然把遮了两个月的 Mythos 级模型摆上桌——Fable 5 给所有人,Mythos 5 只给受信合作方。本文整理官方数据、Reddit/知乎/科技媒体三方评价,以及它对你意味着什么。"
tags = ["AI", "AI资讯", "Claude", "Anthropic"]
categories = ["AI资讯"]
series = []
[cover]
image = "https://minio-api.hubianluanma.com/blog/images/2026/06/12/img_1781245432_1.jpeg"
alt = "Fable 与 Mythos 双兽对峙的暗色系封面"
caption = "Fable(寓言)与 Mythos(神话)同源不同命"
relative = false
hidden = false
+++

2026 年 6 月 9 日深夜,Anthropic 突然开了一场"零预热"发布会,把酝酿两个月的 Mythos 级旗舰模型直接摆上桌——**Claude Fable 5** 给所有普通用户,更强的 **Claude Mythos 5** 只给 Project Glasswing 合作方。

这是 Claude 系列第一次有"SOTA 屠榜"的旗舰真正开放给公众用,而不是藏在 API 灰度里。本文整理**官方数据 + 民间评价 + 你要花多少钱 + 有什么风险**四件事。

## 一、先说结论:它到底有多强

官方公告里 Fable 5 直接刷了 15 项基准,**12 项 SOTA**(State of the Art,当前最佳),而且越复杂、越长程的任务,领先幅度越大。官方原话是:

> "The longer and more complex the task, the larger Fable 5's lead."

几个最值得关注的数字(对比 Opus 4.8 / GPT-5.5 / Gemini 3.1 Pro):

| 维度 | 基准 | Fable 5 | 上一代 SOTA | 提升 |
|---|---|---|---|---|
| 智能体编程 | SWE-Bench Pro | **80.3%** | GPT-5.5: 58.6% | +21.7 |
| 长代码任务 | Terminal-Bench 2.1 | **88.0%** | Opus 4.8: 82.7% | +5.3 |
| 知识工作 | GDPval-AA | **1932** | Opus 4.8: 1890 | SOTA |
| 视觉文档 | GDR.pdf (无工具) | **29.8%** | GPT-5.5: 24.9% | +4.9 |
| 空间推理 | Blueprint-Bench 2 | **38.6%** | GPT-5.5: 36.2% | +2.4 |
| 工具使用 | AutomationBench | **17.4%** | Opus 4.8: 15.5% | +1.9 |
| 操作系统 | OSWorld-Verified | 85.0% | Mythos Preview: 85.4% | 略低 |
| 网络安全 | ExploitBench | **78.0%** | Mythos Preview: 69.0% | +9.0 |
| 生物化学 | BioMysteryBench(hard) | **46.1%** | Mythos Preview: 29.6% | +16.5 |
| 综合推理 | Humanity's Last Exam (带工具) | **64.5%** | Mythos Preview: 64.7% | 持平 |

**几个值得注意的点:**

1. **SWE-Bench Pro 直接 +21.7 分**。这是一个非常硬的工程基准(模拟真实开源仓库 issue),GPT-5.5 在这上面是 58.6%,Fable 5 直接拉到 80.3%,**等于把"AI 写代码能不能干工程师的活"这个问题从"不能"切到"大多数时候能"**。
2. **网络安全 ExploitBench 78%**。这是 Anthropic 自己最警惕的领域,所以 Fable 5 在这上面加了"智能安全降级"——触发关键词会回退到 Opus 4.8。
3. **生物化学 +16.5 分**。Anthropic 内部蛋白质设计专家用 Mythos 5 后,**药物设计流程部分环节加速约 10 倍**。
4. **OSWorld-Verified 略输给 Mythos Preview**。这是唯一一个 Fable 5 < Mythos Preview 的公开基准,说明为了安全,Fable 5 在某些任务上做了主动降级。

## 二、Stripe 的"5000 万行代码一天迁移"案例

发布会上最炸的 demo 来自 Stripe 工程师:

> 在一个 **5000 万行 Ruby 代码库**里,Fable 5 **一天之内完成了整个代码库的迁移**。这件事如果靠人工团队做,需要 2 个多月。

这听起来很夸张,但 Stripe 内部已经在用它重构部分基础设施。Anthropic 还演示了 Fable 5 现场写**流体模拟代码 + 同步一段古典音乐 EDM 混音**,整个流程没人介入。

前 OpenAI 联合创始人 **Andrej Karpathy**(刚加入 Anthropic)原话:

> "这是和去年 11 月 Claude 4.5 同等级别的重大跨越,这是他第一次真实感受到'完全不去看代码'这个想法不是玩笑,而是一种真实的诱惑。"

## 三、价格翻倍,免费用 13 天

这是争议最大的地方——**Fable 5 定价是 Opus 4.8 的两倍**:

| 维度 | Fable 5 / Mythos 5 | Opus 4.8 | GPT-5.5 |
|---|---|---|---|
| 输入(每百万 token) | **$10** | $5 | $5 |
| 输出(每百万 token) | **$50** | $25 | $30 |
| 上下文窗口 | 100 万 token | 20 万 | 20 万 |
| 最大输出 | 12.8 万 token | — | — |

**好消息**:6 月 9 日 ~ 6 月 22 日,**Pro / Max / Team / 企业版(按席位)** 全部免费用 Fable 5。
**坏消息**:6 月 23 日开始,这些订阅里的 Fable 5 额度会被移除,要用就**按量消耗订阅额度或直接走 API**。

**Mythos 5 怎么用?**
现在**用不了**。只对 Project Glasswing(透明翼计划)合作方开放,后续会扩展可信访问计划。Anthropic 内部给的解释是:"Mythos 在网络安全和生物化学的杀伤力,我们希望慢慢放。"

**还有一个霸王条款**:Fable 5 / Mythos 5 **禁止用于训练竞品大模型**,但允许用来做应用、调优、Agent 框架。

## 四、Reddit / 知乎 / 推特 三方评价

**最显眼的争议:基准到底公不公平?**

r/singularity 上最高赞评论:

> "I like how they combined Mythos 5 with Fable 5 benchmark so you don't really see what the actual score of Fable is. Also how they excluded DeepSWE but included the recent FrontierCode, meaning GPT 5.5 probably beat it."

直白翻译:**Anthropic 把 Mythos 5 和 Fable 5 的成绩合并报告,你根本看不到 Fable 5 的真实分数**。而且**他们故意不测 DeepSWE**(GPT-5.5 的新基准),只挑了 FrontierCode——所以 GPT-5.5 实际很可能比 Fable 5 强。

**体感派**(r/ClaudeAI、r/slatestarcodex):
- 有人说在 Cursor 里用 Fable 5 写一个 1.5 万行的无尽赛车游戏,**一次跑通**。
- r/slatestarcodex 上一个用户说:"这是自 GPT-3 以来**第一次让我起鸡皮疙瘩的能力跃迁**"。
- 也有 r/ClaudeCode 用户反驳:"20 小时体验下来,体感比 Opus 4.8 强,但**没有 +10 分的差距**,价格倒是实打实贵了 1 倍"。

**反方观点**(r/ClaudeCode 不受欢迎意见帖):
> "If you have managed to try the new model it sort feels okay I mean the numbers (which really don't translate to the output) they showed on benchmarks…"

翻译:这些基准数字**根本不能反映实际输出质量**,跑分高不代表好用。

**知乎"平凡"的回答**(3k+ 赞):
> "如果只按纯粹的聊天体验去测试它,Fable 5 甚至可能让你觉得**没那么惊艳**。但一旦你把它放进 Agent 框架里,**它可以自主运行数天**。这是其他模型做不到的。"

翻译:**聊天场景 Fable 5 体感提升有限,但 Agent 长程任务才是它真正的杀手锏**。

**Karpathy 的"杰文斯悖论"判断**(引用最广):
> "软件需求不会因为 AI 写代码变快而减少,反而会增加(Jevons paradox)。我们以前因为写不动所以不敢想,现在敢想了。"

## 五、它对你意味着什么

**如果你是独立开发者 / 创业团队**:
- **6 月 22 日前**赶紧把 Fable 5 拉进 Claude Code / Cursor / Cline,白嫖窗口期别浪费。
- 重点测**长程 Agent 任务**(代码迁移、批量重构、自动化测试生成),这是它真正领先的地方。
- 聊天/单次生成场景可能体感不强,**不要按 Opus 4.8 时代的预期来用**。

**如果你是企业 / 中后台**:
- 价格是 Opus 4.8 的 2 倍,**预算要重新算**。一次 SWE-Bench Pro 80% 的代码任务跑下来,token 消耗会比 Opus 多,但**完成率高 21.7%,所以总成本可能反而更低**——具体看你的任务类型。
- 6 月 22 日后**订阅用户需要消耗额度**,重度用户建议直接走 API。

**如果你是求职者 / 在校生**:
- 关注"**AI + 长程 Agent**"方向的岗位,这是 2026 下半年最热的细分。
- 提示词工程师的薪资在 2026 年开始出现**两极分化**——只会写模板的会被淘汰,懂 Agent 编排、能调工具链的反而更值钱。
- 短期(1-2 年)内 **Mythos 5 不可能对你开放**,先把 Fable 5 用熟。

**如果你是研究者 / 创业者**:
- **网络安全和生物化学的护城河会越来越厚**。Mythos 5 不开放,这部分研究只能用 Fable 5 + 关键词降级,做不了前沿实验。
- "**用 Fable 5 训练竞品**"是被禁止的,但 Agent 框架 / RAG / 工具调用是开放的。

## 六、它最让人不安的地方

Anthropic 在发布前一周刚发过一篇警告:

> "AI is getting too dangerous."

一周后立刻把 Mythos 级模型开放给公众。这种**"先警告再发布"**的策略,Reddit 上有用户评价为:

> "Anthropic is doing the most aggressive safety theater of any major AI lab."

翻译:这是迄今为止**最激进的"安全表演"**。先把 AI 危险论调子拉满,再选择性开放,既拿到监管好感,又拿到市场份额。

我个人看法:**Mythos 5 的存在本身就是 Anthropic 的合规保险**。万一未来 Mythos 5 的能力被滥用,Anthropic 可以在法庭上说:"我们只给受信方用,没有公开 Mythos 5。" 而 Fable 5 加了"智能安全降级",出问题 Anthropic 也只是回退到 Opus 4.8,不会真正承担危险。

这是聪明的策略,但不一定是安全的策略。

## 七、最后

Fable 5 / Mythos 5 是 2026 年目前为止**最重要的 AI 发布**。它不是"比上一代快一点"的那种迭代,而是**第一次把"AI 自主干几天活"这个科幻场景变成了可用的现实**。

接下来 13 天免费期,如果你还没用过,**强烈建议至少跑一次**真正的长程 Agent 任务(代码迁移、自动化测试、数据分析管道),而不是聊天试一下就完事。

---

**参考来源**:
- [Anthropic 官方公告](https://www.anthropic.com/news/claude-fable-5-mythos-5)
- [TechCrunch 报道](https://techcrunch.com/2026/06/09/anthropic-released-claude-fable-5-its-most-powerful-model-publicly-days-after-warning-ai-is-getting-too-dangerous/)
- [Artificial Analysis 评测](https://artificialanalysis.ai/articles/claude-fable-5-mythos)
- [r/ClaudeAI 体验帖](https://www.reddit.com/r/ClaudeAI/comments/1u1c8eg/claude_fable_5_a_mythos_model/)
- [r/ClaudeCode 反方观点](https://www.reddit.com/r/ClaudeCode/comments/1u1g0l3/unpopular_opinion_claude_fable_5_feels_like_a/)
- [r/singularity 基准争议](https://www.reddit.com/r/singularity/comments/1u1b9hx/claude_mythosfable_5_benchmarks/)
- [r/slatestarcodex 长评](https://www.reddit.com/r/slatestarcodex/comments/1u1cl0w/claude_fable_5_and_claude_mythos_5/)
- [知乎"平凡"的回答](https://www.zhihu.com/question/2047851398734394746/answer/2047897957450753359)
- [知乎 5000 万行代码案例](https://zhuanlan.zhihu.com/p/2047932995126956032)
- [新浪科技价格分析](https://finance.sina.com.cn/tech/roll/2026-06-10/doc-iniawpmz4604154.shtml)
- [异次元软件 综合报道](https://www.iplaysoft.com/p/claude-fable5)
- [掘金 双模型架构解析](https://juejin.cn/post/7649741594028441609)
- [Anthropic 官方定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)
