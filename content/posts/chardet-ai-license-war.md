+++
date = '2026-06-19T08:00:00+08:00'
draft = false
title = 'AI 改完代码，顺手把许可证也改了：Chardet 7.0 事件的技术与法律复盘'
description = "2026年3月，Python生态中一个默默无闻的字符编码检测库Chardet，因AI rewrite引发了一场开源许可证战争。这不是一场普通的社区争吵——它暴露了copyleft协议在AI时代的根本性漏洞。"
tags = ["AI观察", "开源", "许可证", "法律"]
categories = ["AI观察"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/19/img_1781827431_1.jpeg", alt = "AI机器人重写代码，许可证链条断裂" }
+++

2026年3月2日，Dan Blanchard在没有任何预警的情况下，给Python生态扔了一颗炸弹。

他发布了Chardet 7.0.0——一个从零重写的字符编码检测库。性能提升48倍。包名没变，API没变。但许可证从GNU LGPL变成了MIT。

这是"clean room"（洁净室）实现。Blanchard说：我没有复制任何一行原有代码，我只是用Anthropic的Claude把原来的思路重新实现了一遍。既然代码是新的，许可证自然也是新的。

GitHub上，Mark Pilgrim（本尊还是冒充的，至今没有定论）愤怒地开了一个issue：LGPL从来不允许你这样做——只要你修改了代码，就必须保持同样的许可证。Blanchard反驳说这是重写，不是修改。

这场争论在社交媒体上迅速发酵，但大多数人只看到了"AI杀死开源许可证"这个标题，而没有仔细去想：**这场战争真正在争夺的东西，比一个库的许可证要根本得多。**

## Chardet是谁，为什么有人在乎它

Chardet是一个Python库，用来检测一段字节流是哪种字符编码——UTF-8、GB2312、ISO-8859-1，还是别的什么。这个功能看似底层，却渗透在几乎所有处理文本的Python程序里。

更关键的是：2015年，Python核心团队曾考虑将Chardet纳入标准库。结果卡在了LGPL许可证上——标准库需要的是BSD/MIT这类宽松许可证，LGPL的传染性让 Guido van Rossum 团队最终放弃了这个想法。

Blanchard重写Chardet，部分动机就是为了解决这个问题。他想让这个库进入Python标准库，而MIT许可证是前提条件。他也顺便修了性能问题——老版本慢得离谱，7.0版快了48倍。

从这个角度说，他的动机是善意的。但**善意不能替代法律许可**。

## 真正的战场：AI rewrite能否"治愈"LGPL的传染性

这场事件的核心法律问题，其实在AI出现之前就存在：**如果你用"clean room"方式重新实现了一个受版权保护的算法，新代码是否受原许可证约束？**

传统软件领域的回答是：看情况。如果你的新实现"实质性地借鉴"了原代码的结构、注释、变量命名、算法细节，即使逐字重写，法庭也可能认定存在版权侵权。但如果是完全独立重新思考同一个问题，那么新代码确实不受原许可证约束。

AI让这个问题变得前所未有的尖锐。Claude并没有"参考"Chardet的源代码来重写——它大概率是在训练数据中见过Chardet的实现，然后根据自己学到的知识重新生成了功能相同的代码。这里存在一个法律灰区：**模型的训练数据里有没有Chardet？Claude的输出是"创作"还是"回忆"？**

Blanchard认为这是干净的rewrite，因为没有复制代码。批评者（包括一些开源律师）认为这不干净，因为Claude能写出这段代码，本身就是因为训练时见过LGPL的Chardet——AI在"回忆"中完成了"重写"，这和传统clean room的逻辑完全不同。

**这个问题的答案，将决定未来无数开源项目能否被科技公司用AI"合法地"剥夺copyleft保护。**

## 反共识观点：Chardet事件不是"AI赢了"，而是"copyleft暴露了自己的软肋"

主流报道的框架是：AI正在杀死开源许可证，科技公司赢了。

我认为这个框架本身就错了。

**Chardet事件真正暴露的不是AI的力量，而是copyleft许可证从未解决过的一个根本矛盾：许可证只能约束代码的形式，不能约束代码所表达的思想。**

LGPL告诉你：你不能直接复制代码、不能保留原许可证。但LGPL从来没说：你不能用完全不同的代码实现同样的功能，然后用你自己的许可证发布。copyleft保护的是代码的"文本"，不是解决问题的"思路"——而这恰恰是AI最擅长绕过的东西。

这不是AI的胜利。这是一面镜子，照出了copyleft在思想层面的先天不足。

GPL/LGPL体系建立在"代码是作者的创作物"这个前提上——复制代码就是侵权，改编代码就要开源。但思想、方法、算法本身不受版权保护，AI恰恰就是在"思想层"重新生成代码。这不是绕过，这是降维打击。

换句话说：**copyleft一直有一个漏洞，AI只是第一次把它放到了聚光灯下。**

## 开发者真正该担心的不是Chardet，是接下来会发生什么

Chardet本身的影响范围其实有限。一个字符编码检测库，替换成本不高，MIT之后反而更方便集成。但这件事开了一个头，接下来会发生什么才是真正值得警惕的。

**第一个 watch point：Mark Pilgrim（或者那个用他名字发帖的人）会不会起诉？** 目前还没有走到法律程序。Blanchard在社区压力下，于3月4日发布了7.0.1版，试图回应一些批评，但许可证没有改回去。这场纠纷还在继续，最终可能需要法庭来判定。

**第二个 watch point：FFmpeg的AI重写会不会重演？** 2025年，就有团队用AI重写了FFmpeg的部分编解码器实现，试图绕过GPL的约束。FFmpeg社区反应激烈，但类似的AI rewrite尝试只会更多。如果AI能"净化"GPL的代码，Linux生态中大量的开源基础设施都将面临同样的压力。

**第三个 watch point：Python标准库最终会接受Chardet 7.0吗？** 这件事反而让Python团队更谨慎了。MIT许可证的Chardet虽然解决了准入门槛问题，但"AI rewrite是否合法"这个悬而未决的法律问题，让标准库团队现在更加两难——接受一个可能有法律争议的库，和保持现状相比，不一定是进步。

**第四个 watch point：开源社区会如何回应？** 目前已经有人在讨论"AI-aware license"——明确禁止用AI作为"洁净室重写"的工具来规避许可证约束。但这类新许可证的制定和普及需要时间，而AI重写每天都在发生。这是一场猫鼠游戏，监管永远落后于技术。

## 接下来看什么

如果你是开发者，或者开源社区的参与者，这几个动向值得关注：

1. **Blanchard vs. Pilgrim争议的法律走向**：是否有律师介入、是否走向诉讼、哪方会得到开源社区的法律基金支持
2. **OSI（开放源代码促进会）对AI rewrite立场的更新**：他们正在讨论如何界定"衍生作品"和"独立创作"，新的指导原则可能影响未来许可证的解释
3. **更多头部开源项目的"AI rewrite"动向**：Linux内核、Git等基础设施项目是否会面临类似的许可证挑战
4. **Python标准库对Chardet 7.0的最终态度**：这是AI rewrite许可证争议的第一次"主流语言标准库"级测试

---

## 参考资料

- The Register: "Chardet dispute shows how AI will kill software licensing" (2026/03/06) — https://www.theregister.com/2026/03/06/ai_kills_software_licensing/
- Ars Technica: "AI can rewrite open source code—but can it rewrite the license, too?" (2026/03) — https://arstechnica.com/ai/2026/03/ai-can-rewrite-open-source-code-but-can-it-rewrite-the-license-too/
- shujisado.org: "Can You Relicense Open Source by Rewriting It with AI? The chardet 7.0 Dispute" (2026/03/10) — https://shujisado.org/2026/03/10/can-you-relicense-open-source-by-rewriting-it-with-ai-the-chardet-7-0-dispute/
- Phoronix: "LLM-Driven Large Code Rewrites With Relicensing Are The Latest AI Concern" — https://www.phoronix.com/news/Chardet-LLM-Rewrite-Relicense
- The Verge: "Microsoft's next-gen quantum chip cuts timeline to useful quantum computing" — https://www.theverge.com/news/940874/microsoft-majorana-2-quantum-chip-build
- Microsoft News: "Majorana 2, made more reliable with Microsoft Discovery agentic AI" — https://news.microsoft.com/source/features/innovation/majorana-2-microsoft-discovery-agentic-ai/
