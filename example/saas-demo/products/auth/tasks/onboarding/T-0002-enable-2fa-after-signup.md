---
task_id: T-0002
slug: enable-2fa-after-signup
title: "我注册当天扫码启用 2FA"
journey_stage: onboarding
audience: ["new-user"]
task_type: user-action
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 已完成 T-0001 邮箱注册并验证
  - 手边有支持 TOTP 的 App（Google Authenticator / Authy / 1Password / iCloud 钥匙串）
limits:
  - 2FA 启用是**强制**步骤，不能跳过
  - 启用后 Recovery Code 仅显示一次，需当场保存
related_intents:
  - acceptance.intent#T-0002-2fa-recovery-codes-shown-once
  - invariants.intent#2fa-required-after-onboarding
related_next_tasks:
  - T-0010  # 启用 2FA 后才能正式登录
fact_cite: []
last_verified: null
---

# 我注册当天扫码启用 2FA

> 你刚验证完邮箱，被自动带到 2FA 启用页。本 task 是 onboarding 的**强制最后一步**——本 SaaS 不接受 2FA 可选。

## 本 task 可解答

- "为什么必须扫码 2FA？"
- "我能跳过 2FA 吗？"
- "TOTP 是什么？"
- "Recovery Code 是什么？要保存吗？"
- "我没装 Authenticator App 怎么办？"

## 前提与限制

- **强制**：你的账号在没启用 2FA 之前**不能登录任何 product**（包括 billing / admin-console）
- Recovery Code（10 个一次性备用码）**只在本 task 里显示一次**——关掉页面就再也看不到（见 [T-0021 设备丢失](../troubleshooting/T-0021-lost-2fa-device.md)）
- 如果你坚持不想要 2FA，你**不适合**本 SaaS（这是 PDR-0001 § Decision 的明确取舍——客户群体是企业/团队用户）

## 完成路径

1. **进入 2FA 启用页**（注册流程自动跳转）—— 看到一个 QR 码 + 一段 16 字符的 secret
2. **用 Authenticator App 扫描 QR 码**（或手动输入 secret）—— App 中出现 6 位动态码
3. **在页面输入当前 6 位码** → 后端验证一致后激活 2FA
4. **页面显示 10 个 Recovery Code**（一次性备用码）—— 用户必须**当场保存**（提供「下载 .txt」按钮）
5. **用户点击「我已保存」** → 注册流程结束，跳到 product 主界面
6. **后端发送「2FA 已启用」邮件**作为审计提示

> 期望耗时：**≤ 60 秒**。Recovery Code 必须在用户确认前**只显示一次**。
> 见 [`acceptance.intent#T-0002-2fa-recovery-codes-shown-once`](../../intents/acceptance.intent)

## 可观察的成功标志

- 数据库中该 User 的 `totp_secret` 字段非空、`totp_enabled_at` 已写入时间戳
- 该 User 有 10 条 Recovery Code 哈希记录（明文永不存储）
- 用户邮箱收到「2FA 已启用」通知邮件
- 用户被跳转到 product 主界面（不再回到注册流程）

## Related Next Tasks

- **必经下一步**: 用户的后续任何登录都走 [T-0010 邮箱密码 + 2FA 登录](../daily-ops/T-0010-login-with-email-password.md)
- **如果 2FA 设备丢了**: [T-0021 用 Recovery Code 重置 2FA](../troubleshooting/T-0021-lost-2fa-device.md)
- **跨 product**: 启用 2FA 后才允许跳到 billing 完成 [J-0001 § Stage 4 首次付费](../../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)

## 反向引用

- 跨 product 旅程: [J-0001 § Stage 2](../../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)
- PDR: [PDR-0001 § Decision (D2)](../../decisions/pdr/PDR-0001-auth-strategy.md)
- Invariant: `invariants.intent#2fa-required-after-onboarding`

## Charter Compliance

- [x] 标题用户原话句子（第一人称）
- [x] frontmatter 8 必填字段齐全
- [x] 「本 task 可解答」5 个 query 锚点
- [x] 完成路径 6 步
- [x] 引用真实 acceptance.intent block
- [x] 文件 ≤ 250 行（实际 ~75 行）
