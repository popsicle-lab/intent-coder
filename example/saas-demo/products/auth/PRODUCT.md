---
Last-Updated: 2026-05-13
Last-Decision-Ref: PDR-0001
Product: auth
Audience: ["PM", "客户成功", "销售", "AI Copilot"]
---

# Auth — 登录注册产品

> SaaS 用户的入口。负责注册、登录、会话管理、2FA、账号生命周期。
> 所有其它 product（billing、admin-console）都依赖本 product 的会话状态。

---

## What this product does

Auth 给 SaaS 用户提供安全、低摩擦的身份认证。

**核心定位**:

- **首日**：用户用邮箱地址 + 密码注册一个 Account，并在注册当天**强制**启用 2FA
  （TOTP）—— 不接受「2FA 可选」的妥协（见 PDR-0001 § Decision）
- **日常**：用户用「邮箱密码 + 2FA 验证码」或「Google OAuth 一键」登录；登录后
  获得 Session 凭证，有效期 30 天，IP/UA 异常自动失效
- **故障**：忘记密码走「邮箱验证链接」恢复；2FA 设备丢失走「Recovery Code」恢
  复——10 个一次性备用码，启用 2FA 时一次性下发
- **管理**：Account Admin 可吊销旗下 User 的所有 Session；本 SaaS 内部 Admin 可
  跨 Account 查询 / 吊销，所有操作进 Audit Log
- **终结**：用户随时可注销账号；注销前 7 天进入「冷却期」，Audit Log 保留 90 天

`Decision-Ref: PDR-0001`

---

## User Intents Catalog

> AI Copilot 的核心索引：用户原话问句 → task 映射。
>
> 「为什么这么选」类策略问题不进本表，进 PDR。

| User Query | → Task | Journey Stage | Audience |
|---|---|---|---|
| "怎么注册账号？" | [T-0001](tasks/onboarding/T-0001-first-time-signup-with-email.md) | onboarding | new-user |
| "为什么必须扫码 2FA？我能不能跳过？" | [T-0002](tasks/onboarding/T-0002-enable-2fa-after-signup.md) | onboarding | new-user |
| "怎么登录？" | [T-0010](tasks/daily-ops/T-0010-login-with-email-password.md) | daily-ops | end-user |
| "我能用 Google 账号登录吗？" | [T-0011](tasks/daily-ops/T-0011-login-with-google-oauth.md) | daily-ops | end-user |
| "忘记密码了怎么办？" | [T-0020](tasks/troubleshooting/T-0020-recover-account-via-email.md) | troubleshooting | end-user |
| "我换手机了，2FA 怎么办？" | [T-0021](tasks/troubleshooting/T-0021-lost-2fa-device.md) | troubleshooting | end-user |
| "怎么把某个员工的登录会话全部踢掉？" | [T-0030](tasks/admin/T-0030-revoke-user-sessions.md) | admin | Account Admin |
| "我不用了，怎么注销？" | [T-0040](tasks/lifecycle/T-0040-delete-account.md) | lifecycle | end-user |
| "为什么 Pro 版才能用 SSO？" | —— *(策略问题，去 PDR-0001)* | —— | 商业咨询 |

---

## Intents Catalog

> 形式化 intent 文件的索引。所有 acceptance block 与 task_id 双射。

| Intent Layer | File | Owner Tasks |
|---|---|---|
| Acceptance | [`intents/acceptance.intent`](intents/acceptance.intent) | T-0001, T-0002, T-0010, T-0011, T-0020, T-0021, T-0030, T-0040 |
| Invariants | [`intents/invariants.intent`](intents/invariants.intent) | 跨 task（email-uniqueness / no-plaintext-password / session-isolation 等）|
| Contracts | [`intents/contracts.intent`](intents/contracts.intent) | 大部分 `[Awaiting ADR-XXXX]`——等 arch-debate 落地 |

---

## Status

- **当前迭代**: MVP（PDR-0001 落地）
- **可见 task 数**: 8（覆盖 5 个旅程阶段）
- **未完成的关键依赖**:
  - ADR-XXXX（Verifier 模块如何对 Session 服务暴露接口）—— 阻塞 contracts.intent
  - 项目自带 intent spec writer —— 阻塞 acceptance.intent 种子收紧
  - intent-consistency-check skill 上线 —— 阻塞 Z3 闸

`Decision-Ref: PDR-0001`

---

## 不在本 product 范围

- ❌ 计费 / 订阅 / 套餐升降 → `billing` product
- ❌ 跨 Account 的用户查询 / 报表 → `admin-console` product
- ❌ 单点登录 SAML/OIDC 服务端（IdP 模式）—— 本 SaaS 只做 OAuth 客户端（SP 模式）

`Decision-Ref: PDR-0001`
