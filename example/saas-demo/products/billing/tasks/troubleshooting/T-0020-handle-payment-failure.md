---
task_id: T-0020
slug: handle-payment-failure
title: "我处理一次扣费失败"
journey_stage: troubleshooting
audience: ["Pro/Team 用户"]
task_type: recovery
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 收到「扣费失败」邮件 / 应用内通知
  - 当前订阅是 Pro 或 Team
limits:
  - Stripe Dunning 自动重试 3 次（1/3/7 天）
  - 3 次失败后订阅自动降级到 Free
related_intents:
  - acceptance.intent#T-0020-dunning-retry-schedule
  - acceptance.intent#T-0020-downgrade-on-3rd-failure
  - invariants.intent#no-double-charge
related_next_tasks:
  - T-0011
fact_cite: []
last_verified: null
---

# 我处理一次扣费失败

> 你的信用卡过期 / 余额不足 / 被发卡行风控。Stripe 已发邮件提示扣费失败。本 task 教你恢复订阅。

## 本 task 可解答

- "我的信用卡过期了订阅怎么办？"
- "扣费失败会自动重试吗？"
- "重试几次都失败会被停服吗？"
- "我能立即手动重试一次吗？"
- "停服后还能恢复吗？"

## 前提与限制

- Stripe 自动 Dunning：扣费失败后 **1 天 / 3 天 / 7 天** 各重试一次（共 4 次含原次）
- 第 3 次重试仍失败 → 订阅自动降级到 Free（数据保留，功能限制）
- 立即降级前会收到至少 3 封邮件提示
- 降级后随时可走 [T-0001](../onboarding/T-0001-upgrade-from-free-to-pro.md) 重新升级

## 完成路径

**用户主动恢复路径**（建议）:

1. **收到邮件后立即换卡**：走 [T-0011 更换支付方式](../daily-ops/T-0011-update-payment-method.md)
2. **本 SaaS 页面点「立即重试」按钮** → 后端调用 Stripe `pay_invoice` API
3. **Stripe 用新卡尝试扣费** → 成功则订阅状态恢复 `active`

**被动等待自动重试**:

1. Stripe 按 1/3/7 天间隔重试，每次失败发邮件
2. 第 3 次仍失败 → 订阅自动降级到 Free（功能受限但不删数据）
3. 用户随时可走 T-0001 重新升级

> Retry 时序见 [`acceptance.intent#T-0020-dunning-retry-schedule`](../../intents/acceptance.intent)

## 可观察的成功标志

- **恢复成功**：发票状态 `paid`，Subscription `active`，用户邮箱收到「续费成功」邮件
- **降级**：Subscription `plan=free`，product 内 Pro 功能进入只读模式
- 任何情况下**不会出现重复扣费**（见 `invariants.intent#no-double-charge`）

## Related Next Tasks

- **想换卡再试**: [T-0011 更换支付方式](../daily-ops/T-0011-update-payment-method.md)
- **降级后想恢复**: [T-0001 升级](../onboarding/T-0001-upgrade-from-free-to-pro.md)

## 反向引用

- PDR: [PDR-0001 § Decision (D4)](../../decisions/pdr/PDR-0001-billing-strategy.md)

## Charter Compliance

- [x] 用户原话标题 / frontmatter / 5 锚点 / ≤ 250 行
