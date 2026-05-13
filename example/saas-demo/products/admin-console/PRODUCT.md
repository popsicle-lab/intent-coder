---
Last-Updated: 2026-05-13
Last-Decision-Ref: project-init bootstrap
Product: admin-console
Audience: ["内部 Admin", "客户支持"]
Status: scaffold-only
---

# Admin Console — 管理后台 [TBD]

> ⚠️ **Status**: scaffold-only —— 仅占位，所有内容 `[TBD: needs PDR]`
>
> 本 product 不是首切片，按 IDD bootstrap 序列，首切片（auth）跑通后才会回头跑
> admin-console 的 product-debate + prd-writer。

---

## What this product does (未来)

`[TBD]` 内部 Admin 跨 Account 查询用户 / 订阅 / 审计 Log / 处理 support 工单。

---

## User Intents Catalog

`[TBD: needs PDR]`

| User Query | → Task | Journey Stage | Audience |
|---|---|---|---|
| `[TBD]` | —— | —— | —— |

---

## Intents Catalog

`[TBD: 等首批 PDR 落地]`

| Intent Layer | File | Owner Tasks |
|---|---|---|
| Acceptance | [`intents/acceptance.intent`](intents/acceptance.intent) | （空）|
| Invariants | [`intents/invariants.intent`](intents/invariants.intent) | （空）|
| Contracts | [`intents/contracts.intent`](intents/contracts.intent) | （空）|

---

## 跨 Product 依赖（预期）

| 来自 | 类型 | 用途 |
|---|---|---|
| `auth` | RPC: `verify_session` | 内部 Admin 也要登录 |
| `auth` | RPC: `revoke_session_by_admin` | 跨 Account 吊销 |
| `auth` | event: Audit Log | 审计查询的核心数据源 |
| `billing` | RPC: `query_subscription` | 客户咨询订阅状态 |

---

## Status

- **当前迭代**: 未启动（scaffold-only）
- **可见 task 数**: 0
- **未完成**: 等首切片 auth 完成后启动 product-debate
