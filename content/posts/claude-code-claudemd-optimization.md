+++
date = '2026-08-25T07:31:56+08:00'
draft = false
title = "Claude Code 每次多花800 token？一张 CLAUDE.md 把账单砍一半"
description = "CLAUDE.md 是 Claude Code 里最大的单次成本杠杆。Benchmark 数据：优化后可节省 62% tokens（1300 token/次），$500/月 API 账单降到 $40。附具体配置方法和踩坑实录。"
tags = ["AI", "工具", "Claude"]
categories = ["AI 工具"]
author = "Spiral"
+++

Claude Code 每次启动会话，光 startup tokens 就烧掉约 800 个。这不是模型回答，是**纯开销**——每开一个新 chat，这 800 token 就消失了，但你付的钱一样多。

 CLAUDE.md 是这个问题的最优解。它在项目根目录，每次 Claude Code 启动时自动加载，成本却只算一次。问题是：大多数人的 CLAUDE.md 写得太厚、太多、反而把 startup 开销撑得更大。

 核心数据来自两个公开 benchmark（来源见文末）：

 - **Boring Bot 测试**：优化后 CLAUDE.md 裁剪到「模型无法自行推断的内容」，context reduction 达 **91.9%，无质量回撤**。月 API 费用 $500 → $40。
 - **Reddit r/ClaudeAI 用户实测**：通用型 CLAUDE.md 模板，让 Claude 输出 token 减少 **63%**，约 384 tokens/4 次对话。

 这篇文章不写「Claude Code 是什么」——写的是**怎么配 CLAUDE.md，才能让这 800 个 startup token 花得值**。

## CLAUDE.md 的本质：它是缓存，不是说明书

 新手最容易犯的错：把 CLAUDE.md 当作「给 Claude 的完整项目介绍」。塞架构图、塞代码规范、塞十几条 dos and don'ts。效果是反的——CLAUDE.md 每一行都在**每次 API 调用**里重复计费，塞得越厚，每次对话成本越高。

 正确认知：**CLAUDE.md = 模型无法自行推断的上下文**。代码库里已经有的东西，不需要重复写。

 实践原则（优先级从高到低）：

 1. **项目结构**：Claude 读不到你本地目录树，写清楚 src/、tests/、docs/ 在哪
 2. **技术约束**：用了哪些 linter/formatter、CI 强制要求什么、数据库迁移规则
 3. **业务规则**：认证流程、支付逻辑、领域模型的核心概念（不是实现细节）
 4. **不要写**：具体函数实现、已存在的代码片段、常规编程知识

## 实战配置：三种场景，三种模板

### 场景一：中小型 Web 项目（推荐配置）

```markdown
# 项目结构
src/          # 业务逻辑
tests/        # 测试文件
scripts/      # 运维脚本

# 技术栈
- Node.js + TypeScript
- Prisma ORM + PostgreSQL
- Jest 测试框架

# 规范
- ESLint + Prettier 强制开启
- 提交前必须 `npm test` 通过
- 禁止直接操作 `process.env`，统一走 config.ts
```

 行数：约 18 行。Token 估算：**< 500 tokens**。

### 场景二：大型 monorepo（分块加载策略）

 Monorepo 里塞满整个项目结构不现实。改用**分段文件引用**：

```bash
# 根目录放 CLAUDE.md（极简）
# 项目级上下文放各子包 CLAUDE.md，Claude 按需读取
```

 具体方案：

 1. 根目录 CLAUDE.md：只写 monorepo 整体架构 + 各 package 职责
 2. `packages/api/CLAUDE.md`：API 层专项约束（中间件顺序、错误码规范）
 3. `packages/web/CLAUDE.md`：前端专项（路由规则、状态管理约定）

 Claude Code 的 `@` 引用文件功能，让你在需要时手动加载专项上下文，而不是每次 startup 全量注入。

### 场景三：多人协作团队（Allowlist > Blocklist）

 团队里常见写法：

 ```markdown
 # 不要做的事
 - 不要用 var
 - 不要直接拼接 SQL
 - 不要忽略 TypeScript 错误
 - 不要用 console.log 调试
 ...
 ```

 这种 blocklist 越写越长，CLAUDE.md 越来越厚。改用 **allowlist**：

 ```markdown
 # 允许的模式
 - 只允许 const / let
 - SQL 统一用 Prisma Client 或 Knex
 - 必须显式声明类型，不接受 `any`
 - 日志统一走 logger.ts，禁止 console.*
 ```

 Allowlist 的 token 效率更高：说「可以做什么」比说「不能做什么」更短、更明确、更不容易被钻空子。

## 实测数字：配 vs 不配，差多少

 从公开 benchmark 提取的数字：

 | 指标 | 无优化 CLAUDE.md | 优化后 CLAUDE.md | 节省 |
 |------|------------------|-----------------|------|
 | Startup tokens | ~800 | ~300 | **62%** |
 | 每次会话 token 消耗 | ~2100 | ~800 | **62%** |
 | 月 API 费用（100 sessions/天，5人团队） | $360 | $72 | **$288** |
 | 模型输出质量 | — | 无显著回撤 | — |

 来源：Medium @jpranav97 实测（月费用 $72 = 100 sessions/天 × 5 devs，2026 年 4 月数据）；Boring Bot Substack benchmark（91.9% context reduction，无质量回撤）。

### /clear 的正确用法

 做完一个独立任务，用 `/clear` 重置 context window——这是免费的操作，不省 token，但能**防止 context 里积累无用历史**，让后续任务从干净状态开始。

 具体策略：

 - 每个独立功能开发：开发前 /clear
 - 每个 bug 修复：修复完成后 /clear
 - 每个 PR review：review 完成后 /clear

 不要等 context 满了才 /clear，那时候模型已经开始「忘记」早期上下文了。

## 踩坑实录：什么时候 CLAUDE.md 会帮倒忙

 CLAUDE.md 不是万能药。以下情况，不写比写更好：

 **情况一：低频一次性任务**

 改一个配置文件、查一个 bug，这类低频操作不值得专门写 CLAUDE.md。开销（CLAUDE.md 每次加载的那几百 token）可能超过它带来的节省。

 **情况二：多项目并行开发**

 同一台机器开了 5 个 Claude Code session，每个都加载了相同的 CLAUDE.md——这不是浪费，是**混淆**。解决方案：用 `@` 按需引用，而不是依赖全局加载。

 **情况三：CLAUDE.md 超过 2000 tokens**

 超过这个阈值，每次 API 调用的固定开销就太大了。此时应该**拆分**：主 CLAUDE.md 只保留最核心的 200 行，专项上下文移到子文件。

## .claudeignore：另一个被忽视的省 token 工具

 Claude Code 默认会扫描整个项目目录来「理解」结构。`node_modules/`、`dist/`、`.git/` 这些目录既没用又烧 token。

 在项目根目录加 `.claudeignore`：

 ```
 node_modules/
 dist/
 .git/
 coverage/
 *.log
 ```

 一次性配置，每个 session 永久生效。

## 总结：配置清单

 0. **CLAUDE.md 写什么**：模型无法自行推断的项目结构 + 技术约束 + 业务规则；不写代码实现
 1. **目标行数**：18–35 行；超过 500 tokens 就拆
 2. **模式选择**：allowlist > blocklist
 3. **monorepo**：拆到子包 CLAUDE.md + @ 按需引用
 4. **.claudeignore**：必须配，排除 node_modules/dist/.git
 5. **/clear**：每个独立任务完成后执行
 6. **验收指标**：Startup tokens 从 ~800 降到 ~300；每次会话节省 62% tokens

## 接下来看什么

 - **你的团队月 API 账单是多少？** 代入上面的表格算一下，$288/月节省值不值得改
 - **CLAUDE.md 的 token 计数**：Claude Code 本身不显示 CLAUDE.md 加载了多少 token，可以用 API 日志或第三方 proxy 工具测一次，拿到自己的基准线再优化
 - **Anthropic 官方文档**：Anthropic 官方的 [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)（Brave 搜索结果原文已无法访问，可通过 Wayback Machine 或官方 blog 访问）给出了 hooks + skills 的组合用法，比单纯写 CLAUDE.md 更系统

---

**参考资料**

1. Boring Bot Substack: *How to save millions in Claude tokens (code included)* — 91.9% context reduction benchmark, $500→$40 月账单
   https://boringbot.substack.com/p/how-to-save-millions-in-claude-tokens
2. Reddit r/ClaudeAI: *I built a universal CLAUDE.md that cuts Claude output tokens by 63%* — ~384 tokens saved per 4 prompts
   https://www.reddit.com/r/ClaudeAI/comments/1s7qu07/i_built_a_universal_claudemd_that_cuts_claude/
3. Medium @jpranav97: *Stop Wasting Tokens: How to Optimize Claude Code Context by 60%* — $0.024/session, $72/月 (5 devs, 100 sessions/天)
   https://medium.com/@jpranav97/stop-wasting-tokens-how-to-optimize-claude-code-context-by-60-bfad6fd477e5
4. GitHub drona23/claude-token-efficient — Drop-in CLAUDE.md 模板，减少 output verbosity
   https://github.com/drona23/claude-token-efficient
5. SmartScope Blog: *Claude Code Advanced Best Practices: 11 Practical Techniques for Hooks, Subagents & Context Management [2026]* — hooks + skills + subagents 系统化用法
   https://smartscope.blog/en/generative-ai/claude/claude-code-best-practices-advanced-2026/
