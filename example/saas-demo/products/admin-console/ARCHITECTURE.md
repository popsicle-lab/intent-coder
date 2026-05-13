---
Last-Updated: 2026-05-13
Last-Decision-Ref: project-init bootstrap
Product: admin-console
Status: scaffold-only
---

# Admin Console — 架构 [TBD]

> ⚠️ scaffold-only。`[TBD: needs ADR + PDR]`

## Module 划分

`[TBD]`

## 关键技术选型

`[TBD]`

## 跨 product 集成

| 对方 | 类型 | 状态 |
|---|---|---|
| `auth` | RPC verify_session / revoke_session_by_admin / Audit Log 订阅 | `[TBD]` |
| `billing` | RPC query_subscription | `[TBD]` |

## 关键 Invariant

`[TBD]`

可能需要：跨 Account 查询时**强制**写 Audit Log（参考 auth/invariants.intent#audit-log-retention）
