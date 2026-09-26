+++
date = '2026-09-26T07:31:17+08:00'
draft = false
title = 'Python 的 GIL 拆除与 JIT 上马：3.13/3.14 里被低估的三个真相'
description = "Python 3.13 引入 free-threaded 模式和实验性 JIT，3.14 将 free-threaded 升为稳定版。这不只是「性能提升」，而是语言核心执行模型的第一次重构。本文拆解它的实际影响：free-threaded 单线程慢了 10-20%、JIT 在 3.13 里只是地基、很多 C 扩展根本还没跟上。"
tags = ["编程", "技术", "Python", "性能"]
categories = ["编程技术"]
author = "Spiral"
+++

Python 3.13 在 2024 年 10 月发布，公告打出的两个最大卖点是 free-threaded（无 GIL）模式和实验性 JIT 编译器。2025 年 10 月，Python 3.14 顺接，将 free-threaded 升为稳定版。技术社区的反应两极：一边是「Python 性能革命」的欢呼，另一边是「单线程倒退 10-20%」的冷水。

两个阵营都没说错，但都没说到点子上。

## 真相一：free-threaded 快了，是多线程快了，单线程反而慢了

「无 GIL = 更快」是最常见的误读。

GIL（Global Interpreter Lock）保证同一时刻只有一个线程执行 Python 字节码。它的存在让 Python 多线程无法真正并行，但也让单线程执行没有任何锁开销。去掉 GIL 之后，这个锁的开销转移到了 reference counting 层面：每个对象需要一个「biased reference counting」机制——拥有该对象的线程可以无锁修改引用计数，其他线程修改时需要加锁。

结果是：**free-threaded 模式下，单线程执行同一段代码，比带 GIL 的标准版慢 10-20%**。这不是 bug，是设计取舍。Meta 的 Sam Gross 在 PEP 703 里明确写了：单线程性能损耗是已知代价，换来的是多核并行能力。

对于 CPU-bound 的并行任务（如数据处理、ML 推理），free-threaded 确实能跑出接近线性扩展的并行收益。但对于绝大多数 Web 服务、脚本工具、单线程数据处理场景——你的代码大概率会变慢，而不是变快。

> 实用建议：用 `uv python install 3.13+freethreaded` 装一个 free-threaded 版本，用 `python3.13t -c "import sys; print(sys._is_gil_enabled())"` 确认进入了无 GIL 模式，跑自己的 benchmark 再决定要不要切。

## 真相二：3.13 的 JIT 只是地基，3.14 的 10-15% 也不值得现在就押注

Python 3.13 的 JIT（PEP 744）基于「copy-and-patch」技术：构建时用 LLVM 生成机器码模板（stencils），运行时把具体地址 patch 进去。这个架构没有运行时依赖，构建简单，但 3.13 里的实际加速效果是 **0-5%**，Python 官方文档明确说它是 experimental、不应在生产环境依赖。

Python 3.14 的 JIT 改进到 10-15%（部分 benchmark），这是认真的进步，但需要注意两点：

**第一，benchmark 里的 10-15% 不等于你的代码提升 10-15%。** JIT 优化的是「热路径」——被反复执行的字节码。热点是 tight loop、数值计算、递归函数的应用（典型场景：科学计算、图像处理、游戏引擎）。而 Web 请求处理、API 调用、I/O 密集型任务，JIT 帮助极其有限。

**第二，JIT 优化是面向未来的投资，不是眼前的收益。** Python 团队的计划是 3.14 打基础、3.15 继续挖潜。现在用 JIT 的人，实际上是在帮 Python 调优未来的编译器——这本身有价值，但如果你期待立竿见影的性能回报，大概率会失望。

```bash
# 检查你的 Python 是否跑了 JIT
python -X jit -c "import sys; print(f'JIT: {sys._is_jit_enabled()}')"

# 用 JIT 运行
PYTHON_JIT=1 python your_script.py
```

## 真相三：C 扩展生态的跟上速度，才是 free-threaded 真正的瓶颈

free-threaded 模式需要为每个 C 扩展单独编译一个「无 GIL 版本」，因为 C 扩展依赖 `Py_GIL_ENABLED` 宏来决定加不加锁。Python 3.13 的 free-threaded build 有独立的 ABI，C 扩展需要重新编译才能跑在 `python3.13t` 上。

这意味着什么？NumPy、pandas、psycopg2、Cython 编译的扩展——只要你依赖任何一个，它们还没发布 `3.13t` 兼容 wheel，你的 free-threaded Python 就是残缺的。pip 24.1+ 才支持 free-threaded 模式的 C 扩展安装，但 wheel 是否存在是另一回事。

现实是：2026 年初，大多数主流 C 扩展的 free-threaded wheel 还在补齐中。你真正能「无缝迁移」的场景，其实很有限。

**这个局面的影响被严重低估了。** 技术媒体的叙事是「Python 无 GIL 时代到来」，但对生产项目来说，真正的问题是：「我的依赖支持了吗？」答案经常是「还没有」。

## 独立观点：free-threaded 和 JIT 是两件事，混在一起谈会误导决策

现在技术社区有一个趋势：把 free-threaded 和 JIT 当成同一个「Python 性能升级」来讨论。这在叙事上很方便，但它们解决的问题完全不同：

- **free-threaded（无 GIL）**解决的是「多线程并行」问题——适合多核 CPU 上的 CPU-bound 并行任务
- **JIT** 解决的是「单线程执行效率」问题——适合热循环、数值计算

一个用多核换单线程性能，一个提升单线程执行速度。如果你关心的是 Web 服务响应延迟，JIT 比 free-threaded 更相关；如果你在做数据管道并行处理，free-threaded 才是重点。它们不是互相加强的关系，而是针对不同场景的工具。

把它们混为一谈，会导致两种常见错误：
1. 因为「free-threaded 无 GIL」就认为单线程 Python 会变快（错）
2. 因为「JIT 实验性」就认为 Python 性能没有改善（错，3.14 已经有实质进步）

## 接下来看什么

以下几个信号可以帮你判断这个方向的成熟度：

- **NumPy/pandas 的 free-threaded wheel 覆盖率**：到 2026 年底如果主流 C 扩展都有了 `3.13t` 兼容版，迁移成本会大幅下降
- **Python 3.15 的 JIT 进展**：3.14 的 10-15% 在 3.15 可能变成 20-30%，JIT 的 roadmap 在 PEP 744 里有明确规划
- **实际生产案例**：哪类应用在切换到 free-threaded 后真正获得了可量化的收益？这个数据目前还很稀缺
- **PyPy 的教训**：JIT 并不是新思路，PyPy 多年前就在做，生态始终没有起来。Python 官方的 JIT 会不会走出不同的路？

## 小结

Python 3.13/3.14 的 free-threaded + JIT 确实是语言史上的一次重要迭代，但「革命」这个词还为时过早。单线程性能反而慢了（free-threaded 的代价）、JIT 还在打基础（3.13 是 0-5%，3.14 才到 10-15%）、C 扩展生态还没跟上（迁移成本被低估）——这三个真相加在一起，更准确的描述是：

> **Python 正在为多核时代的并行执行能力打基础，但这个过渡期可能持续 2-3 年。对大多数开发者来说，现在最实际的收益是 Python 3.14 更快的启动时间（~10%）和更友好的错误提示，而不是 JIT 带来的执行加速。**

---

## 参考资料

- PEP 703 – Free-threaded CPython: https://peps.python.org/pep-0703/
- PEP 744 – Copy-and-Patch JIT: https://peps.python.org/pep-0744/
- Python 3.13 Release Notes: https://docs.python.org/3.13/whatsnew/3.13.html
- Python 3.14 Release Notes: https://docs.python.org/3.14/whatsnew/3.14.html
- "Python 3.13: The Performance Release We've Been Waiting For": https://devstarsj.github.io/2026/02/10/python-313-performance-features
- "Modern Python in 2026": https://labhub.hopto.org/blog/culture/2026-05-16-modern-python-2026-python-3-13-3-14-free-threaded-uv-ruff-polars-fastapi-litestar-robyn-deep-dive
