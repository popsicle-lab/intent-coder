---
task_id: T-0020
slug: recover-account-via-email
title: "我忘记密码后通过邮箱验证恢复"
journey_stage: troubleshooting
audience: ["end-user"]
task_type: recovery
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 你能访问当初注册时的邮箱
  - 不需要 2FA 设备——但密码重置后**仍需**用 2FA 完成登录
limits:
  - 同一邮箱 1 小时内最多 3 次重置请求
  - 重置链接有效期 30 分钟，一次性使用
related_intents:
  - acceptance.intent#T-0020-recovery-email-arrives-in-60s
  - invariants.intent#no-plaintext-password
related_next_tasks:
  - T-0010  # 重置后回到正常登录流程
fact_cite: []
last_verified: null
---

# 我忘记密码后通过邮箱验证恢复

> 你忘记了登录密码。本 task 教你用邮箱验证流程**只**重置密码——**不**触动 2FA。

## 本 task 可解答

- "忘记密码怎么办？"
- "重置密码邮件多久到？"
- "重置密码后还要 2FA 吗？"
- "我能通过邮箱直接重置 2FA 吗？"（**不能**——见下文）
- "重置后旧 Session 会失效吗？"

## 前提与限制

- 你**仍能访问**注册时的邮箱——如果邮箱也丢了，去 [T-0021](T-0021-lost-2fa-device.md)（其实那是 2FA 设备丢失的，邮箱丢失是更严重的场景，本 demo 暂未覆盖，需要人工 support）
- 本流程**只重置密码**，**不重置 2FA**——这是关键的安全设计：邮箱被入侵也不能绕过 2FA
- 同一邮箱 1 小时内 3 次请求上限（防滥用）

## 完成路径

1. **登录页点「忘记密码」** → 输入邮箱
2. **后端生成一次性 reset_token**（30 分钟有效，一次性）→ 发邮件含验证链接
3. **用户打开邮件点链接** → 浏览器跳回本 SaaS 的密码重置页
4. **用户输入新密码两次**（前端做强度校验，规则同 T-0001）
5. **后端验证 reset_token + 更新密码哈希**（bcrypt cost=12）→ **立即吊销该 User 所有现存 Session**（防攻击者已登录）
6. **用户被跳到登录页** → 用新密码 + 2FA 走 [T-0010 登录](../daily-ops/T-0010-login-with-email-password.md)

> 期望耗时：邮件**60 秒内**到达。见 [`acceptance.intent#T-0020-recovery-email-arrives-in-60s`](../../intents/acceptance.intent)

## 可观察的成功标志

- 用户邮箱收到「密码重置请求」邮件
- 数据库该 User 的密码哈希已更新；所有旧 Session 行被标记为 `revoked_at`
- 用户能用新密码 + 2FA 成功登录

## Related Next Tasks

- **重置后下一步**: [T-0010 邮箱密码 + 2FA 登录](../daily-ops/T-0010-login-with-email-password.md)
- **2FA 也丢了**: [T-0021 用 Recovery Code 重置 2FA](T-0021-lost-2fa-device.md)（独立流程）
- **邮箱也丢了**: 联系人工 support（本 demo 不覆盖）

## 反向引用

- PDR: [PDR-0001 § Decision (D4)](../../decisions/pdr/PDR-0001-auth-strategy.md)

## Charter Compliance

- [x] 标题用户原话句子
- [x] frontmatter 完整
- [x] 5 个 query 锚点
- [x] 6 步完成路径
- [x] 文件 ≤ 250 行
