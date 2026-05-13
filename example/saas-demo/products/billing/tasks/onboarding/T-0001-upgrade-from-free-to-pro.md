---
task_id: T-0001
slug: upgrade-from-free-to-pro
title: "我从 Free 升级到 Pro 并完成首次付款"
journey_stage: onboarding
audience: ["Free 用户"]
task_type: user-action
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 已完成 auth/T-0001 + auth/T-0002（注册 + 2FA）
  - 当前订阅为 Free
limits:
  - 单次只能升一档（Free→Pro 或 Free→Team），不能跨档
  - 升级流程在 Stripe Checkout 完成；本 SaaS 不接收明文卡号
related_intents:
  - acceptance.intent#T-0001-upgrade-effective-within-30s
  - invariants.intent#single-active-subscription
related_next_tasks:
  - T-0010
fact_cite: []
last_verified: null
---

# 我从 Free 升级到 Pro 并完成首次付款

> 你试用了 Free 版觉得不错，决定升级到 Pro。本 task 走完整 Stripe Checkout 流程。

## 本 task 可解答

- "怎么升级到 Pro？"
- "怎么付款？"
- "升级后多久生效？"
- "我能直接升到 Team 吗？"
- "本 SaaS 看得到我的卡号吗？"

## 前提与限制

- 必须已注册 + 启用 2FA（升级是敏感操作，要求强身份验证）
- 单次只能升一档；要从 Free → Team 也支持，但 UI 只暴露两个直跳按钮
- 卡号**永不**经过本 SaaS 服务器——全部走 Stripe Checkout 重定向

## 完成路径

1. **设置页 → 套餐** → 点击「升级到 Pro」按钮 → 弹出 plan 对比表
2. **二次确认月付 / 年付选项** → 跳转到 Stripe Checkout 页（独立域名）
3. **用户在 Stripe 页面输卡号 / 完成 3DS 验证**
4. **支付成功** → Stripe 回调 `/billing/webhook` 推送 `checkout.session.completed`
5. **本 SaaS 后端**校验 webhook 签名、更新该 Account 的 Subscription 状态为 `active(plan=pro)`
6. **用户被重定向回**本 SaaS，显示「升级成功，已解锁 Pro 功能」

> 期望耗时：升级**30 秒内**生效（从 Stripe 回调到本 SaaS 显示新状态）。
> 见 [`acceptance.intent#T-0001-upgrade-effective-within-30s`](../../intents/acceptance.intent)

## 可观察的成功标志

- 数据库该 Account 的 Subscription 行 `plan = pro`, `status = active`
- 用户邮箱收到 Stripe 的支付收据 + 本 SaaS 的「升级成功」邮件
- product 内 Pro 专属功能立即解锁

## Related Next Tasks

- **查发票**: [T-0010 查看本周期发票](../daily-ops/T-0010-view-current-invoice.md)
- **跨 product**: 本 task 是 [J-0001 § Stage 5-6](../../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md) 的核心

## 反向引用

- PDR: [PDR-0001 § Decision (D1)](../../decisions/pdr/PDR-0001-billing-strategy.md)
- 跨 product 旅程: [J-0001 § Stage 4-6](../../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)

## Charter Compliance

- [x] 用户原话标题 / frontmatter / 5 锚点 / 6 步 / ≤ 250 行
