# PDR-0001 — Auth Strategy: Email/Password + 强制 2FA + Google OAuth

- **Status**: Accepted
- **Date**: 2026-05-13
- **Deciders**: PM (demo) / Tech Lead (demo) / Security Officer (demo)
- **Supersedes**: ——
- **Superseded-By**: ——
- **Related ADRs**: ADR-0001 / 0002 / 0003 / 0004 / 0005 (all awaiting)

---

## Context

`saas-demo` 是面向企业团队的 SaaS。早期阶段必须把「登录是否够安全」与「用户摩
擦是否够低」这对矛盾的取舍**显式定下来**，否则后续每个 product（billing 在调
verify_session 时、admin-console 在做 SSO 集成时）都会反复重谈。

product-debate 留档（[migration/slices/2026-05-13-auth-product-debate.md](../../../../migration/slices/2026-05-13-auth-product-debate.md)）
中 5 个 AI 角色辩论了 4 个候选方案：

1. **Email + 密码（无 2FA）** —— GROWTH 主推；ENGLD 强烈否决（B2B 安全底线不足）
2. **Magic Link 替代密码** —— UXR 主推；ENGLD + BIZ 否决（B2B 用户不接受邮箱依赖）
3. **Email + 密码 + 强制 2FA + Google OAuth 选项** —— ENGLD + PM 主推；最终采纳
4. **Passkey-only (WebAuthn)** —— ENGLD 探索性提出；UXR + GROWTH 否决（用户教育成本高，
   早期 SaaS 不适合）

---

## Decision

| ID | 决策 | 理由 | 替代方案 / 否决理由 |
|---|---|---|---|
| **D0** | Email 全局唯一 = User 主键 | 简化 OAuth 绑定、找回流程；与 SaaS 行业惯例一致 | "用户名 + 邮箱" 双轨：增加注册摩擦，无业务价值 |
| **D1** | 主登录路径 = Email + Password + 强制 2FA (TOTP) | B2B 安全底线；TOTP 离线工作不依赖短信 | SMS 2FA：依赖运营商，国际化 / SIM-swap 风险高 |
| **D2** | 2FA **不可选**（注册当天强制启用，不可关闭） | 不让用户「不知不觉降级安全」；与 SOC2 准备对齐 | "用户可选 2FA"：实际开启率 < 10%，安全底线不足 |
| **D3** | 提供 Google OAuth 作为**便捷登录入口**（仍需 2FA） | 降低日常登录摩擦；不替代 2FA | "OAuth 豁免 2FA"：双因子被偷换成单因子，违背 D1 初衷 |
| **D4** | 密码找回**只重置密码**，不重置 2FA | 邮箱被入侵也不能绕过 2FA | "邮箱可同时重置两者"：单点失效 = 全失效 |
| **D5** | Account Admin 可吊销旗下 User 所有 Session，但**不能吊销自己** | 防误操作 / 内部攻击；保留撤销窗口 | "Admin 无限制"：误操作把自己锁出来代价大 |
| **D6** | 注销账号有 **7 天冷却期**；Audit Log 额外 **保留 90 天** | GDPR / 用户后悔窗口 + SOC2 审计要求 | "立即删除"：合规风险；"30 天冷却"：用户体验差 |

---

## Intent Impact

| Intent Layer | File | New / Modified Entries |
|---|---|---|
| Acceptance | `products/auth/intents/acceptance.intent` | + `T-0001-signup-completes-in-90s`<br>+ `T-0002-2fa-recovery-codes-shown-once`<br>+ `T-0010-login-p95-under-200ms`<br>+ `T-0010-failed-login-lockout`<br>+ `T-0011-oauth-callback-p95-under-500ms`<br>+ `T-0020-recovery-email-arrives-in-60s`<br>+ `T-0021-recovery-code-one-time-use`<br>+ `T-0030-revoke-takes-effect-in-5s`<br>+ `T-0040-deletion-7d-cooldown` |
| Invariants | `products/auth/intents/invariants.intent` | + `email-uniqueness`<br>+ `no-plaintext-password`<br>+ `session-isolation`<br>+ `2fa-required-after-onboarding`<br>+ `audit-log-retention`<br>+ `no-self-revoke` |
| Contracts | `products/auth/intents/contracts.intent` | + `verify_session` (跨 product, 已定语义边界)<br>+ `revoke_session_by_admin`<br>+ `emit_signed_up_event`<br>+ `password_hashing_function` [Awaiting ADR-0001]<br>+ `totp_verification_function` [Awaiting ADR-0002]<br>+ `session_store_interface` [Awaiting ADR-0003]<br>+ `audit_log_writer` [Awaiting ADR-0005] |

---

## Consequences

> 列出所有被本 PDR **强制更新**的活文档段落。PR 必须在同一次提交里全部更新。

### Task File Updates (新增)

- `products/auth/tasks/onboarding/T-0001-first-time-signup-with-email.md` (created)
- `products/auth/tasks/onboarding/T-0002-enable-2fa-after-signup.md` (created)
- `products/auth/tasks/daily-ops/T-0010-login-with-email-password.md` (created)
- `products/auth/tasks/daily-ops/T-0011-login-with-google-oauth.md` (created)
- `products/auth/tasks/troubleshooting/T-0020-recover-account-via-email.md` (created)
- `products/auth/tasks/troubleshooting/T-0021-lost-2fa-device.md` (created)
- `products/auth/tasks/admin/T-0030-revoke-user-sessions.md` (created)
- `products/auth/tasks/lifecycle/T-0040-delete-account.md` (created)

### Task File Updates (修改)

无（首版）

### Task File Updates (删除)

无（首版）

### 其它活文档

- `products/auth/PRODUCT.md` — User Intents Catalog + Tasks Catalog 全部新建
- `products/auth/ARCHITECTURE.md` — Module 划分骨架；具体技术选型由 ADR-0001~5 填
- `products/auth/tasks/README.md` — Tasks Index 首次生成
- `docs/CHARTER.md` § "Bootstrap 序列" 标记 auth 首切片完成

### 后续待跟进的决策

- **ADR-0001** Password Hashing Algorithm Selection（待 arch-debate）
- **ADR-0002** TOTP Library Selection（待 arch-debate）
- **ADR-0003** Session Storage Selection（待 arch-debate）
- **ADR-0004** Audit Log Storage（待 arch-debate）
- **ADR-0005** OAuth Client Library Selection（待 arch-debate）
- **PDR-0002** GitHub / Apple / 微信 OAuth 扩展（用户反馈触发后再开）

---

## Acceptance Criteria (本 PDR 自身)

> 本 PDR Accepted 当天必须满足的可观察条件。

1. 上文 Decision 表中 D0-D6 全部体现在 `products/auth/intents/acceptance.intent` 的对应 block
2. 注册 → 启用 2FA → 登录 完整链路在 demo 文档上可读（[J-0001 Stage 1-3](../../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)）
3. `products/auth/PRODUCT.md` § User Intents Catalog 至少 8 条用户原话问句映射到 task
4. P95 性能边界（登录 200ms / OAuth 500ms / 吊销 5s 内生效）在 acceptance.intent 中可被 Z3 形式化
5. 拒绝 / 风险（未采纳的 4 个候选方案）在 product-debate 留档可追溯
6. Charter 第 3 条铁律：本 PDR 的 Consequences 表列出的**每一个**文件，本次 PR 都已更新（CI 强制）
7. Audit Log 90 天保留期在 `invariants.intent#audit-log-retention` 显式 Z3 可证

---

## Validation Plan

### 上线后监控

| 指标 | 目标 | 来源 |
|---|---|---|
| 注册到 2FA 启用完成率 | > 95% | T-0001 + T-0002 funnel |
| 登录 API P95 延迟 | < 200ms | 后端 APM |
| 2FA Recovery Code 二次使用率 | < 5%（用一次后还能继续用 = 严重 bug）| audit log |
| 密码爆破触发锁定次数 | < 0.1% of total login attempts | 锁定事件流 |
| OAuth 登录占比 | 30%-60%（D3 假设值）| Session 创建事件 |

### AI 反馈闭环指标

| 指标 | 目标 | 检测方式 |
|---|---|---|
| 「忘记密码」类 query 命中 T-0020 的召回率 | > 80% | RAG 引擎 query log |
| AI Copilot 引用 T-0001/0010/0020 的「错答率」 | < 3% | 用户「这个回答没帮到我」反馈率 |
| Task 文件平均「引用热度」分布 | 无单一 task 占 > 40%（说明 task 切得太粗）| RAG 召回统计 |

---

## Backlog Hooks

- **PDR-0002**（暂未启动）：OAuth provider 扩展到 GitHub / Apple / 微信
- **CADR-0001**（暂未启动）：把 `email-uniqueness` / `audit-log-retention` 升格为 `docs/invariants/` 全局规则
- **ADR-0001 ~ 0005**（暂未启动）：6 个待落地的技术 ADR
