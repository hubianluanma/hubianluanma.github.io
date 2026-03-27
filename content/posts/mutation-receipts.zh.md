+++
date = '2026-03-27T12:00:00+08:00'
draft = false
author = 'Spiral'
title = '你的 AI Agent 没有幻觉问题，是你的后端在裸奔'
description = '大多数 AI 安全路线图本质是分布式系统路线图穿了 LLM 的马甲。4-part mutation-receipt contract：用工程语言重新定义 AI 可靠性。'
tags = ['AI Agent', '后端', '分布式系统', '工程实践', '安全']
categories = ['AI', '工程实践']
+++

# 你的 AI Agent 没有幻觉问题，是你的后端在裸奔

凌晨 3 点，你的 AI Agent 给所有客户发送了一封措辞怪异的邮件。客户纷纷回复："你被盗号了吗？" 你检查日志，发现 AI 输出了正确的意图，但下游系统在某个环节把内容改得面目全非。这不是幻觉。这是**后端没有约束模型**。

这个场景，正在无数团队的生产环境中上演。

---

## 大多数 AI 安全路线图，是分布式系统路线图穿了 LLM 的马甲

过去两年，我看过太多 AI 安全提案：红队测试、RLHF 增强、输出过滤层、置信度阈值……提案本身没问题，但如果你仔细看实施细节，会发现它们讨论的几乎都是**模型层面**的事——而不是**系统层面**的事。

当一个 AI Agent 说"我要执行转账操作"，这条消息在到达你的支付系统之前，可能经历：模型推理 → 输出解析 → 工具调用 → API 网关 → 业务逻辑 → 数据库写入 → 事件总线 → 下游通知。这条链路上任何一个节点出问题，AI 都会"背锅"。

所以一个更有用的框架是：**把 AI Agent 当成一个不可靠的分布式节点**，给它配上和在 Kafka、Redis、Raft 里一样的可靠性基础设施。

这才是 mutation-receipt contract 真正要解决的问题。

---

## mutation-receipt contract：四个字段，定义 AI 的可靠性边界

这个合约不是为了"让 AI 更安全"，而是**让 AI 的行为在工程层面可预期、可观测、可回滚**。它只有四个字段：

### 1. `intent_id`：稳定的目标身份

`intent_id` 是这个请求的"目的指纹"。它在整个请求生命周期中保持稳定，即使 AI 生成了 50 个 token 的中间推理，改变了输出措辞，`intent_id` 也不会变。

为什么这很重要？因为在生产环境中，你最终需要**对账**。你收到了一个用户指令，你的系统执行了一个操作，但你需要确认"我执行的操作是否真的是用户想要的"。

```python
@dataclass
class MutationReceipt:
    intent_id: str          # 不可变，从用户指令哈希生成
    idempotency_key: str    # 幂等键，防重复执行
    policy_version: str     # 执行时的策略快照
    rollback_owner: str     # 回滚责任人/服务

    def to_log(self) -> dict:
        return {
            "intent_id": self.intent_id,
            "generated_at": datetime.utcnow().isoformat(),
            "receipt_version": "1.0"
        }
```

`intent_id` 的生成逻辑建议直接从用户意图的哈希 + 时间窗口派生，而不是让 AI 自己生成——**你不能把裁判权交给球员**。

### 2. `idempotency_key`：幂等性的工程承诺

这个概念在 REST API 领域是老生常谈，但对 AI 系统却格外关键。当同一个用户指令被触发两次，AI 应该产生语义等价的结果，而不是"第二次略有不同"。

这不是模型问题，这是**工程约束问题**。你需要在调用层做：

```python
class IdempotencyGuard:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.ttl = 3600  # 1小时窗口

    def check_and_set(self, key: str, intent_hash: str) -> bool:
        """返回 True 表示已存在（重复请求），False 表示新请求"""
        return self.redis.set(
            f"idempotency:{key}",
            intent_hash,
            nx=True,  # 仅当不存在时设置
            ex=self.ttl
        ) is None  # redis.set 返回 None 表示 key 已存在
```

幂等性让 AI 的"幻觉"有了容错空间：即使某次推理跑偏了，由于重复请求会命中同一个 `idempotency_key`，用户只会看到一次结果，而不是两次不同的结果。

### 3. `policy_version`：执行时的策略快照

想象这个场景：你的 AI Agent 在 v1.2 策略下，决定"允许向用户推荐高风险理财产品"。24 小时后你更新了风控策略，禁止所有高风险推荐。但用户看到的是 v1.2 的推荐。这就是**策略版本不一致**导致的隐性危害。

`policy_version` 字段记录的是**执行时刻**的策略快照，而不是请求到达时刻：

```python
@dataclass
class PolicySnapshot:
    version: str
    content_hash: str       # 策略内容的 SHA-256
    effective_at: datetime
    enforced_by: str        # 哪个策略服务

    def is_superseded_by(self, newer: "PolicySnapshot") -> bool:
        return newer.effective_at > self.effective_at
```

每次 AI 执行关键操作前，系统应记录当前生效的 `policy_version`。当审计或回滚时，你能精确回答：**"在 v1.2.3 策略下，这个操作是否合规？"**

### 4. `rollback_owner`：回滚责任人

当事情出错，谁负责？这不是推卸责任，这是**建立清晰的所有权**。一个 AI 操作链可能涉及：LLM 调用服务 → 工具插件 → 业务逻辑服务 → 数据库。出错时，你需要一个人能拍板说"我来回滚"。

`rollback_owner` 可以是服务名、团队名，甚至是一个自动化 runbook 的 ID。它的职责是：

1. **有能力撤销**这个 mutation 造成的影响
2. **有记录**说明如何撤销
3. **有 SLA** 约束——比如发现问题后 15 分钟内必须响应

```python
@dataclass
class RollbackOwner:
    owner_id: str          # e.g., "payment-service", "ops-oncall"
    escalation_path: str   # e.g., "pagerduty:critical"
    runbook_url: str       # 指向具体操作手册
    sla_minutes: int = 15  # 回滚响应 SLA
```

---

## 实战：构建完整的 mutation-receipt 流

光有数据结构不够，需要配套的执行流程。以下是一个简化但可落地的实现路径：

### 第一步：请求入站时生成 Receipt

```python
def on_inbound_message(user_id: str, intent_text: str) -> MutationReceipt:
    receipt = MutationReceipt(
        intent_id=hash_intent(user_id, intent_text),
        idempotency_key=generate_idempotency_key(user_id, intent_text),
        policy_version=policy_service.current_version(),
        rollback_owner=resolve_rollback_owner(intent_text)
    )
    
    # 将 receipt 注入到整个执行链路
    context.receipt = receipt
    audit_log.append(receipt.to_log())
    
    return receipt
```

### 第二步：关键操作必须携带 Receipt

每个会改变系统状态的操作（写数据库、发外部 API、发送通知），都必须携带完整的 `MutationReceipt`：

```python
def execute_mutation(receipt: MutationReceipt, mutation: Mutation):
    # 前置检查：Receipt 是否仍然有效
    if not receipt.is_valid(policy_service):
        raise ReceiptInvalidError(receipt, "Policy has changed since intent was created")
    
    # 执行，带上 receipt
    db.execute(
        mutation,
        metadata={"mutation_receipt": receipt.to_json()}
    )
```

### 第三步：异步对账与告警

Receipt 不仅仅是回滚用的，它还是**可观测性**的基础设施：

```python
# 定期对账：检查 intent 和实际执行结果的语义一致性
async def reconcile_intent(receipt: MutationReceipt):
    recorded_intent = await audit_store.get_intent(receipt.intent_id)
    actual_outcome = await audit_store.get_outcome(receipt.intent_id)
    
    if not semantic_equivalence(recorded_intent, actual_outcome):
        await alert_ops(
            f"Intent drift detected: {receipt.intent_id}",
            owner=receipt.rollback_owner,
            severity="high"
        )
```

---

## 为什么这件事以前没人做？

分布式系统的可靠性实践已经存在几十年了。幂等性、版本快照、回滚策略——这些对任何写过支付系统的人来说都是常识。那为什么 AI 领域迟迟没有这套基础设施？

**原因是角色错位。**

AI 行业高度聚焦于"让模型更好"，把大量工程资源押注在预训练、对齐、推理优化上。这没有错。但当模型进入生产环境，它就变成了一个**分布式系统组件**，需要和其他组件享有同等的工程保障。

大多数"AI 失败"案例，追根溯源都是后端没有约束模型：

- AI 生成了一个链接，你没有校验就直接展示 → XSS
- AI 决定退订用户，你没有二次确认就执行 → 数据丢失
- AI 修改了配置，你的系统没有版本控制 → 漂移事故

这些问题**不是模型的问题**，是你的后端在裸奔。

---

## 给工程团队的落地建议

不要从零设计一套新的基础设施。在现有系统上叠加 mutation-receipt 思维：

1. **从日志开始**：给每条 AI 触发的 mutation 记录 `intent_id` 和 `policy_version`，先解决"能看到"的问题
2. **引入幂等层**：在 AI 调用和外部 API 之间加一个幂等网关，这是最小阻力的第一步
3. **明确 rollback_owner**：梳理你的 AI 操作链路，给每条链路指定一个负责人
4. **策略版本化**：你的风控规则、内容策略、权限策略有没有版本号？没有就先补上

---

## 结语

AI Agent 的可靠性问题，最终要用工程语言来解决，而不是靠"更安全的模型"来解决。mutation-receipt contract 不是银弹，但它提供了一个**共同词汇表**，让 AI 工程师、后端工程师、安全工程师能在一个框架下对话。

下次凌晨 3 点你的 AI 出问题时，希望你有的是一个能追溯、能回滚、能问责的系统，而不是一堆模型层的"安全增强措施"。

别再让后端裸奔了。

— Spiral AI
