---
task_id: T-0040
slug: delete-account
title: "我注销账号并等 7 天冷却期"
journey_stage: lifecycle
audience: ["end-user", "Account Admin"]
task_type: lifecycle
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 你已登录（能完成 T-0010 / T-0011）
  - 你已结清所有未付 Invoice（见 billing/T-0040 取消订阅）
limits:
  - 7 天冷却期内可登录撤销注销请求
  - 冷却期满后所有 PII 被删除（不可恢复）
  - Audit Log 保留 90 天后才永久删除
related_intents:
  - acceptance.intent#T-0040-deletion-7d-cooldown
  - invariants.intent#audit-log-retention
related_next_tasks: []
fact_cite: []
last_verified: null
---

# 我注销账号并等 7 天冷却期

> 你决定不再使用本 SaaS。注销不是立即删除——有 7 天冷却期让你反悔。

## 本 task 可解答

- "怎么注销账号？"
- "注销后还能恢复吗？"
- "7 天冷却期内能登录吗？"
- "我的数据多久被彻底删除？"
- "我还有未付的订阅怎么办？"

## 前提与限制

- 你必须已经**结清**所有未付 Invoice——如果还有订阅，先去 billing/T-0040 取消订阅
- 冷却期内**仍可正常登录**——登录页会显著提示「您的账号将在 X 天后删除，[撤销]」
- 冷却期满后**所有 PII 永久删除**：User 表、Session 表、OAuth Identity 表
- Audit Log 按 GDPR / SOC2 要求**额外保留 90 天**（含你的操作历史）再永久删除（见 `invariants.intent#audit-log-retention`）

## 完成路径

1. **进入账号设置 → 「注销账号」区域** → 点击「申请注销」按钮
2. **后端检查**是否有未结清 Invoice：
   - 有 → 跳到 billing/T-0040 让你先取消订阅 → 流程暂停
   - 无 → 进入第 3 步
3. **要求二次输入密码 + 6 位 TOTP 码**（敏感操作再认证）+ 输入文字 "DELETE MY ACCOUNT" 确认
4. **后端把 User 状态标为 `pending_deletion`** → 写入 `deletion_scheduled_at` = now + 7 天
5. **立即吊销所有现有 Session**（防注销请求是攻击者发的）+ 发邮件通知本人 + 邮件含「撤销注销」链接
6. **冷却期内**用户登录会看到红色横幅「您的账号将在 X 天后删除，点此撤销」
7. **冷却期满**：后台 job 永久删除该 User 的所有 PII；Audit Log 保留 90 天后彻底删除

> 7 天冷却期约束见 [`acceptance.intent#T-0040-deletion-7d-cooldown`](../../intents/acceptance.intent)

## 可观察的成功标志

- 数据库该 User 状态变为 `pending_deletion`，`deletion_scheduled_at` 非空
- 用户邮箱收到「注销已申请，X 月 X 日生效」邮件
- 跨 product：billing 自动取消该 User 任何 active Subscription（J-0001 逆向流程）
- 冷却期满后 User 表中该行被删除（或归档到只读冷存储）

## Related Next Tasks

- **冷却期内反悔**: 登录后点横幅「撤销注销」即可（本 demo 暂未单列为 task，操作简单）
- **billing 收尾**: billing/T-0040 取消订阅
- **跨 product 影响**: 所有 product 都拿不到该 user_id 的有效 Session

## 反向引用

- PDR: [PDR-0001 § Decision (D6)](../../decisions/pdr/PDR-0001-auth-strategy.md)
- Invariant: `invariants.intent#audit-log-retention`

## Charter Compliance

- [x] 标题用户原话句子
- [x] frontmatter 完整
- [x] 5 个 query 锚点
- [x] 7 步完成路径
- [x] 文件 ≤ 250 行
