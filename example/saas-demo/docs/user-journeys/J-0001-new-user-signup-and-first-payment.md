---
journey_id: J-0001
slug: new-user-signup-and-first-payment
title: "我注册账号、启用 2FA、首次登录、并完成第一次付费"
involves_products: ["auth", "billing"]
audience: ["new-user"]
decision_refs:
  - auth/PDR-0001
  - billing/PDR-0001
last_updated: 2026-05-13
status: Accepted
---

# 我注册账号、启用 2FA、首次登录、并完成第一次付费

> 一个新用户从打开网站到付款成功的**完整旅程**。
> 跨 product：auth (Stage 1-3) + billing (Stage 4-6)。
> 顺序对体验关键——Stage 顺序变了 onboarding 漏斗就崩了。

---

## 本 Journey 可解答

- "从注册到付款一共要做几步？"
- "我能先付款再启用 2FA 吗？"（**不能**——见 Stage 2 / 5 约束）
- "我注册了但没付款，能用 Free 版多久？"
- "如果我注册一半放弃了，下次还能继续吗？"
- "团队版第一次怎么开通？"

---

## Stages

### Stage 1: 用邮箱注册账号（auth）

**Task**: [auth/T-0001 我用邮箱注册一个新账号](../../products/auth/tasks/onboarding/T-0001-first-time-signup-with-email.md)

- 输入邮箱 + 密码 + Account 名称
- 邮箱收验证邮件 → 点链接验证
- 期望耗时 ≤ 90 秒

**关键约束**: 邮箱必须全局未注册（`invariants.intent#email-uniqueness`）

---

### Stage 2: 强制启用 2FA（auth）

**Task**: [auth/T-0002 我注册当天扫码启用 2FA](../../products/auth/tasks/onboarding/T-0002-enable-2fa-after-signup.md)

- 扫码绑定 Authenticator App
- **保存 10 个 Recovery Code**（只显示一次！）
- 期望耗时 ≤ 60 秒

**关键约束**:
- 用户**不能跳过本 stage** 直接到 Stage 3（`invariants.intent#2fa-required-after-onboarding`）
- 这是 PDR-0001/D2 的核心取舍——B2B 不接受 2FA 可选

---

### Stage 3: 首次登录（auth）

**Task**: [auth/T-0010 我用邮箱密码 + 2FA 登录](../../products/auth/tasks/daily-ops/T-0010-login-with-email-password.md)

- 输入邮箱 + 密码 → 通过密码校验
- 输入 6 位 TOTP 码 → 通过 2FA 校验
- 拿到 Session 凭证
- 期望耗时 P95 < 200ms

**Stage 3 完成 = auth onboarding 完成；自动跳到 product 主界面**

---

### Stage 4: 自动创建 Free 订阅（billing - 系统侧）

**Task**: 不是用户主动 task，是 billing 监听 auth 的 `user.signed_up` 事件后自动执行。

- billing 收到 `user.signed_up` 事件（contract: `consume_user_signed_up_event`）
- 创建该 Account 的 Subscription（plan=free, status=active）
- 用户在 Stage 3 跳到主界面时，已经是 Free 用户

**关键约束**:
- 同一事件必须 idempotent（`billing/invariants.intent#single-active-subscription`）
- 用户**感知不到** Stage 4——这是后台异步动作

---

### Stage 5: 试用 Free，主动升级到 Pro（用户决策）

**用户决策点**: 用户用了 N 天 Free 版（典型 N=3-14），决定升级。

**Task**: [billing/T-0001 我从 Free 升级到 Pro 并完成首次付款](../../products/billing/tasks/onboarding/T-0001-upgrade-from-free-to-pro.md)

- 设置页 → 套餐 → 升级到 Pro
- 跳转 Stripe Checkout → 输卡 + 3DS
- Stripe 回调 → 本 SaaS 解锁 Pro

**关键约束**:
- 必须先完成 Stage 2（启用 2FA）才能升级（升级是敏感操作）
- 卡号永不入本 SaaS（`billing/invariants.intent#no-plaintext-card-number`）

---

### Stage 6: 升级生效 + 解锁 Pro 功能（billing）

- Stripe webhook 推送 `checkout.session.completed`
- 本 SaaS 在 30 秒内把 Subscription `plan=pro, status=active`
- 用户在 product 内立即看到 Pro 功能解锁

**完整 Journey 完成**

---

## 整体期望

| 指标 | 目标 |
|---|---|
| Stage 1-3 完成率（注册到首次登录）| > 95% |
| Stage 3 → Stage 5 转化率（Free → Pro）| 行业基线 2-5% |
| 整个 Journey 中位耗时（Stage 1 → 3）| ≤ 3 分钟 |
| 整个 Journey 中位耗时（Stage 1 → 6 含决策期）| 3-14 天 |

---

## 漏斗逆向：可能的脱离点

| 在哪一步流失 | 可能原因 | 补救 task |
|---|---|---|
| Stage 1 邮箱验证 | 邮件没到 / 嫌麻烦 | [auth/T-0020](../../products/auth/tasks/troubleshooting/T-0020-recover-account-via-email.md) |
| Stage 2 2FA | 不想装 Authenticator | 改善 UX 文案（不放本 demo） |
| Stage 3 登录失败 | 密码忘了 / 2FA 设备问题 | [auth/T-0020](../../products/auth/tasks/troubleshooting/T-0020-recover-account-via-email.md) / [auth/T-0021](../../products/auth/tasks/troubleshooting/T-0021-lost-2fa-device.md) |
| Stage 5 升级前犹豫 | 价格 / 功能不足 | 营销 / 客户成功（不放本 demo） |
| Stage 5 支付失败 | 卡过期 / 风控 | [billing/T-0020](../../products/billing/tasks/troubleshooting/T-0020-handle-payment-failure.md) |

---

## Decision-Ref

- auth: [PDR-0001](../../products/auth/decisions/pdr/PDR-0001-auth-strategy.md) — Stage 1/2/3 的策略决定
- billing: [PDR-0001](../../products/billing/decisions/pdr/PDR-0001-billing-strategy.md) — Stage 4/5/6 的策略决定
