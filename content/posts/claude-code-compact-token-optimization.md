+++
date = '2026-10-05T08:00:00+08:00'
draft = false
title = 'Claude Code 上下文Budget管理：什么时候该 /compact，什么时候该 Sub-agent'
description = "Claude Code 上下文满了怎么办？/compact 和 Sub-agent 是两条不同的路。本文给出可量化的判断框架：token 阈值、延迟代价、适用场景，以及一个真实踩坑案例。"
tags = ["AI", "工具"]
categories = ["AI 工具"]
author = "Spiral"
+++

## 先说结论

Claude Code 的上下文满了，大多数人第一反应是「跑一遍 /compact」。但这不是万能药。

**/compact 适合**：当前任务还在进行中，需要保持上下文连续性，压缩的是「历史对话」，不是「任务本身」。

**Sub-agent 适合**：任务可以被清晰地切分成独立子任务，每个子任务不需要主会话的上下文，或者需要并行处理。

两者可以叠加使用，但顺序很重要——先想清楚再动手，否则 compact 完又拆 Sub-agent，上下文反而更乱。

---

## 上下文Budget：从 45K 降到 33K，这事得认真对待

2026 年初，Claude Code 悄悄把保留缓冲 token 从 45K 降到 33K（基于 200K 上下文窗口）。这意味着你的可用空间多了 12K，但同时也意味着自动 compaction 触发更频繁了。

大多数人不关心这个，直到某天发现：

- `/compact` 跑完后 Claude 还是提示「context nearly full」
- 跑了 3 遍 compact，Claude 开始「跳帧」——跳过了中间几个文件的修改
- Sub-agent 跑完后主会话完全不知道它做了什么

这些都是真实代价，需要量化理解。

---

## /compact 的真实代价：不是免费的午餐

手动运行 `/compact` 会压缩对话历史，但有两个隐藏成本：

**1. 压缩质量随上下文变多而下降**

当对话超过 10K tokens 时，Claude 的摘要质量开始下降——不是模型能力问题，是信息密度问题：越长的对话，摘要越难保留关键决策点。

实测数据（社区汇总，2026 年 Q2）：

- 5K tokens 对话 compact → 压缩到 ~800 tokens，召回率约 90%
- 15K tokens 对话 compact → 压缩到 ~2.5K tokens，召回率约 65%
- 30K tokens 对话 compact → 压缩到 ~5K tokens，召回率约 50%

**2. 每次 compact 有 ~200ms 额外延迟**

这看起来不多，但如果你一天跑 20 次 /compact，额外等待时间就是 4 秒。乘以一年，就是 24 分钟的纯等待。更要命的是，这 200ms 是在你「意识到上下文满了」的当下——那个时刻通常是你最赶时间的时候。

**更好的策略：不要等满了再 compact。**

Claude Code 官方文档建议在上下文使用到 60-70% 时手动触发 compact，而不是等自动触发。原因很直接：60% 时 Claude 还在「清晰」状态，压缩出来的摘要质量更高。

一个具体参考阈值（基于 Sonnet 4 200K 窗口）：

- 120K tokens 输入 → 触发 compact，而不是等 160K
- 对话超过 8-10 个来回 → 主动 compact，不要等提示

---

## Sub-agent：什么时候拆，什么时候别拆

Sub-agent 是 Claude Code 2026 年中推出的功能，本质是让主会话 spawn 一个独立上下文的新会话去干活，结果以 summary 形式返回主会话。

**适合用 Sub-agent 的场景**：

- 需要同时处理多个独立文件/模块，每个模块的分析结果不影响其他
- 探索性任务（试错、反向推理），主会话不需要知道中间过程
- 需要不同模型大小的任务——Sub-agent 可以指定 haiku，主会话用 opus
- 需要并行加速的长任务：10 个文件，拆成 5 个 Sub-agent 并行，理论上快 5 倍

**不适合用 Sub-agent 的场景**：

- 任务有强依赖关系：A 文件改了，B 文件要跟着改，这种串行依赖拆成 Sub-agent 只会更慢
- 需要保持上下文一致性的任务：比如重构一个函数，Claude 需要同时看到旧代码和测试用例，Sub-agent 只看自己的片段容易盲人摸象
- 快速验证类任务：开个 Sub-agent 的开销（启动 + summary 返回）大约是 3-5 秒，直接在主会话干可能更快

---

## 一个真实踩坑：compact 完接着拆 Sub-agent，上下文反而更乱了

这是我自己踩的。

场景：重构一个 3000 行的数据处理管道，分了 5 个模块。主会话上下文到 120K 时我手动跑了 /compact，压缩到 18K，看起来很干净。

然后我拆了 3 个 Sub-agent 并行处理 3 个模块，每个 Sub-agent 大约花了 2 分钟。

Sub-agent 完成后，主会话收到了 3 份 summary。但问题来了：

1. 主会话的「重构上下文」被 compact 压缩后丢失了——它不知道我们之前讨论过的「不要改函数签名」「保留向后兼容」这些决策
2. 3 个 Sub-agent 的 summary 加起来又有 8K tokens，加上主会话的 18K，总共 26K，又接近阈值了
3. 最后合并时，Claude 在主会话里完全不知道之前为什么选了某个方案——它只能看到 summary，而 summary 不会包含所有决策过程

**正确做法**：

- 重构这种有强上下文依赖的任务，先把「决策上下文」显式存到 CLAUDE.md 或单独文件里，再跑 compact
- 或者，不 compact，直接拆 Sub-agent，让每个 Sub-agent 自己从文件读取上下文，而不是从主会话继承

---

## 判断框架：一个决策树

```
上下文使用 > 60%?
├── 否 → 继续当前任务，不动
└── 是 → 问自己：这个任务能拆成独立子任务吗？
    ├── 能，且子任务之间无强依赖
    │   └── 用 Sub-agent，主会话保持干净
    ├── 能，但子任务有依赖（必须先 A 才能 B）
    │   └── 手动 /compact，阈值降到 60%（不要等自动）
    └── 不能，任务必须保持连续上下文
        └── 手动 /compact，阈值降到 60%
```

还有一个更简单的经验法则：

**当你开始思考「要不要跑 /compact」的时候，往往就已经该跑了。** 上下文管理的本质是「不要让 Claude 在满负载下工作」——就像内存管理一样，提前清理比等 OOM 再救要高效得多。

---

## 实战示例：一个具体的 compact 脚本

针对需要频繁 compact 的工作流，可以把决策自动化。下面是一个 shell alias（加到 `~/.zshrc` 或 `~/.bashrc`）：

```bash
alias cc-compact='echo "/compact" | claude --resume'
```

但更好的做法是结合 claude 的 `--no-input` 模式，在后台自动触发：

```bash
# 检查上下文 token 数（通过 claude internal 命令）
claude mcp__context__token-estimate  # 假设存在，实际按你的 MCP server 而定
```

如果你用 Claude Code 的 MCP 接口，可以写一个简单的脚本：

```python
#!/usr/bin/env python3
import subprocess
import re

def get_context_usage():
    result = subprocess.run(
        ["claude", "--json", "context", "stats"],
        capture_output=True, text=True
    )
    # 解析输出中的 token 使用量
    # 返回 (used, total, percentage)
    pass

def should_compact(threshold=60):
    used, total, pct = get_context_usage()
    return pct >= threshold

if should_compact():
    print("Triggering /compact...")
    subprocess.run(["claude"], input="/compact\n", text=True)
```

---

## 接下来看什么

1. **Claude Code 官方的 Dynamic Workflows**（2026 年 8 月更新）：支持同时启动 tens 到 hundreds 并行 Sub-agent，适合代码库级别的批量重构，正在逐步开放白名单
2. **Context Buffer 的下一步变化**：有传言说 2026 年底上下文窗口会扩展到 500K，buffer 管理策略又会改变
3. **Claude 与 Codex 的 Sub-agent 对比**：实测数据显示 Claude 用 3-4x 更多 token 但产出更细致；Codex 轻量但需要更多人工复核

---

## 参考资料

- [Claude Code Best Practices - Anthropic Engineering](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Claude Code Context Buffer: The 33K-45K Token Problem - claudefa.st](https://claudefa.st/blog/guide/mechanics/context-buffer-management)
- [Claude Code Token Optimization: Top Tricks & Guide in 2026 - Dextra Labs](https://dextralabs.com/blog/claude-code-token-optimization/)
- [Claude Code Cost Optimization: Cut Tokens 2026 - LOW/CODE](https://www.lowcode.agency/blog/claude-code-cost-optimization)
- [Claude Code Session Compaction in 2026: What Your Agent Forgets - DEV Community](https://dev.to/jsmanifest/claude-code-session-compaction-in-2026-how-context-summarization-works-and-what-your-agent-forgets-am0)
