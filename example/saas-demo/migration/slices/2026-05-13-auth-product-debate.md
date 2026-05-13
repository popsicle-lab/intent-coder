# Product Debate Record — auth (2026-05-13)

> **Skill**: `product-debate` (intent-coder)
> **Target Product**: auth
> **Slice**: 1（首切片）
> **Outcome**: → PDR-0001-auth-strategy Accepted
>
> ⚠️ 本文件是**留档摘要**。真实工作流里 transcript 应该是几千字的多轮交锋。
> 为 demo 简洁起见保留要点。

---

## Phase 1 — Setup

**Inputs**:
- 无 `fact-extraction-report`（greenfield）
- `project-init-plan` → Product Inventory: auth / billing / admin-console
- 用户置信度: 4/5（PM 已有清晰想法，需要 challenge）

**Roles**:
| 简称 | 视角 | 配置文件 |
|---|---|---|
| PM | 产品经理 | references/default-roles.md § PM |
| UXR | 用户研究 | § UXR |
| GROWTH | 增长 / 转化 | § GROWTH |
| ENGLD | 工程 / 安全 | § ENGLD |
| BIZ | 商业 / 合规 | § BIZ |

**辩论焦点**（用户提出）:
> "SaaS 登录怎么做，邮箱密码 / 魔法链接 / OAuth / 多因子的取舍"

---

## Phase 2 — Divergent (探索方案空间)

5 个角色独立提出方案：

| 角色 | 倾向方案 | 核心论点 |
|---|---|---|
| PM | A: Email + 密码 + 强制 2FA + Google OAuth 选项 | 安全 + 摩擦平衡 |
| UXR | B: Magic Link only | 用户调研显示「不想记密码」是 top 抱怨 |
| GROWTH | C: Email + 密码 (无 2FA) + 多 OAuth | 注册转化率最高 |
| ENGLD | A 或 D: Passkey-only (WebAuthn) | 安全底线；密码即未来负债 |
| BIZ | A | SOC2 准备；2FA 是 B2B 客户基线要求 |

**质询**:
- ENGLD → UXR: "Magic Link 在邮箱被入侵时**完全**失守，B2B 客户不接受"
- BIZ → GROWTH: "无 2FA 注册的客户后期会要求加，迁移成本远大于早期摩擦"
- UXR → ENGLD: "Passkey-only 教育成本太高，2026 年还不是 mass-market 时机"
- PM → 所有: "我们要的是企业团队用户，转化漏斗里 2FA 摩擦是可接受成本"

---

## Phase 3 — Convergent (收敛到 1-2 个方案)

**剩下两个候选**:
- 方案 A: Email + 密码 + 强制 2FA + Google OAuth 选项（PM / ENGLD / BIZ 共同支持）
- 方案 D: Passkey-only（ENGLD 探索性支持）

**最终 trade-off 讨论**:
- 方案 D 在 2026 年的 mass-market 接受度不足（UXR + GROWTH 共同否决）
- 方案 A 是行业惯例，与 B2B 客户期待一致
- **Resolution**: 方案 A，但保留 PDR-0002 backlog 用于「未来转 Passkey 为主」

---

## Phase 4 — Conclude (IDD 专属: Task 识别 + Intent 归类)

### 候选 Tasks (待用户确认)

```
TBD-1 [onboarding]       「我用邮箱注册一个新账号」
TBD-2 [onboarding]       「我注册当天扫码启用 2FA」
TBD-3 [daily-ops]        「我用邮箱密码 + 2FA 登录」
TBD-4 [daily-ops]        「我用 Google 账号一键登录」
TBD-5 [troubleshooting]  「我忘记密码后通过邮箱验证恢复」
TBD-6 [troubleshooting]  「我换了手机后用 Recovery Code 重置 2FA」
TBD-7 [admin]            「我作为 Account Admin 吊销某个员工的所有 Session」
TBD-8 [lifecycle]        「我注销账号并等 7 天冷却期」
```

### Intent 层归类

| 声明 | 层 | 关联 task |
|---|---|---|
| 登录 P95 < 200ms | acceptance | TBD-3 |
| 注册 ≤ 90s | acceptance | TBD-1 |
| Recovery Code 只显示一次 | acceptance | TBD-2 |
| 吊销 5s 内生效 | acceptance | TBD-7 |
| 邮箱全局唯一 | invariants（候选 docs/invariants/）| 跨 task |
| 永不明文密码 | invariants | 跨 task |
| 2FA 启用后不可关闭 | invariants | TBD-2 + 跨 task |
| Audit Log 90 天保留 | invariants（候选 docs/invariants/）| TBD-7, TBD-8 |
| `verify_session` 跨 product RPC | contracts（[Awaiting ADR-0003]）| 跨 product |
| `revoke_session_by_admin` | contracts（[Awaiting ADR-0004]）| TBD-7 |

### 用户确认

✅ Task 划分确认通过（用户「就 8 个 task，不拆不合」）
✅ Intent 归类确认通过

---

## Artifacts

- `prd-draft.md` —— 移交给 prd-writer 作为输入
- `decision-matrix.md` —— 4 候选方案打分（A 最高，D 次之）
- 本文件（debate-record）

→ **下一步**: prd-writer skill 把 prd-draft 提升为五件套（PRD overview + 8 task
+ tasks/README.md + acceptance-intent-seed + PDR-0001-skeleton）
