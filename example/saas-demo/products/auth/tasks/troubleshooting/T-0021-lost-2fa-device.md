---
task_id: T-0021
slug: lost-2fa-device
title: "我换了手机后用 Recovery Code 重置 2FA"
journey_stage: troubleshooting
audience: ["end-user"]
task_type: recovery
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 你还能通过邮箱密码（或 Google OAuth）完成第一段身份验证
  - 你保存了启用 2FA 时的 **Recovery Code**（至少还剩 1 个）
limits:
  - Recovery Code 用一次即失效（10 个用完需走人工 support 重发）
  - 用 Recovery Code 后强制重新扫码绑定新 2FA 设备
related_intents:
  - acceptance.intent#T-0021-recovery-code-one-time-use
  - invariants.intent#2fa-required-after-onboarding
related_next_tasks:
  - T-0010
fact_cite: []
last_verified: null
---

# 我换了手机后用 Recovery Code 重置 2FA

> 你换了手机或丢了 Authenticator App，现在登不进来。本 task 教你用启用 2FA 时拿到的 Recovery Code 救回账号。

## 本 task 可解答

- "2FA 设备丢了怎么办？"
- "Recovery Code 是什么？我没保存怎么办？"
- "Recovery Code 能用几次？"
- "用 Recovery Code 后还要重新绑定 2FA 吗？"
- "10 个 Recovery Code 全用光了怎么办？"

## 前提与限制

- **必须**记得你的密码（或 Google 账号）—— Recovery Code **不**替代第一段身份验证
- **必须**还有至少一个未用过的 Recovery Code —— 全用光需联系 support 重发（本 demo 不覆盖）
- 本流程**强制**让你重新绑定新设备 —— 用完 Recovery Code 不会让你继续无 2FA 登录

## 完成路径

1. **登录页输入邮箱 + 密码** → 通过后进入 2FA 验证页
2. **点击「用 Recovery Code 替代」** → 输入一个未用过的 Recovery Code
3. **后端哈希比对 + 标记该 Code 为 used**
4. **强制进入「绑定新 2FA 设备」流程**（同 T-0002 步骤 1-4）
5. **用户扫码 + 输入新 6 位码** → 后端验证后激活新 TOTP secret
6. **签发 Session 凭证** → 跳到 product 主页面
7. **后端发送「2FA 设备已重置」告警邮件**

> 期望耗时：≤ 90 秒（含重新绑定）。Recovery Code 一次性使用约束见 [`acceptance.intent#T-0021-recovery-code-one-time-use`](../../intents/acceptance.intent)

## 可观察的成功标志

- 数据库中该 User 旧 `totp_secret` 被替换为新值
- 用过的 Recovery Code 在表里 `used_at` 字段非空
- 用户邮箱收到告警邮件
- 用户能用新设备生成的 TOTP 码继续 [T-0010 登录](../daily-ops/T-0010-login-with-email-password.md)

## Related Next Tasks

- **重置后**: 回到 [T-0010 邮箱密码 + 2FA 登录](../daily-ops/T-0010-login-with-email-password.md)
- **Recovery Code 全用光**: 联系 support（demo 暂未覆盖，应有 admin 流程通过 audit 审核后重发）
- **担心账号被入侵**: [T-0030 Account Admin 吊销所有 Session](../admin/T-0030-revoke-user-sessions.md)（你是 Admin 时）

## 反向引用

- PDR: [PDR-0001 § Decision (D2)](../../decisions/pdr/PDR-0001-auth-strategy.md)

## Charter Compliance

- [x] 标题用户原话句子
- [x] frontmatter 完整
- [x] 5 个 query 锚点
- [x] 7 步完成路径
- [x] 文件 ≤ 250 行
