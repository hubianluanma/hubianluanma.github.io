+++
date = '2026-07-28T07:31:01+08:00'
draft = false
title = '把团队经验沉淀成可复用 AI 工具：SKILL.md 写作的五个陷阱'
description = "AI Agent 时代，Skills 是团队知识复用的最小单位。本文详解 SKILL.md 的 YAML 结构、trigger 设计、以及 Hermes Agent 场景下 skill_manage 的实操路径，并附 3 个亲手踩过的坑。"
tags = ["AI", "工具", "工作流"]
categories = ["AI 工具"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/07/28/img_1785196880_1.jpeg", alt = "工程蓝图与齿轮的暗色调隐喻" }
+++

## 前言：为什么"写一个 Skill"比"写一段 SOP"更有价值

SOP 文档需要人读、人执行、人判断——实际上只有 20% 能被真正复用。

Skills 则不同。AI Agent 在启动时读取每个 Skill 的 frontmatter name + description，把它们注入自己的系统提示词。这意味着：

- Skill 一旦写好，**每次任务触发 Agent 都会自动知道这个工具存在**
- 不需要人来翻文档，Agent 自己决定什么时候调用
- 团队沉淀的不是"操作手册"，而是**能被 AI 直接调用的能力模块**

据 Agentman Blog 2026 年调研，68% 的生产级 Agent 部署已采用 MCP 或同类标准化工具层，而 Skills 正好搭在这个基础设施之上。Anthropic 在 2026 年 2 月为企业版新增了组织级 Skill 管理功能，Vercel 也推出了官方 Skills 市场。

所以问题变成了：**怎么写一个真正能被 AI 用起来的 Skill？**

---

## 一、SKILL.md 的最小结构：三行 YAML + 一段正文

每个 Skill 目录下必须有且仅有一个 `SKILL.md` 文件。格式是 **YAML frontmatter + Markdown 正文**，缺一不可。

```yaml
---
name: my-skill
description: "当用户要求做 X 时使用本技能。包含具体步骤和验证方法。"
---
```

name 和 description 是前端matter唯二必需字段，其余全部可选。

Agent 启动时只读取这两个字段来构建"技能索引"。正文里的内容不会被预加载，只在 Skill 被触发后被完整读取执行。

### ⚠️ 陷阱一：YAML 解析失败是静默的

根据 Medium 上的一篇深度分析（bibek-poudel.medium.com，2026），**无效 YAML 会导致整个 Skill 被静默跳过**，没有任何报错，Agent 启动时完全不知道这个 Skill 存在。

常见触发原因：

```yaml
# ❌ 错误：description 用中文冒号，YAML 解析会失败
description: "当用户问：如何配置 Nginx 时使用。"

# ❌ 错误：缩进用了 tab，Python/PyYAML 解析器行为不一
    name: my-skill
  description: "..."
```

正确做法：冒号后必须空格，缩进统一用 2 或 4 个空格，永远不用 tab。

---

## 二、trigger 字段：让 Skill 在对的时间被调用

frontmatter 里可以加 `trigger` 字段，告诉 Agent 什么时候该考虑这个 Skill：

```yaml
---
name: hugo-blog-post
description: "为胡编乱码博客创建并发布新文章，或修改现有单页（about / archives / search）。"
trigger: "写博客 / 发文章 / Hugo / 博客发布"
---
```

但这里有个微妙的现实：**trigger 只能增加被考虑的概率，不等于精确触发**。Agent 收到任务后，会把自己的理解与所有已加载 Skill description 做语义匹配，而不是正则匹配 trigger 关键词。

实操建议：description 要写成一个**完整的短句**，让 AI 能直接判断这个 Skill 的适用场景，而不是堆砌关键词。

---

## 三、skill_manage 的正确用法：patch 优于 create/edit

Hermes Agent 提供了 `skill_manage` 工具，支持 create / patch / edit / delete 四种操作。官方文档明确指出 **patch 是处理定向修复的推荐方式**，而 create 适合全新 Skill。

```python
# 正确：新增一个 Skill
skill_manage(
    action="create",
    name="my-new-skill",
    category="productivity",
    content="""---
name: my-new-skill
description: "做 X 任务的标准化流程。"
---

## 步骤

1. ...
2. ...
"""
)

# 正确：修复已有 Skill 的一个错误
skill_manage(
    action="patch",
    name="existing-skill",
    old_string="旧的具体步骤描述",
    new_string="新的具体步骤描述"
)
```

### ⚠️ 陷阱二：patch 目标字符串必须全局唯一

patch 的 old_string 必须是文件中**唯一出现**的字符串，否则 replace_all 默认 false 会拒绝执行。如果同一段文本在 Skill 里出现多次，手动定位后改用 replace_all=true 或明确给出足够长的上下文让字符串唯一。

### ⚠️ 陷阱三：patch 已知 false positive

Hugo 博客的 verify 脚本会检查 cover 字段是否是 dict 结构。TOML 的 inline dict 写法 `{ image = "...", alt = "..." }` 和 section 写法 `[cover]` 是完全等价的，但 verify 脚本只认前者。遇到"cover 检查失败"但 build 产物正常的情况，**直接看 public/ 目录里的 HTML 是否有封面 URL**，不要被 verify 脚本的误报带偏。

---

## 四、一个真实场景：给团队写"代码审查" Skill

假设团队每次做代码审查都要重复同样的流程：运行测试、拉 MR、逐文件过、填写审查清单。把这个流程写成 Skill：

```yaml
---
name: code-review-workflow
description: "执行标准代码审查流程：运行测试、拉取 MR、逐文件过审、填写审查清单并提交 review。"
trigger: "代码审查 / code review / 审 MR / review PR"
---
```

正文结构建议：

```markdown
## 前提条件
- gh CLI 已登录（`gh auth status`）
- 当前目录是 Git 仓库

## 步骤

### 1. 拉取并打开 MR
\`\`\`bash
gh pr checkout $(gh pr list --state open --limit 1 --json number --jq '.[0].number')
\`\`\`

### 2. 运行测试
\`\`\`bash
npm test  # 或项目对应的测试命令
\`\`\`

### 3. 逐文件审查
重点看：安全漏洞、边界条件、性能问题、测试覆盖率。

### 4. 提交 review
\`\`\`bash
gh pr review --approve  # 或 --request-changes + 注释
\`\`\`

## 验证
review 提交后在 GitHub MR 页面能看到已提交的 review 状态。
```

---

## 五、Skill 的生命周期：从创建到维护

写完 Skill 不是终点，还需要持续维护。

根据 hermes-agent 官方文档（hermes-agent.nousresearch.com/docs），Hermes Agent 每 7 天运行一次 Curator，对现有 Skill 进行检查和优化。但 Curator 的触发条件是**至少 2 小时空闲后**，所以频繁使用的 Skill 反而不会被 Curator 自动优化——这意味着维护责任仍然在开发者身上。

维护 Signal 检查清单（自测）：

| 检查项 | 失败表现 |
|--------|---------|
| frontmatter YAML 有效 | `python3 -c "import yaml; yaml.safe_load(open('SKILL.md'))"` 不报错 |
| name 唯一 | 目录下没有同名 Skill |
| description 不含中文冒号 | 纯 ASCII 冒号或中文句号 |
| trigger 覆盖核心关键词 | 同义词 / 缩写 / 英中文变体都考虑到了 |
| 正文步骤可独立执行 | 有人照着步骤做，不需要额外上下文 |

---

## 六、3 个亲手踩过的坑（附解决方案）

**坑 1：description 写成了"功能列表"，导致 Skill 永远不被触发**

```yaml
# ❌ 这样写：Agent 看到的是一串名词，不知道什么时候该用
description: "支持 GitHub PR、GitLab MR、JIRA 工单、Slack 通知、邮件提醒。"

# ✅ 改成场景描述：Agent 能直接判断"用户要求做 X"时用这个
description: "当用户要求审查代码、提交 PR review 或查看代码变更时使用。"
```

**坑 2：正文写了步骤但没写验证方法，Skill 执行完不知道对不对**

Skill 被触发后如果缺少"怎么验证成功"这一步，执行者要么漏步骤，要么重复执行。根据 nevo.systems 的 Skill 写作指南（2026），每个步骤后应跟一行"验证标准"，哪怕是"命令返回 exit code 0"也远比没有强。

**坑 3：Skill 目录名和 frontmatter name 不一致**

Agent 工具集里 Skill 的索引 key 是 frontmatter 的 name 字段，不是目录名。目录名可以随意，name 才是实际调用依据。维护多人的 Skill 集合时，经常出现"目录叫 `blog-post`，name 写成 `hugo-blog`"的情况——这时候用户说"写博客"，Agent 找到的是 name 对应的描述而不是目录名，造成调用错位。

---

## 接下来看什么（Watch Points）

1. **Anthropic 企业版 Skill 管理的下一步**：2026 年 2 月刚推出组织级管理功能，预计年内会有更细粒度的权限控制和审计日志
2. **MCP 协议与 Skills 的关系**：Digital Applied 2026 年调研显示 68% 生产 Agent 已采用 MCP，而 Skills 是 MCP 之上的抽象层，两者的标准之争还在演进
3. **Hermes Curator 的维护策略调整**：Curator 依赖"2 小时空闲"触发，对于高频使用 Skill 可能需要手动干预来补充优化

---

## 参考资料

- [The SKILL.md Pattern: How to Write AI Agent Skills That Actually Work — Bibek Poudel, Medium, 2026](https://bibek-poudel.medium.com/the-skill-md-pattern-how-to-write-ai-agent-skills-that-actually-work-72a3169dd7ee)
- [Hermes Agent Skills System — Nous Research, 2026](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills)
- [The Agent Skills Ecosystem in 2026: Who's Building, What's Working, and What's Next — Agentman Blog, 2026](https://agentman.ai/blog/agent-skills-ecosystem-report-2026)
- [How to Write an AI Agent Skill — Nevo Systems, 2026](https://nevo.systems/blogs/nevo-journal/how-to-write-an-ai-agent-skill)
- [Agent Skills Cheat Sheet 2026 — Webfuse](https://www.webfuse.com/agent-skills-cheat-sheet)
