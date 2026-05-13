# `auth` Tasks Index

> **本文件由 prd-writer 首次生成、living-doc-author 维护。请勿手工编辑**。
> Last-Generated: 2026-05-13

本目录按**用户旅程阶段**组织 auth product 的所有 task。每个 task 是独立的 RAG
chunk，含 YAML frontmatter，可被 AI Copilot 单独召回回答用户问题。

---

## 索引

### onboarding/ — 首次接触到首次成功

| Task | 用户问句锚点 | Audience | Last-Updated |
|---|---|---|---|
| [T-0001 我用邮箱注册一个新账号](onboarding/T-0001-first-time-signup-with-email.md) | 怎么注册账号？/ 注册需要哪些信息？ | new-user | 2026-05-13 |
| [T-0002 我注册当天扫码启用 2FA](onboarding/T-0002-enable-2fa-after-signup.md) | 为什么必须扫码 2FA？/ 能不能跳过？ | new-user | 2026-05-13 |

### daily-ops/ — 日常使用

| Task | 用户问句锚点 | Audience | Last-Updated |
|---|---|---|---|
| [T-0010 我用邮箱密码 + 2FA 登录](daily-ops/T-0010-login-with-email-password.md) | 怎么登录？/ 2FA 验证码在哪？ | end-user | 2026-05-13 |
| [T-0011 我用 Google 账号一键登录](daily-ops/T-0011-login-with-google-oauth.md) | 能用 Google 登录吗？/ OAuth 是什么？ | end-user | 2026-05-13 |

### troubleshooting/ — 故障排查

| Task | 用户问句锚点 | Audience | Last-Updated |
|---|---|---|---|
| [T-0020 我忘记密码后通过邮箱验证恢复](troubleshooting/T-0020-recover-account-via-email.md) | 忘记密码怎么办？/ 验证码邮件多久到？ | end-user | 2026-05-13 |
| [T-0021 我换了手机后用 Recovery Code 重置 2FA](troubleshooting/T-0021-lost-2fa-device.md) | 2FA 设备丢了怎么办？/ Recovery Code 是什么？ | end-user | 2026-05-13 |

### admin/ — 管理类（配额 / 权限 / 审计）

| Task | 用户问句锚点 | Audience | Last-Updated |
|---|---|---|---|
| [T-0030 我作为 Account Admin 吊销某个员工的所有 Session](admin/T-0030-revoke-user-sessions.md) | 怎么把员工踢下线？/ 离职员工会话怎么处理？ | Account Admin | 2026-05-13 |

### lifecycle/ — 终止 / 迁出 / 续费

| Task | 用户问句锚点 | Audience | Last-Updated |
|---|---|---|---|
| [T-0040 我注销账号并等 7 天冷却期](lifecycle/T-0040-delete-account.md) | 不用了怎么注销？/ 注销后还能恢复吗？ | end-user / Account Admin | 2026-05-13 |

---

## 健康度统计

> 由 living-doc-author 在重跑时刷新。本统计是「文档腐烂预警」的核心信号。

| 旅程阶段 | Task 数 | 平均行数 | 上次更新最久 | 未引用 task |
|---|---|---|---|---|
| onboarding | 2 | ~110 | T-0001 (0 天) | 无 |
| daily-ops | 2 | ~95 | T-0010 (0 天) | 无 |
| troubleshooting | 2 | ~115 | T-0020 (0 天) | 无 |
| admin | 1 | ~90 | T-0030 (0 天) | 无 |
| lifecycle | 1 | ~85 | T-0040 (0 天) | 无 |

---

## 跨 Product 旅程

涉及本 product 的跨 product 旅程：

- [J-0001 新用户注册并完成首次付费](../../../docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)
  —— 本 product 承担 Stage 1 / 2 / 3（注册 + 2FA + 首次登录）

---

## 维护规则

1. **新增 task** 由 `prd-writer` skill 通过 PDR 引入
2. **修改 task** 必须有新 PDR（charter 第 3 条铁律）
3. **删除 task** 必须有标注 `Supersedes` 的 PDR 显式废止
4. **重新分类 task**（在 5 个目录之间移动）算修改，需要 PDR
5. **重命名 slug**（不改 task_id）算小改，不需要新 PDR，但要更新 Last-Updated
