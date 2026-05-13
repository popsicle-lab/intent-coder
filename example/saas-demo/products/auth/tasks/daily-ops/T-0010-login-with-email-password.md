---
task_id: T-0010
slug: login-with-email-password
title: "我用邮箱密码 + 2FA 登录"
journey_stage: daily-ops
audience: ["end-user"]
task_type: user-action
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 已完成 T-0001 注册 + T-0002 启用 2FA
  - 手边有 2FA 设备（同启用时）
limits:
  - 同一邮箱 1 小时内连续 5 次密码错误锁定 15 分钟
  - 同一 Session 凭证 30 天后强制过期
related_intents:
  - acceptance.intent#T-0010-login-p95-under-200ms
  - acceptance.intent#T-0010-failed-login-lockout
  - invariants.intent#session-isolation
related_next_tasks: []
fact_cite: []
last_verified: null
---

# 我用邮箱密码 + 2FA 登录

> 你已经注册过并启用 2FA。本 task 是最常见的登录路径——日常进入 SaaS 的标准动作。

## 本 task 可解答

- "怎么登录？"
- "登录要输 2FA 验证码吗？"
- "我输错密码会怎样？"
- "登录有效期多久？"
- "Session 凭证安全吗？"

## 前提与限制

- 邮箱已注册且 2FA 已启用——若没启用 2FA 见 [T-0002](../onboarding/T-0002-enable-2fa-after-signup.md)
- 同一邮箱 1 小时内 5 次密码错误 → 锁定 15 分钟（防爆破）
- 同一 IP 1 分钟内 20 次请求 → 临时限流（429）

## 完成路径

1. **进入登录页**，输入邮箱 + 密码
2. **后端校验**（bcrypt verify 密码）—— 失败时**不**透露「邮箱不存在」还是「密码错误」（统一错误：`邮箱或密码错误`，防用户枚举）
3. **校验通过** → 进入 2FA 二级验证页（不立刻签发 Session）
4. **用户输入 6 位 TOTP 码** → 后端 verify（容忍前后 30s 时间窗）
5. **签发 Session 凭证**：JWT + 服务端会话表双写；绑定本次 IP + UA
6. **跳转到 product 主页面** → 任何后续请求都带 Session 凭证（HTTP-Only Cookie）

> 期望耗时：**P95 < 200ms**（不含用户输入时间）。见 [`acceptance.intent#T-0010-login-p95-under-200ms`](../../intents/acceptance.intent)

## 可观察的成功标志

- 用户浏览器拿到 HTTP-Only Cookie `session_token`
- 数据库 `sessions` 表新增一行（含 `user_id`, `ip`, `ua`, `expires_at`）
- 任何受保护 API 用此 token 都能正常返回 200

## Related Next Tasks

- **密码错误锁定**: [T-0020 通过邮箱恢复账号](../troubleshooting/T-0020-recover-account-via-email.md)
- **2FA 设备丢失**: [T-0021 用 Recovery Code 重置 2FA](../troubleshooting/T-0021-lost-2fa-device.md)
- **想用 OAuth 替代**: [T-0011 Google 一键登录](T-0011-login-with-google-oauth.md)

## 反向引用

- PDR: [PDR-0001 § Decision (D1)](../../decisions/pdr/PDR-0001-auth-strategy.md)
- 跨 product 旅程: [J-0001 § Stage 3](../../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)

## Charter Compliance

- [x] 标题用户原话句子（第一人称）
- [x] frontmatter 完整
- [x] 5 个 query 锚点
- [x] 6 步完成路径
- [x] 文件 ≤ 250 行
