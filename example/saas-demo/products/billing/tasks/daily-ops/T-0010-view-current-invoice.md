---
task_id: T-0010
slug: view-current-invoice
title: "我查看本周期的发票和下次扣费时间"
journey_stage: daily-ops
audience: ["Pro/Team 用户"]
task_type: user-action
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 当前订阅是 Pro 或 Team（Free 用户没有发票）
limits:
  - 仅展示最近 24 个月的发票（更早的可联系 support）
related_intents:
  - acceptance.intent#T-0010-invoice-list-correct
related_next_tasks: []
fact_cite: []
last_verified: null
---

# 我查看本周期的发票和下次扣费时间

> 你想知道这个月被扣了多少 / 下个月什么时候续费 / 历史发票去哪下载。

## 本 task 可解答

- "我的下次扣费是什么时候？"
- "我能下载本月发票吗？"
- "为什么这个月扣了 XX 元？和上个月不一样？"
- "团队 Seat 调整的差额怎么显示？"
- "历史发票能看多久？"

## 前提与限制

- 必须是 Pro 或 Team 订阅—— Free 用户的此页面显示「升级 Pro 后可查看发票」
- 仅展示最近 24 个月的发票

## 完成路径

1. **设置页 → 计费 → 发票** → 加载发票列表
2. **后端从本地缓存读取最近发票** —— 缓存来自 Stripe webhook 推送的 `invoice.paid` / `invoice.payment_failed` 事件
3. **页面展示**：当前周期 / 下次扣费日 / 历史发票表格（金额、状态、PDF 下载链接）
4. **用户点击 PDF 下载** → 走 Stripe hosted invoice URL
5. **若发票数据 > 1 小时未刷新** → 后台触发 Stripe API 拉取最新（异步）

## 可观察的成功标志

- 发票金额与 Stripe 端数据一致
- 下次扣费日 = 当前 Subscription 的 `current_period_end`

## Related Next Tasks

- **金额不对**: [T-0020 处理扣费失败](../troubleshooting/T-0020-handle-payment-failure.md)
- **换卡**: [T-0011 更换支付方式](T-0011-update-payment-method.md)

## 反向引用

- PDR: [PDR-0001 § Decision (D2)](../../decisions/pdr/PDR-0001-billing-strategy.md)

## Charter Compliance

- [x] 用户原话标题 / frontmatter / 5 锚点 / 5 步 / ≤ 250 行
