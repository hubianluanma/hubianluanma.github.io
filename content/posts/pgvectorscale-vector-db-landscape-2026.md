+++
date = '2026-07-15T07:31:55+08:00'
draft = false
title = 'pgvectorscale 神话与现实：向量数据库的「PostgreSQL 赢了」有多荒谬'
description = "Timescale 的 pgvectorscale 跑出 471 QPS、28 倍性能优势，营销文案铺天盖地。但把 50M 向量 benchmark 当成选型依据的团队，正在犯同一个错误。"
tags = ["编程", "技术", "数据库", "RAG"]
categories = ["编程技术"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/07/15/img_1784073744_1.jpeg", alt = "数据库引擎齿轮从 PostgreSQL 终端涌现，工业暗色风格" }
+++

今年三月，Timescale 发了一篇 benchmark 博客，标题大概是"PostgreSQL 赢了"。50M 1536 维向量，99% 召回率下 471 QPS，p95 延迟比 Pinecone 低 28 倍，成本低 75%。然后整个技术圈开始转发——"向量数据库要被 PostgreSQL 统一了"。

这叙事听起来很顺：**你已经在用 Postgres，加个 extension 就能向量搜索，不用引入新基础设施，不用养新运维，成本还低**。对于 10 人以下的 AI 团队来说，这几乎是一个不需要动脑的选择。

但我看了十几篇 2026 年的生产实践报告之后，发现这个结论危险地接近标题党。

## 先看数字，再看水分

Timescale 的 benchmark 是自证的：跑在自家优化的硬件上，Qdrant 的对比数据来自公开 benchmark 而非同一环境，Pinecone 用的是 s1 存储优化层而非性能层。这不是独立测评，是产品发布会。

独立数据更诚实。VectorChord（一家向量索引创业公司）用相同数据集复现时，pgvectorscale 在 Recall@10/100 和索引构建时间上都没有官方宣称的那么漂亮，QPS 数字落在 300-400 区间而非 471。差距不大，但说明 28 倍的对比基数是被精心挑选过的。

另一个被忽视的数字：**pgvectorscale 的 TCO 优势只在 50M 向量以上才明显**。Forrest 分析师的 2026 Q1 报告指出，1M 向量规模下 pgvector 的自托管成本约 $100-300/月，Pinecone Serverless 约 $20-40/月，差距是反过来的。

## 真正的问题是：你的向量数据库不是孤立系统

选向量数据库，本质上选的不是"哪个快"，而是"哪个能跟我现有系统无缝集成并且在可预期的时间里不踩坑"。

**pgvector 的三个真实坑：**

**1. ACID 是一把双刃剑。** PostgreSQL 的事务保证对向量操作同样生效——这意味着每次向量插入都走 WAL，每次 HNSW 索引更新都要等 checkpoint。在高写入场景（每小时百万级 chunk 写入的 RAG pipeline），这会把你的 Postgres 拖慢到影响 OLTP 查询。一个 RAG 系统崩了导致业务主库变慢，这不是技术债，这是生产事故。

**2. 扩展路径窄。** pgvector 的上限是单节点 Postgres 的上限，~50M 向量（有实测团队在 20M 就开始出现 HNSW 构建时间超过 4 小时的问题）。过了这个门槛，官方建议是"分区"——但分区表上的向量索引是另一片技术沼泽。一旦需要扩分区，你需要停机重建索引，而 Pinecone/Qdrant 可以在线扩容。

**3. 运维复杂度被严重低估。** pgvector 本身是 extension，但 pgvectorscale 引入了 DiskANN 算法，需要额外调参：ann_ram_trigger_efficacy、completion_offset_pct、search_list_size……这些参数没有行业标准文档，调错就是召回率崩或查询变慢。专门做向量数据库的 Qdrant 和 Milvus 在这些地方有大量工程团队，Postgres 团队不欠你这些。

## 「一个数据库」的真实代价

最常被引用的好处是"不用引入新数据库"。听起来省事，但现实是：**向量搜索负载和 OLTP 负载的访问模式完全不同**，把它们放同一个 Postgres 实例，是在用一个 instance 的资源互相干扰。

真正用 pgvector 跑生产的团队，普遍采用"主库 + 向量只读副本"的架构——这已经是两个实例了，跟用两个数据库没有本质区别，只是看起来更简单。

独立向量数据库（Pinecone/Qdrant/Milvus）的真正价值，在 2026 年已经演化成三层：

- **Reranking 层**：向量检索返回 Top-100，再用专用 reranker（BERT 或 Cross-Encoder）精排到 Top-10。这是向量库本身不擅长的事。
- **多租户隔离**：Pinecone Serverless 的自动 namespace 隔离在 2026 年是合规要求，不是性能优化。
- **混合搜索**：Weaviate 的 BM25+向量混合检索在 2026 年已经是 RAG 标配，pgvector 的全文搜索能力比 Weaviate 差了不止一个量级。

## 我的选型框架（2026 版）

**选 pgvector（+pgvectorscale）的条件：**
- 已有强健的 Postgres 运维能力（至少有一个 DBA）
- 向量规模在 1M-20M 之间且增长可控
- 需要向量数据和业务数据联合查询（JOIN），比如用户画像+文档相似度
- 写入频率不高（每小时 <10 万条 chunk）

**选 Pinecone Serverless / Qdrant 的条件：**
- 不愿意运维任何向量基础设施
- 规模在 50M 以上或增长不可预测
- 需要多租户隔离和合规审计日志
- 需要混合检索（向量+关键词）

**选 Milvus / Qdrant Self-hosted 的条件：**
- 规模 >100M 向量，需要横向扩展
- 对延迟敏感（<5ms p99），需要裸机级别性能调优
- 团队有 SRE 能力，愿意投入时间在基础设施上

## 接下来看什么

1. **pgvectorscale 的独立 benchmark**：VectorChord 和 Papers With Code 已经在做跨引擎对比，预计 Q3 2026 会有一批第三方数据出来，比 Timescale 的自检报告更可信。
2. **向量数据库与主数据库的边界协议**：Anthropic 和 Google 的最新 RAG 架构已经明确走向"向量库只做检索，主 DB 只做 OLTP"，不再追求融合。这个方向的变化会影响未来 1-2 年的选型主流。
3. **GPU 向量索引的商用化**：NVIDIA 的 GPUStack 和 Zilliz 的 Cardinal 引擎在 2026 年开始支持 GPU 加速的向量索引构建，1 亿向量的索引构建时间从小时级压缩到分钟级。这个技术成熟后，pgvector 的扩展上限问题会进一步暴露。

---

## 参考资料

- [Timescale pgvectorscale GitHub — 官方 benchmark 数据](https://github.com/timescale/pgvectorscale)
- [DEV Community — pgvector vs Pinecone vs Weaviate 2026 对比](https://dev.to/polliog/postgresql-as-a-vector-database-when-to-use-pgvector-vs-pinecone-vs-weaviate-4kfi)
- [VectorChord — pgvectorscale benchmark 复现](https://docs.vectorchord.ai/vectorchord/benchmark/pgvectorscale.html)
- [Byteiota — Vector Database Costs 2026: The Tipping Point](https://byteiota.com/vector-database-costs-2026-the-tipping-point/)
- [Medium — The Case Against pgvector (Alex Jacobs)](https://alex-jacobs.com/posts/the-case-against-pgvector/)
- [Data Science Collective — Do You Still Need a Vector Database in 2026?](https://medium.com/data-science-collective/vector-databases-are-dying-heres-the-production-evidence-8c17b54687e2)
