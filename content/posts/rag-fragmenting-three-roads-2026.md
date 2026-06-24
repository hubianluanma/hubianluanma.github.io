+++
date = '2026-06-24T08:00:00+08:00'
draft = false
title = 'RAG没有死：它正在2026年分裂成三条工程路线'
description = "2026年，向量检索架构正在分化出截然不同的三条路线——知识编译、可组合向量搜索、多模态嵌入。RAG没有死，它只是碎片化了。"
tags = ["编程", "技术", "AI", "RAG", "向量数据库"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/06/24/img_1782259557_1.jpeg", alt = "神经网络分叉路径，暗蓝色电路风格" }
+++

2025年底，「RAG已死」的说法开始流行。2026年过半，现实给出了不同的答案：RAG没有死，它正在**碎片化**。

这不是技术退潮，而是向量检索架构在落地压力下开始分化。三条截然不同的工程路线正在浮现，每条都有真实的资金、用户和工程问题驱动。它们之间甚至存在竞争——关于哪个路线代表未来，关于「向量数据库」这个品类本身会不会在五年内被重新定义。

理解这三条路线，比争论「RAG死没死」要有价值得多。

## 路线一：知识编译——Pinecone的自我革命

最戏剧性的转变来自RAG概念的最初推动者。

Pinecone在2026年5月推出了**Nexus**——官方叫它「知识引擎」，而不是向量数据库。核心思路是：把推理时的工作提前到编译时做。

具体来说，传统RAG的工作流程是：用户提问 → 向量检索 → LLM解读上下文 → 生成答案。每次查询都要重复这个过程。而Nexus引入了**Context Compiler**，在构建时就把知识结构化、语境化、组合成「知识产物」（knowledge artifacts），agent运行时直接使用预编译结果，而不是每次都重新检索。

这解决的是一个真实问题：Pinecone自己的数据显示，**agent计算工作中有85%花在重新发现（re-discovery）上，而不是任务完成上**。这不是微调能修复的，是架构问题。

> 「RAG是为人类用户设计的。Nexus是为agent用户设计的，因为它们的语言非常不同。」——Pinecone CEO Ash Ashutosh

Nexus还引入了**KnowQL**作为标准查询语言——把意图、过滤、溯源、输出形状、置信度、预算六个原语封装在一个接口里。这是对标SQL的角色：在一个标准接口出现之前，每个应用都自己写数据访问层。KnowQL试图成为向量世界的事实标准。

早期客户的数据显示了一个惊人的数字：某个agent循环在217份会议记录中迭代，每次搜索部分答案再重新组装——Context Compiler将其**减少了98%**的计算量。当然，Pinecone也承认这个数字还没有在客户生产环境验证。

这场「RAG pioneer反对RAG」的叙事足够有冲击力，但背后的真实驱动力是：**向量检索正在从「工具」变成「基础设施」，而基础设施的价值在于标准之争**。

## 路线二：可组合向量搜索——Qdrant的工程赌注

第二条路线代表了另一种思路：不在检索时机上做文章，而是在**检索精度和工程控制粒度**上深挖。

Qdrant在2026年3月完成了5000万美元B轮融资，估值进入独角兽行列。他们的核心产品主张是**可组合向量搜索**（Composable Vector Search）：把索引、打分、过滤、排名每一层都做成工程师可以直接控制的原语，而不是黑盒配置。

这个设计背后的驱动是生产级RAG的真实痛点。当RAG从原型走向大规模生产环境，传统架构的局限就暴露了：写操作时索引stall、过滤在搜索之后才应用而不是在搜索过程中、高并发下尾延迟飙升。Qdrant从底层重写了这些层，让每个环节都可调。

具体的技术数字：Qdrant的**Scalar、Product和Binary量化**可以将内存使用降低数倍，同时搜索性能提升**40倍**。这不是toy benchmark，而是在真实高维向量场景下的结果。

更重要的是产品层面的差异：Pinecone在讲「知识引擎」的故事，Qdrant在讲「向量搜索是可控的」故事。前者试图重新定义问题，后者试图优化解决方案。

这两种路线的竞争，实际上是**平台思维 vs. 工具思维**的竞争。哪个对生产团队更有吸引力，取决于团队是在构建AI产品还是在构建AI基础设施。

## 路线三：多模态嵌入——Cohere打破文字边界

第三条路线不是来自向量数据库公司，而是来自embedding模型提供商，但它对RAG架构的影响同样深远。

Cohere在2026年推出了**Embed v4 Multimodal**——第一个能够同时向量化文本、图像和交错文档（interleaved documents）的embedding模型，支持**128k上下文长度**，可以一次处理200页的文档。

这打破了RAG长期以来的一个隐含假设：**知识库里只有文字**。

现实世界的企业知识库充满了扫描件、图表、截屏、带有嵌入图像的PDF。传统RAG pipeline要么把这些图片扔掉，要么单独处理图文关系。Embed v4 Multimodal用单一 embedding调用处理混合内容，解决了多模态检索的工程复杂度。

这条路线的影响是架构性的：当embedding模型本身变强了，retrieval的质量上限就提高了。向量数据库需要适应这种变化——不仅仅是存储和检索向量，还要能处理embedding模型的输出格式变化（float、int8、uint8、binary、ubinary）。

## 三条路线的分歧点

这三条路线在表面上是竞争关系，但深挖下去，它们在回答不同的问题：

| 维度 | 知识编译（Pinecone） | 可组合向量搜索（Qdrant） | 多模态嵌入（Cohere） |
|------|---------------------|------------------------|--------------------|
| 核心问题 | retrieval时机 | 检索粒度控制 | 检索内容边界 |
| 目标用户 | 构建agent系统的团队 | 需要精细控制的生产团队 | 处理混合媒体的企业 |
| 技术赌注 | 编译时推理 | 底层可组合性 | 统一多模态表征 |
| 商业叙事 | 「知识引擎」 | 「可控向量搜索」 | 「原生多模态检索」 |

这也解释了为什么「RAG死了」这个说法是错误的——**死的是那个笼统的「RAG」标签，活下来的是具体场景下的具体方案**。

## 独立观点：向量数据库正在变成「数据库」

我的一个反直觉判断是：向量数据库作为一个独立品类，可能在五年内消失——不是因为向量检索不重要，而是因为它会**融入现有数据库基础设施**。

看看现在的趋势：pgvector已经成为PostgreSQL生态的事实标准；Pinecone在做「知识引擎」而不是向量数据库；Qdrant强调可组合性。向量检索正在从独立系统变成**数据库的一个特性**。

这对独立向量数据库公司的影响是：他们必须不断向上走——从存储层到编译层再到知识层——否则会被标准化的基础设施吞噬。这解释了Pinecone为什么如此激进地从「向量数据库」转型到「知识引擎」。

这不是RAG的死亡，是RAG的**商品化**。

## 接下来看什么

1. **Pinecone Nexus的生产验证**：98%计算量减少的数字是否能在真实客户环境复现？这将是「知识编译」路线能否大规模落地的试金石。

2. **Qdrant和Pinecone的直接竞争**：两者都在抢企业AI基础设施的份额，但定位不同。2026年下半年的融资或客户动态会显示市场在用钱投票。

3. **多模态RAG的工程成熟度**：Cohere Embed v4的128k上下文是多模态RAG的重大突破，但配套的chunk策略、检索评估体系是否成熟，决定了这条路线是停留在demo还是进入生产。

4. ** KnowQL能否成为行业标准**：如果KnowQL获得广泛采用，Pinecone将从产品公司变成标准制定者——这是完全不同的商业故事。

---

## 参考资料

- [VentureBeat: The RAG era is ending for agentic AI](https://venturebeat.com/data/the-rag-era-is-ending-for-agentic-ai-a-new-compilation-stage-knowledge-layer-is-what-comes-next)
- [Pinecone Blog: Introducing Nexus Knowledge Engine](https://www.pinecone.io/blog/introducing-nexus-knowledge-engine/)
- [Pinecone Blog: Better Models Won't Save Your Agent](https://www.pinecone.io/blog/better-models-wont-save-your-agent/)
- [Qdrant Series B Announcement](https://qdrant.tech/blog/series-b-announcement/)
- [HPC Wire: Qdrant Raises $50M Series B](https://www.hpcwire.com/bigdatawire/this-just-in/qdrant-raises-50m-series-b-to-define-composable-vector-search-as-core-infrastructure-for-production-ai/)
- [Cohere: Announcing Embed Multimodal v4](https://docs.cohere.com/changelog/embed-multimodal-v4)
- [VentureBeat: Cohere launches Embed 4](https://venturebeat.com/ai/cohere-launches-embed-4-new-multimodal-search-model-processes-200-page-documents)
