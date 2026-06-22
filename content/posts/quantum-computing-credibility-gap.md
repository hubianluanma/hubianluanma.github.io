+++
date = '2026-06-22T08:00:00+08:00'
draft = false
title = '量子计算的「信誉鸿沟」：微软画饼2029，小公司悄然FTQC'
description = "当微软还在用PPT承诺2029年上线时，一家名不见经传的公司已在IBM量子云上悄悄跨过FTQC门槛。物理学家的集体质疑让这场量子竞赛的叙事彻底颠倒。"
tags = ["量子计算", "AI观察", "微软", "技术"]
categories = ["AI观察"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/22/img_1782086551_1.jpeg", alt = "量子芯片在黑暗中发出蓝光，与周围形成强烈对比" }
+++

6月的第二周，量子计算圈同时发生了两个事件，叙事方向却截然相反。

微软在6月2日高调发布 Majorana 2 芯片，宣称比前代"可靠1000倍"，并将实用量子机器的时间表定在2029年。紧接着，物理学家们在网上公开反驳——圣安德鲁斯大学的 Henry Legg 告诉 BBC，微软的量子研究"已firmly moved away from science and entered the realm of faith"。同一周，一家叫 AIX Global Innovations 的小公司在6月15日悄然发布了一份100页技术报告，记录其在4月已通过租用 IBM Quantum 云硬件，实现了量子计算领域的圣杯：FTQC（容错量子计算），150量子比特寄存器，零检测到的逻辑错误。

两条新闻放在一起读，味道很怪。

## 物理学家和微软的公开决裂

这不是微软第一次遭遇公开质疑。2025年2月，微软宣布 Majorana 1 是全球首个拓扑量子比特芯片，物理圈立刻炸锅。

Physics World 记录的 peer review 文件显示，专家审查结论毫不客气：**"manuscript 中的结果不构成在所报告设备中存在 Majorana 零模式的证据"**。Science News 描述了随后在加州 Anaheim 举行的物理学会议——微软这场关于拓扑量子比特的演讲，成了全场最受关注的焦点，也招来了最密集的炮火。

更关键的是：这次质疑不是学术圈私下的嘀咕。Fortune 报道提到，那份质疑微软的 peer review 文件是公开的，任何人都能读到。**当一家万亿市值的科技公司被物理学家用公开文件挂在耻辱柱上，这种事在半导体行业史上极为罕见。**

今年6月2日的 Majorana 2 声明，本质上是微软对去年质疑的一次回应。但物理学家并不买账。Scientific American 引用 Legg 的话：**"If this was from any other group or Ph.D. student, it would never make it through peer review."** 换句话说，微软之所以能跳过 peer review 直接向公众宣布，靠的是它的体量和舆论能力，而不是数据本身的过硬程度。

## 微软的叙事困境：为什么非说不可？

微软为什么在争议未解决的情况下持续推进宣传机器？

答案可能在于**量子计算的「主街效应」**——当一项技术足够重要、又足够遥远时，任何关于它的进展都会被媒体放大，无论数据是否经得起推敲。量子计算领域的投资者、分析师、开发者都在等待一个"量子优势"的可验证案例，这种集体焦虑构成了完美的叙事市场。

微软是一家需要持续向华尔街讲故事的万亿公司。2029年的承诺本身就是一个精心设计的时间节点——足够远，远到现有质疑大概率已经被遗忘或被新消息覆盖；又足够近，近到能影响今天的投资决策和人才招聘。

这种"PPT量子计算"的问题在于：它正在消耗整个领域的信誉。Science News 报道提到，在 Anaheim 会议上，有物理学家私下表示对整个量子计算产业持保留态度——不是因为理论上做不到，而是因为浮夸宣传太多。**当产业自己开始产出反效果时，受损的不只是微软，而是整个生态。**

## AIX：静默者的逆袭

就在舆论聚焦微软的时候，AIX Global Innovations 6月15日的公告几乎没有任何主流科技媒体的报道。

但这份公告的分量可能比微软的 Majorana 2 声明更重。

根据 Business Wire 发布的官方新闻稿和 Quantum Computing Report 的详细解读，AIX 实现了以下突破：

- 在 IBM Quantum 云上使用真实硬件（非模拟），完成端到端 FTQC 堆栈
- 150量子比特编码寄存器，约**一比一的物理/逻辑量子比特比例**，Near-perfect fidelity
- **零检测到的逻辑错误**（zero detected logical errors）
- 同时通过了 distance-3 和 distance-5 表面码量子纠错
- 使用通用 Clifford+T 门组

这是第一次有公开报告称在云计算平台上同时满足 FTQC 的四个核心结构标准。Quantum Zeitgeist 的报道甚至直接用了"Clears FTQC Threshold"这样的标题——这是量子计算领域最严格的技术门槛。

AIX 没有万亿市值，没有全球发布会，也没有物理学家的公开质疑。它的团队选择了另一种存在方式：发100页技术报告，让数据说话。

## 反共识的核心观察：量子计算的「信誉逆转」正在发生

传统叙事里，大公司 = 可信，小公司 = 风险。量子计算领域过去十年基本遵循这个规律：IBM、谷歌、微软、英特尔轮流登头条，每家都在"5年内实现量子优势"的承诺上反复横跳。

但这次的信号不一样：**物理学家开始公开站出来说话，而且是对着大公司说话。** 这种学术诚信机制的启动，对于整个产业而言可能是一件好事——它意味着量子计算正在从"宣传竞赛"向"技术验证"过渡。

真正的问题不是"哪个公司先造出量子计算机"，而是**哪个生态能建立起可验证、可复现的技术标准**。AIX 的方法论值得注意：它租用 IBM Quantum 硬件，在公开云平台上完成验证，这意味着任何有技术能力的团队都可以去复现。开放性才是量子计算走向实用的真正门槛。

微软的问题也不是技术本身——拓扑量子比特的理论路线并未被证伪——而是它选择用公关替代 peer review。当一家公司的技术声明需要靠"万亿市值的光环"来背书而不是靠可复现的数据时，物理学家有权利说"我们还没看到证据"。

## 接下来看什么

量子计算正在进入一个"去魅"阶段，以下几个事件值得持续关注：

1. **微软能否提供可 peer review 的数据**：Majorana 2 的核心材料堆栈已更新（铅替代铝），物理学家明确要求看原始数据。如果微软在接下来6个月内提交正式论文并通过同行评审，争议才会真正平息。

2. **AIX 的后续验证**：100页报告是第一步，但量子计算社区需要其他团队在 IBM Quantum 云上复现结果。如果复现成功，量子计算的"FTQC时代"会比多数人预期更早到来。

3. **IBM 的战略反应**：IBM Quantum 是 AIX 突破的硬件提供方，如果 AIX 的结果可复现，IBM 可能会加大云端量子服务的推广力度，这对整个量子云服务市场是重大利好。

4. **谷歌和英特尔的量子路线图更新**：微软的信誉危机可能倒逼其他大厂更谨慎地发布进展——这对于整个产业的长期健康发展未必是坏事。

---

## 参考资料

- [Microsoft announces Majorana 2 quantum computing chip (Tom's Hardware, 2026/06/02)](https://www.tomshardware.com/tech-industry/quantum-computing/microsoft-announces-majorana-2-quantum-computing-chip-claims-a-practical-machine-will-come-in-2029)
- [Microsoft quantum chip upgrade draws physicist skepticism (Science News, 2026/06)](https://www.sciencenews.org/article/microsoft-quantum-chip-upgrade-majorana)
- [Microsoft's upgraded Majorana quantum computing chip fizzles with physicists (Scientific American, 2026/06)](https://www.scientificamerican.com/article/microsofts-upgraded-majorana-quantum-computing-chip-fizzles-with-physicists/)
- [Microsoft claims new quantum chip 1,000 times better (BBC, 2026/06/02)](https://www.bbc.com/news/articles/cj4p7gyvp52o)
- [Experts weigh in on Microsoft's topological qubit claim (Physics World, 2025)](https://physicsworld.com/a/experts-weigh-in-on-microsofts-topological-qubit-claim/)
- [Microsoft's quantum computing breakthrough questioned (Fortune, 2025/02/20)](https://fortune.com/2025/02/20/microsoft-quantum-computing-breakthrough-questioned-by-experts/)
- [AIX Global Innovations FTQC Breakthrough (Business Wire, 2026/06/15)](https://www.businesswire.com/news/home/20260615360168/en/AIX-Global-Innovations-Announces-FTQC-Breakthrough-Quietly-Achieved-in-April-2026-Accelerating-the-Quantum-Compute-Timeline)
- [AIX Global Innovations Discloses FTQC Campaign on IBM Hardware (Quantum Computing Report, 2026/06)](https://quantumcomputingreport.com/aix-global-innovations-discloses-software-governed-fault-tolerant-quantum-computing-campaign-on-cloud-accessible-ibm-hardware/)
- [AIX Clears FTQC Threshold on 150-Qubit Register (Quantum Zeitgeist, 2026/06)](https://quantumzeitgeist.com/ftqc-150-qubit-register-aix-global/)
