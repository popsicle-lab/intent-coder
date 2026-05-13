---
task_id: T-0001
slug: first-time-signup-with-email
title: "我用邮箱注册一个新账号"
journey_stage: onboarding
audience: ["new-user"]
task_type: user-action
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 拥有一个能收件的邮箱
  - 没有用这个邮箱注册过本 SaaS（email 全局唯一）
limits:
  - 同一 IP 1 小时内最多 5 次注册（防刷）
  - 注册后强制立即启用 2FA（见 T-0002）
related_intents:
  - acceptance.intent#T-0001-signup-completes-in-90s
  - invariants.intent#email-uniqueness
  - invariants.intent#no-plaintext-password
related_next_tasks:
  - T-0002  # 注册成功后强制进入 2FA 启用
fact_cite: []  # greenfield，无 baseline 数据
last_verified: null
---

# 我用邮箱注册一个新账号

> 你是一个新用户，第一次接触本 SaaS。用邮箱地址 + 自定义密码完成注册，并立即进入 2FA 启用流程。

## 本 task 可解答

- "怎么注册账号？"
- "注册需要填什么信息？"
- "我已经有一个 Google 账号了，还要单独注册吗？"
- "注册要多久？"
- "我的邮箱已经被注册过怎么办？"

## 前提与限制

- 你的邮箱**全局**没注册过本 SaaS——若已注册，请直接 [T-0010 登录](../daily-ops/T-0010-login-with-email-password.md)
- 同一 IP 1 小时内已注册超过 5 次会被临时拒绝（异常情况见 [T-0020](../troubleshooting/T-0020-recover-account-via-email.md) 找回旧账号）
- 注册完成后**无法跳过 2FA 启用**——如果你不想用 2FA，本 SaaS 不是为你设计的（见 PDR-0001 § Decision）

## 完成路径

1. **进入注册页**，填三个字段：邮箱、密码（≥ 12 位含字母数字）、Account 名称（团队场景下显示用）
2. **点击「注册」**，前端做密码强度校验（弱密码立即提示）
3. **后端校验**邮箱全局唯一性，重复则返回明确错误（`此邮箱已注册，请直接登录`）
4. **后端创建 User**：密码 bcrypt cost=12 哈希存储（明文永不落任何介质——见 `invariants.intent#no-plaintext-password`）
5. **发送验证邮件**到该邮箱，链接有效期 1 小时
6. **用户在邮箱里点验证链接** → 浏览器跳回本 SaaS，自动进入 [T-0002 启用 2FA](T-0002-enable-2fa-after-signup.md)

> 期望耗时：**≤ 90 秒**（从打开注册页到点完邮件验证链接）。见 [`acceptance.intent#T-0001-signup-completes-in-90s`](../../intents/acceptance.intent)

## 可观察的成功标志

- 用户邮箱收到「欢迎注册」邮件
- 浏览器自动跳到 2FA 启用页（不是登录页——这是关键，避免「注册完不知道下一步」）
- 数据库中存在该用户记录，`email_verified_at` 字段非空
- 验证流程结束：[`acceptance.intent#T-0001-signup-completes-in-90s`](../../intents/acceptance.intent)

## Related Next Tasks

- **必经下一步**: [T-0002 注册当天扫码启用 2FA](T-0002-enable-2fa-after-signup.md)
- **如果跳出**: 注册了但没启用 2FA 的用户不能登录任何 product（见 `invariants.intent#2fa-required-after-onboarding`）
- **跨 product**: 注册完成会触发 billing product 自动创建 Free 订阅（J-0001 Stage 4，见 [J-0001](../../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)）

## 反向引用

- 跨 product 旅程: [J-0001 § Stage 1](../../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)
- PDR: [PDR-0001 § Consequences](../../decisions/pdr/PDR-0001-auth-strategy.md)

## Charter Compliance

- [x] 标题是用户原话句子（第一人称）
- [x] frontmatter 8 必填字段齐全
- [x] 「本 task 可解答」≥ 3 个 query 锚点
- [x] 完成路径在 3-7 步之间（6 步）
- [x] 引用 acceptance.intent 中真实存在的 block
- [x] 文件 ≤ 250 行（实际 ~75 行）
- [x] 末尾 Decision-Ref 已声明（frontmatter `decision_ref: PDR-0001`）
