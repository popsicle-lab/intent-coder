---
task_id: T-0040
slug: cancel-subscription-with-prorate
title: "我取消订阅并按当前周期结束自动降级"
journey_stage: lifecycle
audience: ["Pro/Team 用户"]
task_type: lifecycle
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 当前订阅是 Pro 或 Team
  - 你是 Pro 用户本人 或 Team Account Admin
limits:
  - 取消**不立即停服**——当前周期结束自动降级到 Free
  - 不退还本周期已扣款（除非客服 SLA 例外）
related_intents:
  - acceptance.intent#T-0040-cancel-effective-at-period-end
  - invariants.intent#no-double-charge
related_next_tasks: []
fact_cite: []
last_verified: null
---

# 我取消订阅并按当前周期结束自动降级

> 你决定不再续费。本 task 不立即停服——继续用到本周期结束，到期自动降到 Free。

## 本 task 可解答

- "怎么取消订阅？"
- "取消后立即停服吗？"
- "已经付的钱会退吗？"
- "周期内能反悔吗？"
- "降级后我的数据还在吗？"

## 前提与限制

- 取消**不退款**（除非走人工 support 走 SLA 例外）
- 取消后到本周期结束前**仍可继续使用** Pro/Team 功能
- 周期结束后自动降级到 Free —— 数据**全部保留**，仅功能受限
- 周期内**可反悔**：进设置页点「恢复订阅」即可

## 完成路径

1. **设置页 → 计费 → 取消订阅** → 弹出挽留页（可选填取消原因）
2. **二次确认 + 2FA 校验**（敏感操作再认证）
3. **后端调用 Stripe** `subscription.update(cancel_at_period_end=true)` —— **不**立即 cancel
4. **本 SaaS** 更新 Subscription `will_cancel_at = current_period_end`，发邮件确认
5. **周期内**：用户登录看到顶部横幅「订阅将在 X 月 X 日到期，[恢复订阅]」
6. **周期到期**：Stripe 自动 cancel；本 SaaS 把 plan 切到 Free；发降级邮件

## 可观察的成功标志

- 取消后：Stripe Subscription 状态 `active`，但 `cancel_at_period_end = true`
- 周期到期：Subscription `status = canceled`；本 SaaS plan = Free
- 用户**无任何**额外扣费（见 `invariants.intent#no-double-charge`）
- 全部数据保留（功能受限）

## Related Next Tasks

- **想反悔**: 周期内进设置页点「恢复订阅」（本 demo 暂未单列）
- **彻底删除账号**: auth/T-0040 注销账号（注销前必须先取消订阅）

## 反向引用

- PDR: [PDR-0001 § Decision (D6)](../../decisions/pdr/PDR-0001-billing-strategy.md)
- 跨 product: 是 auth/T-0040 的前置（注销账号必须先取消订阅）

## Charter Compliance

- [x] 用户原话标题 / frontmatter / 5 锚点 / 6 步 / ≤ 250 行
