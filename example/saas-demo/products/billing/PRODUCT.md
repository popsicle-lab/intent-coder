---
Last-Updated: 2026-05-13
Last-Decision-Ref: PDR-0001
Product: billing
Audience: ["PM", "客户成功", "财务", "AI Copilot"]
---

# Billing — 订阅计费产品

> 把用户的订阅意愿变成账单。Stripe 集成、三档套餐、续费 / 升降 / 取消、计费失败重试。

---

## What this product does

Billing 接住 auth 创建的 User，给他们提供订阅服务。

**核心定位**:

- **三档定价**：Free（永久免费）/ Pro（个人付费）/ Team（团队按 Seat 付费）—— 见 PDR-0001 § Decision
- **支付**：仅通过 Stripe，不自建支付通道
- **首次升级**：Free → Pro / Team 走 Stripe Checkout，成功后立即解锁权限
- **计费失败**：Stripe Dunning 自动重试 3 次（间隔 1/3/7 天）；3 次失败降级到 Free
- **升降档**：周期内升档立即生效（prorate 多扣差额）；周期内降档下个周期生效
- **团队 Seat**：Team 版按月折算 Seat 数；增减 Seat 实时 prorate
- **取消**：当前周期内保留访问；周期结束自动到 Free（不立即停用）

`Decision-Ref: PDR-0001`

---

## User Intents Catalog

| User Query | → Task | Journey Stage | Audience |
|---|---|---|---|
| "怎么从 Free 升到 Pro？" | [T-0001](tasks/onboarding/T-0001-upgrade-from-free-to-pro.md) | onboarding | Free 用户 |
| "我的下次扣费是什么时候？" | [T-0010](tasks/daily-ops/T-0010-view-current-invoice.md) | daily-ops | Pro/Team 用户 |
| "怎么换信用卡？" | [T-0011](tasks/daily-ops/T-0011-update-payment-method.md) | daily-ops | Pro/Team 用户 |
| "我的信用卡过期了，订阅怎么办？" | [T-0020](tasks/troubleshooting/T-0020-handle-payment-failure.md) | troubleshooting | Pro/Team 用户 |
| "我们团队要加 5 个人，怎么操作？" | [T-0030](tasks/admin/T-0030-manage-team-seats.md) | admin | Account Admin (Team) |
| "我不想续费了，怎么取消？" | [T-0040](tasks/lifecycle/T-0040-cancel-subscription-with-prorate.md) | lifecycle | Pro/Team 用户 |

---

## Intents Catalog

| Intent Layer | File | Owner Tasks |
|---|---|---|
| Acceptance | [`intents/acceptance.intent`](intents/acceptance.intent) | T-0001 ~ T-0040 |
| Invariants | [`intents/invariants.intent`](intents/invariants.intent) | single-active-subscription / no-double-charge / prorate-symmetric |
| Contracts | [`intents/contracts.intent`](intents/contracts.intent) | `[Awaiting ADR-XXXX]` |

---

## 跨 Product 依赖

| 来自 | 类型 | 用途 |
|---|---|---|
| `auth` | event: `user.signed_up` | 触发创建 Free 订阅 |
| `auth` | RPC: `verify_session(token)` | 每次计费操作前验证身份 |
| `auth` | RPC: `revoke_session_by_admin` | （间接）注销账号时清理 |

---

## Status

- **当前迭代**: MVP（PDR-0001 落地）
- **可见 task 数**: 6（覆盖 5 个旅程阶段）
- **未完成的关键依赖**:
  - Stripe API key 配置（运维）
  - Webhook 端点签名验证（待 ADR-XXXX）

---

## 不在本 product 范围

- ❌ 计费失败的实际催收（dunning email 模板交给 marketing）
- ❌ 财务对账报表（admin-console 范畴）
- ❌ 退款审核（admin-console + 人工 support）

`Decision-Ref: PDR-0001`
