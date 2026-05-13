---
task_id: T-0011
slug: update-payment-method
title: "我更换绑定的支付方式"
journey_stage: daily-ops
audience: ["Pro/Team 用户"]
task_type: user-action
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 当前订阅是 Pro 或 Team
limits:
  - 仅信用卡 / 借记卡（不支持电汇）
  - 更换瞬间生效；下次扣费用新卡
related_intents:
  - acceptance.intent#T-0011-payment-method-updated
  - invariants.intent#no-plaintext-card-number
related_next_tasks: []
fact_cite: []
last_verified: null
---

# 我更换绑定的支付方式

> 旧卡过期 / 你想换一张卡 / 团队换公司卡。本 task 走 Stripe SetupIntent，不影响当前 Subscription。

## 本 task 可解答

- "怎么换信用卡？"
- "换卡后立即生效吗？"
- "本 SaaS 看得到我的新卡号吗？"（不能）
- "换卡时需要重新付一次款吗？"（不需要）
- "我能保留多张卡吗？"

## 前提与限制

- 本 SaaS **不接收**明文卡号——更换流程走 Stripe Elements / Checkout，由 Stripe 收卡
- 当前 Subscription 不变；下次扣费用新卡
- 一个 Account 同一时刻只保留**一张 default payment method**（避免「不知道用哪张」）

## 完成路径

1. **设置页 → 计费 → 支付方式** → 点击「更换」
2. **后端调用 Stripe** 创建 SetupIntent → 返回 client secret
3. **前端 Stripe Elements** 收新卡号 → 完成 3DS（如适用）→ Stripe 创建新 PaymentMethod
4. **本 SaaS 后端**把该 PaymentMethod 绑定为该 Customer 的 default
5. **用户看到「已更换」提示**；下次扣费日不变

## 可观察的成功标志

- 数据库该 Customer 的 `default_payment_method_id` 已更新
- Stripe 端可查到新 PaymentMethod attached
- 本 SaaS 服务器内**无任何明文卡号**（见 `invariants.intent#no-plaintext-card-number`）

## Related Next Tasks

- **下次扣费失败**: [T-0020 处理扣费失败](../troubleshooting/T-0020-handle-payment-failure.md)

## 反向引用

- PDR: [PDR-0001 § Decision (D3)](../../decisions/pdr/PDR-0001-billing-strategy.md)

## Charter Compliance

- [x] 用户原话标题 / frontmatter / 5 锚点 / 5 步 / ≤ 250 行
