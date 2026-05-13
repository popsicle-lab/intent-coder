---
task_id: T-0030
slug: manage-team-seats
title: "我作为 Team Admin 增减 Seat"
journey_stage: admin
audience: ["Account Admin (Team)"]
task_type: management
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 你是 Team 订阅的 Account Admin
  - 当前订阅状态为 active
limits:
  - 单次增减不超过 ±20 Seat（防误操作）
  - Seat 调整按当前周期剩余天数 prorate
related_intents:
  - acceptance.intent#T-0030-seat-prorate-correct
  - invariants.intent#prorate-symmetric
related_next_tasks:
  - T-0010
fact_cite: []
last_verified: null
---

# 我作为 Team Admin 增减 Seat

> 你管理一个 Team 版 Account。今天新员工入职 / 老员工离职，你需要调整 Seat 数。

## 本 task 可解答

- "我们团队要加 5 个人怎么操作？"
- "Seat 增减立即扣 / 退款吗？"
- "周期内调整怎么 prorate？"
- "Seat 数能小于当前已加入的人数吗？"
- "Seat 调整需要重新付款吗？"

## 前提与限制

- 仅 Account Admin 可操作（普通 Team 成员不可见此入口）
- 单次调整 ±20 上限——超过需联系 support
- **Seat 不能低于当前已加入的人数**——要先把多余成员移除，才能减 Seat

## 完成路径

1. **设置页 → Team** → 看到当前 Seat 数 / 已用 / 剩余
2. **点击「调整 Seat」** → 输入新 Seat 数（增或减）
3. **页面预览 prorate 金额**：增 Seat 立即扣差额；减 Seat 退还差额到余额（下个月抵扣）
4. **二次确认 + 2FA 校验**（同 auth/T-0010 的敏感操作再认证）
5. **后端调用 Stripe** 更新 Subscription quantity → Stripe 立即生成 prorate Invoice
6. **本 SaaS** 更新 Seat 上限 + 通知 Account 所有成员 + 写 Audit Log

> Prorate 对称性见 [`acceptance.intent#T-0030-seat-prorate-correct`](../../intents/acceptance.intent)
> 与 [`invariants.intent#prorate-symmetric`](../../intents/invariants.intent)

## 可观察的成功标志

- Subscription quantity 已更新
- Stripe 生成 prorate Invoice（金额可在 [T-0010](../daily-ops/T-0010-view-current-invoice.md) 查到）
- Account 所有成员收到「Seat 已调整」通知邮件
- Audit Log 含本次操作

## Related Next Tasks

- **查 prorate 金额**: [T-0010 查看本周期发票](../daily-ops/T-0010-view-current-invoice.md)
- **减员**: 先去 auth/admin 区移除 User，再回本 task 减 Seat

## 反向引用

- PDR: [PDR-0001 § Decision (D5)](../../decisions/pdr/PDR-0001-billing-strategy.md)

## Charter Compliance

- [x] 用户原话标题 / frontmatter / 5 锚点 / 6 步 / ≤ 250 行
