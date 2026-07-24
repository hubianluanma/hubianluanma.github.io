+++
date = '2026-07-24T07:31:15+08:00'
draft = false
title = 'MCP OAuth 共享凭证灾难：一条限流引发的 50 租户级联故障'
description = "MCP Server 生产环境中，OAuth 共享凭证如何把一个租户的 rate limit 爆炸成全场灾难，以及如何用 per-user token isolation 修复它。"
tags = ["AI", "工具"]
categories = ["AI 工具"]
author = "Spiral"
cover = { image = "https://minio-api.hubianluanma.com/blog/images/2026/07/24/img_1784851263_1.jpeg", alt = "interconnected server nodes cascading failure" }
+++

凌晨两点，Sentry 的 MCP Server 突然开始批量拒绝 Cursor 的工具调用。错误信息清一色是 429 Rate Limited。值班工程师花了两小时排查，发现问题源头是一个大客户在调试时跑了循环脚本，把自己的 Notion OAuth 凭证耗尽了——然后这个凭证同时挂在 50 个共享这个 integration 的租户下面，一个人的限流把其他 49 个客户全炸了。

这不是孤例。2026 年上半年，至少三家主流 MCP 托管服务商报告过类似的级联故障。问题不在 MCP 协议本身，而在于实现者把 OAuth 2.0 的「代表用户」语义套到了「代表租户」的多租户架构上——字面意义上，一颗老鼠屎坏了一锅粥。

## 问题根源：OAuth token 在多租户场景下的语义错配

MCP Server 通常这样接入 Notion：

```python
# 常见错误写法（不要学）
notion_client = NotionClient(
    token=os.environ["NOTION_INTEGRATION_TOKEN"]  # 一个 token，N 个租户共享
)
```

MCP 官方文档里，这种写法随处可见。一个 internal integration token 对应一个 Notion workspace，但工程师把它当成了「所有租户共享的 API key」——这在 OAuth 授权码流程里是错的，因为每个终端用户应该用自己的 refresh token 换取 access token，而不是大家共用一个。

正确的模型应该是：**每个终端用户（或者说每个租户实例）持有一个独立的 OAuth token 对**。

两种解法：

**解法 A：per-user token（推荐）**

每个租户在授权后，服务器存储该租户独有的 `access_token` + `refresh_token` 对。请求时动态路由：

```python
class TenantTokenManager:
    def __init__(self, db):
        self.db = db  # 存 per-user OAuth tokens

    def get_client(self, tenant_id: str) -> NotionClient:
        tokens = self.db.get_tokens(tenant_id)
        if tokens["expires_at"] < time.time():
            tokens = self._refresh(tokens)
            self.db.save_tokens(tenant_id, tokens)
        return NotionClient(token=tokens["access_token"])

    def _refresh(self, tokens: dict) -> dict:
        resp = oauth_client.refresh_token(
            tokens["refresh_token"],
            provider="notion",
            client_id=NOTION_CLIENT_ID,
            client_secret=NOTION_CLIENT_SECRET
        )
        return {
            "access_token": resp["access_token"],
            "refresh_token": resp["refresh_token"],
            "expires_at": time.time() + resp["expires_in"]
        }
```

这样 A 租户被限流，只影响 A。B 租户的 access token 完全独立。

**解法 B：token bucket + 熔断**

如果短期内无法改造 per-user token 架构，至少在 MCP Server 入口加一层 token bucket + 熔断：

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address, default_limits=["60/minute"])

@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    # 按 tenant_id 隔离 bucket，不是按 IP
    tenant_id = request.headers.get("X-Tenant-ID", "anonymous")
    bucket_key = f"mcp:{tenant_id}"

    if not rate_limiter.consume(bucket_key, 1):
        return JSONResponse(
            {"error": "rate_limit_exceeded", "tenant_id": tenant_id},
            status_code=429
        )
    response = await call_next(request)
    return response
```

但这只是缓解，不是根治。真正的问题在数据模型层。

## 隐藏成本：custom MCP server 每年 50K-150K 美元

Truto 在 2026 年中的测算显示，企业自建一个 MCP integration（含 OAuth、token 管理、多租户隔离）平均成本：

| 成本项 | 低配版 | 生产级 |
|---|---|---|
| 初始开发 | $20K | $50K |
| OAuth 合规（PKCE、token 轮换） | $8K | $20K |
| 多租户隔离工程 | $10K | $30K |
| 年度维护 + 合规更新 | $12K | $50K |
| **合计** | **$50K** | **$150K** |

这不是在吓你，而是在说：如果团队规模 < 10 人，自建 MCP Server 几乎一定是错误的选择。托管方案（Cloudflare AI Gateway、MCP Gateway、Truto）提供开箱即用的 per-user token 隔离，按 seat 计费，省去的运维成本远低于自建。

踩坑之前，先问自己：**这个 MCP 真的值得自建吗？**

## 一个具体的故障复盘：Sentry MCP 的 429 级联

GitHub Issue #844 记录了完整的事故链：

1. Cursor 用户 A 在 Sentry MCP 上跑一个 bug bounty 扫描脚本
2. 脚本在 20 秒内触发了 ~200 次 `sentry_api.search_issues` 调用
3. Sentry 的 rate limit 是 60 req/min（共享给所有使用同一 OAuth credential 的用户）
4. 429 从 A 的请求扩散到同一 Cursor workspace 的所有用户
5. 值班工程师 2 小时后才发现是 OAuth credential 共享模型的问题

Sentry 官方后续修复方案：**切换到 per-user OAuth token 架构**。代价是工程量约 3 周。

如果你正在评估 MCP 托管方案，问供应商这两个问题：

1. **「你们的 rate limit 是 per-user 还是 per-integration？」** 如果答案是 per-integration，直接 pass
2. **「多租户场景下，某个租户的 token 被 revoke，其他租户会受影响吗？」** 如果答案是「会」或者「不确定」，同上

## 什么时候 MCP Server 不该用 OAuth

MCP 协议本身支持三种认证模式：

| 模式 | 适用场景 | 风险 |
|---|---|---|
| 无认证 | 内部工具、完全信任的网络 | 完全暴露 |
| API Key | 单租户或 B2B 点对点 | key 轮换困难 |
| OAuth 2.1 + PKCE | 多租户 SaaS | 实现复杂，有上述踩坑风险 |

**不要用 OAuth 的场景**：
- 团队内部工具，机器和用户都在同一个受控网络里
- 只跑一个固定的数据源，不需要动态授权撤销
- 用户量和 token 生命周期管理不复杂的工具

**用 OAuth 的场景**：
- 用户需要连接自己的 Notion/GitHub/Google Workspace 账号
- 需要支持「随时撤销某个第三方账号的访问权限」
- 多租户隔离是合规要求（比如 GDPR 数据处理协议）

硬要用 OAuth 但团队 < 5 人？直接用官方维护的 `makenotion/notion-mcp-server`，不要自己造轮子。

## 接下来看什么

1. **MCP Gateway 的 per-tenant token 隔离方案**：Cloudflare 刚在 7 月初发布了 MCP Gateway 的多租户测试版，per-user token 隔离是其核心卖点，预计 Q3 正式版
2. **OAuth 2.1 PKCE 强制落地**：2026 年底所有 MCP 托管商将被要求默认启用 PKCE，目前多数还在用 authorization code + client secret 的 legacy 模式
3. **Sentry 修复后的 per-user token 成本报告**：Sentry 团队承诺在 8 月公开这次故障的完整 RCA 和修复后的 P99 latency 数据，可以关注他们的 engineering blog
4. **Notion API 2026-08 更新**：Notion 将在 8 月调整 OAuth token 刷新策略，每个 integration 的 refresh token 上限从 100 降到 25——如果你有大量 MCP 集成，需要提前规划 token 淘汰策略

## 参考资料

- [MCP Rate Limits: A Practical Guide for Agent Builders - Peliqan](https://peliqan.io/blog/mcp-rate-limits-guide/)
- [Rate Limiting in Virtual MCP Servers: Per-User, Tool & Tenant - Scalekit](https://www.scalekit.com/blog/rate-limiting-virtual-mcp-servers)
- [Sentry MCP Issue #844: Rate limiting after authentication](https://github.com/getsentry/sentry-mcp/issues/844)
- [Build vs. Buy: The Hidden Costs of Custom MCP Servers - Truto](https://truto.one/blog/build-vs-buy-the-hidden-costs-of-custom-mcp-servers/)
- [MCP Server Security Best Practices: 2026 Engineering Guide](https://www.digitalapplied.com/blog/mcp-server-security-best-practices-2026-engineering-guide)
- [Official Notion MCP Server - GitHub](https://github.com/makenotion/notion-mcp-server)
- [OAuth 2.1 with PKCE - MCP spec recommendation](https://www.ayautomate.com/blog/mcp-server-development-guide)
