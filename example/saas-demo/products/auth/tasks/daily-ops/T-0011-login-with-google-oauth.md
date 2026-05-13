---
task_id: T-0011
slug: login-with-google-oauth
title: "我用 Google 账号一键登录"
journey_stage: daily-ops
audience: ["end-user"]
task_type: user-action
decision_ref: PDR-0001
last_updated: 2026-05-13
prerequisites:
  - 拥有一个 Google 账号
  - **首次**用 OAuth 登录需先完成 T-0001 邮箱注册并绑定 Google 身份（或本次注册时直接选 Google）
limits:
  - 一个 Google 账号只能绑定**一个**本 SaaS User（一对一）
  - OAuth 登录**不豁免** 2FA——首次绑定后仍需走 T-0002 启用 2FA
related_intents:
  - acceptance.intent#T-0011-oauth-callback-p95-under-500ms
  - invariants.intent#email-uniqueness
  - invariants.intent#2fa-required-after-onboarding
related_next_tasks: []
fact_cite: []
last_verified: null
---

# 我用 Google 账号一键登录

> 你想跳过手动输密码 + 6 位 TOTP 码，用 Google 账号一键登录。本 SaaS 支持 OAuth 客户端模式（不是 IdP）。

## 本 task 可解答

- "能用 Google 登录吗？"
- "OAuth 比邮箱密码方便吗？还要 2FA 吗？"
- "我之前用邮箱注册的，能改成 Google 登录吗？"
- "我能用 GitHub 登录吗？"
- "Google 账号被盗了我的 SaaS 账号会被盗吗？"

## 前提与限制

- 本 SaaS 当前**仅支持 Google OAuth**——GitHub / Apple / 微信等待 PDR-0002 落地（暂未规划）
- 一个 Google 账号**只能**绑定一个本 SaaS User —— 见 `invariants.intent#email-uniqueness`
- OAuth 登录**不能豁免 2FA**——即使你的 Google 账号已开 2FA，本 SaaS 仍要求你绑定 TOTP（见 PDR-0001 § Decision D2 备注）
- Google 账号被盗时本 SaaS 账号有风险——但因为有 2FA 二级验证，攻击者拿到 Google 也不能登进本 SaaS

## 完成路径

1. **登录页点击「Continue with Google」** → 浏览器跳到 Google OAuth 授权页
2. **用户确认授权** → Google 把 OAuth code 回调到本 SaaS 的 `/oauth/google/callback`
3. **后端用 code 换 ID Token**（调用 Google 的 token endpoint）
4. **从 ID Token 取 email**，查本 SaaS User 表：
   - **已存在**：直接进入第 5 步 2FA 验证
   - **不存在**：（首次绑定）走简化注册流程——自动创建 User、跳到 [T-0002 启用 2FA](../onboarding/T-0002-enable-2fa-after-signup.md)
5. **要求用户输入 6 位 TOTP 码** → 验证通过签发 Session
6. **跳转到 product 主页面**

> 期望耗时：**P95 < 500ms**（含 Google 外部调用）。见 [`acceptance.intent#T-0011-oauth-callback-p95-under-500ms`](../../intents/acceptance.intent)

## 可观察的成功标志

- 数据库 `oauth_identities` 表存在 `(provider=google, sub=<google-uid>, user_id=<本 SaaS uid>)` 记录
- 同 [T-0010](T-0010-login-with-email-password.md) 的 Session 凭证产物

## Related Next Tasks

- **首次绑定后必经**: [T-0002 启用 2FA](../onboarding/T-0002-enable-2fa-after-signup.md)
- **后续登录**: 直接回到本 task 第 1 步
- **想解绑 Google**: 走「账号设置 → 解绑外部身份」（本 demo 暂未单列为 task）

## 反向引用

- PDR: [PDR-0001 § Decision (D3)](../../decisions/pdr/PDR-0001-auth-strategy.md)

## Charter Compliance

- [x] 标题用户原话句子
- [x] frontmatter 完整
- [x] 5 个 query 锚点
- [x] 6 步完成路径
- [x] 文件 ≤ 250 行
