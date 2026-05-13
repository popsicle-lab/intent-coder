---
Last-Updated: 2026-05-13
Last-Decision-Ref: PDR-0001
Product: billing
Audience: ["工程师", "AI agent"]
---

# Billing — 架构

> 多处 `[TBD: Awaiting ADR-XXXX]` —— 同 auth/ARCHITECTURE.md，本 demo 故意不装 arch-debate。

---

## Module 划分

| 模块 | 职责 | 状态 |
|---|---|---|
| `billing-api` | HTTP 入口（升降档 / 查发票 / 改支付方式）| `[TBD: Awaiting ADR-XXXX]` |
| `billing-core` | 业务逻辑（套餐切换 / Seat 计算 / prorate）| `[TBD: Awaiting ADR-XXXX]` |
| `billing-stripe` | Stripe 客户端封装 + Webhook 处理 | `[TBD: Awaiting ADR-XXXX]` |
| `billing-store` | 持久化（Subscription / Invoice / PaymentMethod 表）| `[TBD: Awaiting ADR-XXXX]` |

---

## 关键技术选型

| 选型 | 决策 | 来源 |
|---|---|---|
| 支付通道 | Stripe（唯一）| PDR-0001 |
| Webhook 签名验证 | `[TBD: Awaiting ADR-XXXX]` | —— |
| Subscription 存储 | `[TBD: Awaiting ADR-XXXX]` | —— |
| Prorate 计算精度 | `[TBD: Awaiting ADR-XXXX]` —— 货币最小单位 (分) | —— |

---

## 跨 product 集成点

详见 `intents/contracts.intent`。auth → billing 单向依赖；billing 永远是消费方。

---

## 性能与可用性目标

| 指标 | 目标 | Decision-Ref |
|---|---|---|
| 升级 API P95（含 Stripe 调用）| < 2s | PDR-0001 |
| Webhook 处理 P95 | < 500ms | PDR-0001 |
| 升降档生效延迟 | < 30s（含 Stripe 异步通知）| PDR-0001 |

---

## 关键 Invariant

| Invariant | 来源 |
|---|---|
| `single-active-subscription` —— 每个 Account 同一时刻有且仅有一个 active sub | PDR-0001 |
| `no-double-charge` —— 同一计费周期不会向 Stripe 发起重复 charge | PDR-0001 |
| `prorate-symmetric` —— 周期内升档 prorate 多扣 = 同周期降档 prorate 退还（金额一致）| PDR-0001 |

---

`Decision-Ref: PDR-0001`
