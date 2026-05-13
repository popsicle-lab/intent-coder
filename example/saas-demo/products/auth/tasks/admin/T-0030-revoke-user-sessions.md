---
task_id: T-0030
slug: revoke-user-sessions
title: "我作为 Account Admin 吊销某个员工的所有 Session"
journey_stage: admin
audience: ["Account Admin"]
task_type: management
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 你是该 Account 的 Admin（在 Team 版下）
  - 目标 User 属于你管理的 Account
limits:
  - 你**不能**吊销自己的 Session（防误操作把自己锁外面）
  - 操作进 Audit Log，不可撤销
related_intents:
  - acceptance.intent#T-0030-revoke-takes-effect-in-5s
  - invariants.intent#session-isolation
related_next_tasks: []
fact_cite: []
last_verified: null
---

# 我作为 Account Admin 吊销某个员工的所有 Session

> 你管理一个 Team 版 Account。某个员工离职 / 设备丢失 / 怀疑被入侵，你需要把他在所有设备上踢下线。

## 本 task 可解答

- "怎么把员工踢下线？"
- "离职员工的会话怎么处理？"
- "吊销 Session 多久生效？"
- "我能吊销自己的 Session 吗？"
- "吊销有审计记录吗？"

## 前提与限制

- 你是该 Account 的 **Account Admin**（不是本 SaaS 内部 Admin）
- 目标 User 必须属于你的 Account——跨 Account 吊销由本 SaaS 内部 Admin 通过 admin-console 做（本 demo 暂未覆盖）
- **不能吊销自己**——防止误操作把自己锁在外面（要换密码请走 [T-0020](../troubleshooting/T-0020-recover-account-via-email.md)）

## 完成路径

1. **进入 Account 设置页 → 成员列表** → 找到目标 User
2. **点击该 User 行的「吊销所有 Session」按钮** → 弹出确认框，要求二次输入你的密码（敏感操作再认证）
3. **输入密码 + 确认** → 后端验证 Admin 身份 + 校验目标 User 同 Account
4. **后端把目标 User 所有 active Session 标记为 `revoked_at`** → 目标 User 任何 product 的下一次 API 请求返回 401
5. **后端写 Audit Log**（包含：操作人 / 目标 User / 时间 / 来源 IP）
6. **可选**：发送邮件通知目标 User「您的 Session 已被 Admin 吊销」

> 期望耗时：吊销**5 秒内**对目标 User 全网生效。见 [`acceptance.intent#T-0030-revoke-takes-effect-in-5s`](../../intents/acceptance.intent)

## 可观察的成功标志

- 数据库目标 User 所有 `sessions` 行 `revoked_at` 非空
- 目标 User 在任何 product（auth / billing / admin-console）下一次 API 请求返回 401
- Audit Log 出现该操作记录
- （若选了通知）目标 User 邮箱收到告警

## Related Next Tasks

- **跨 Account 吊销**: admin-console product（本 demo 不覆盖）
- **想审计历史**: admin-console § Audit Log 查询（本 demo 不覆盖）

## 反向引用

- PDR: [PDR-0001 § Decision (D5)](../../decisions/pdr/PDR-0001-auth-strategy.md)

## Charter Compliance

- [x] 标题用户原话句子
- [x] frontmatter 完整
- [x] 5 个 query 锚点
- [x] 6 步完成路径
- [x] 文件 ≤ 250 行
