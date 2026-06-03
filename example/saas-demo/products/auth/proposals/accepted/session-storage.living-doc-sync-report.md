---
artifact: living-doc-sync-report
slug: session-storage
generated_by: living-doc-author
target: tasks-index, task-backrefs        # 本次按用户点名运行；附带 last-verified 回填
last_updated: 2026-06-03
docs_scanned: 14
drift_signals: 4
docs_refreshed: 3
manual_followups: 3
query_anchors:
  - "新增的 session 契约对账进活文档了吗？"
  - "哪些 task 还指着已迁走 / 已剥离的 intent 块？"
  - "Phase 2 活文档保活这次真跑通了吗？"
---

# 活文档保活报告 — session-storage

> 由 `living-doc-author` skill 生成。**只对账与刷新活文档元数据**,不创作正文、不改
> 业务逻辑——后者必须走 prd-writer + PDR / intent-spec-writer（charter 铁律）。

## Summary

| 指标 | 值 |
|---|---|
| target | tasks-index + task-backrefs（附带 last-verified 回填） |
| 扫描文档数 | 14（8 task + 4 auth .intent + ADR-0003 + intent-consistency-report） |
| 发现 drift 信号 | 4（断链 / 孤儿 / 未验证 / 1 处 .intent 正文不一致） |
| 本次刷新文档数 | 3（README + T-0010 + T-0030） |
| 待人工处置项 | 3 |
| 结论 | session 契约已对账落地；存量系统性锚点 drift 待人工 |

一句话结论:本次新增的 session 契约（`session.intent`,由 ADR-0003 落地、Z3 verified）已对账进
`tasks/README` 健康度、T-0010 / T-0030 的 `related_intents`(断链重指向)、反向引用(加 ADR-0003 +
session.intent)与 `last_verified`(回填报告日期);**Phase 2 保活闭环验证通过**。同时扫出一批
存量系统性锚点 drift 与 1 处 `.intent` 正文矛盾,均越界,转待人工。

## Scan Checklist

- [x] target 已确认（用户点名 tasks/README + 反向引用 → tasks-index + task-backrefs）
- [x] 所有 task / intent / PDR / PRODUCT.md 已枚举（8 task、4 auth intent、ADR-0003、一致性报告）
- [x] 四类 drift 信号已逐条核对,证据已记录
- [x] 已区分「可自动刷新的元数据」与「需 PDR / intent-spec-writer 的正文改动」

## Drift 信号

### 1. 过期 staleness

| 文档 | 证据 | 严重度 |
|---|---|---|
| 全部 8 个 auth task | `last_updated: 2026-05-13`,距今 21 天 | 正常(< 60 天告警阈值) |
| tasks/README | `Last-Generated: 2026-05-13`,健康度仍写「(0 天)」 | 轻微(已刷新) |

### 2. 断链 broken-ref

| 来源 | 失效引用 | 类型 | 处置 |
|---|---|---|---|
| T-0030 `related_intents` | `invariants.intent#session-isolation` | INV 已迁到 `session.intent`(invariants.intent 头注释第 28 行) | ✅ 本次重指向 |
| T-0010 `related_intents` | `invariants.intent#session-isolation` | 同上 | ✅ 本次重指向 |
| T-0001/0010/0011/0020/0030/0040 `related_intents` | `acceptance.intent#T-00xx-...-90s/-p95-/-60s/-5s` 等时间锚点 | D2 剥离后这些约束已降级为 task 成功标志,不再是 intent 块 | ⚠️ 待人工 |
| T-0001/0011 `related_intents` | `invariants.intent#email-uniqueness` | 对应 `theorem EmailUniqueness`,当前 skipped(struct-forall 未实现) | ⚠️ 待人工 |
| T-0001/0020 `related_intents` | `invariants.intent#no-plaintext-password` / T-0040 `#audit-log-retention` | D2 剥离,运行时/时间事实,无对应 intent 块 | ⚠️ 待人工 |

> 系统性根因:`related_intents` 用「逻辑 INV 名」锚点,而 Phase 0/1 重写后 intent 块改用
> intent-lang 风格块名(`SessionResolvesToOwner` / `TwoFactorStaysOn` …)+ D2 把时间/性能/
> 运行时约束整体剥离到 task 成功标志。**本次只修与新增 session 契约直接相关的 2 条**,其余存量
> 锚点 drift 需一次锚点约定收敛(转待人工)。

### 3. 孤儿 orphan

| 对象 | 情况 |
|---|---|
| `session.intent`(VerifySession / RejectRevokedSession / SessionResolvesToOwner) | 新增后**无任何 task 反向指入** → 本次修好 T-0010/T-0030 断链后解除孤儿 ✅ |
| acceptance 块 ↔ task 双射 | 9 个 acceptance intent 的 `// task: T-00xx` 注释与 8 个 task 文件一一可达,无孤儿 |

### 4. 未验证 unverified

| Task | 当前 last_verified | 报告中的状态 | 可回填? |
|---|---|---|---|
| T-0030 | ~ → 2026-05-13 | session.intent `SessionResolvesToOwner` verified | ✅ 已回填 |
| T-0010 | ~ → 2026-05-13 | 同上(依赖 session-isolation) | ✅ 已回填 |
| T-0002 / T-0021 | ~ | `2fa-required`(TwoFactorStaysOn)verified | 可回填,但超本次 session 范围 → 待人工 |
| T-0001/0011/0020/0040 | ~ | 关联块含 skipped(EmailUniqueness)或 D2 剥离项,非干净 verified | 不回填(保持 ~) |

## 刷新动作

| 文件 | 改动 |
|---|---|
| `products/auth/tasks/README.md` | `Last-Generated` 2026-05-13 → 2026-06-03;健康度「(0 天)」→「(21 天)」;加 session 对账说明段 |
| `tasks/admin/T-0030-revoke-user-sessions.md` | `related_intents`: `invariants.intent#session-isolation` → `session.intent#session-isolation`;`last_verified` ~ → 2026-05-13;反向引用加 ADR-0003 + session.intent |
| `tasks/daily-ops/T-0010-login-with-email-password.md` | 同 T-0030 的三处(断链重指向 / last_verified 回填 / 反向引用加 ADR-0003 + session.intent) |

## 健康度快照

| 旅程阶段 | Task 数 | 平均行数 | 最久未更新 | 未引用 task |
|---|---|---|---|---|
| onboarding | 2 | ~110 | T-0001(21 天) | 无 |
| daily-ops | 2 | ~95 | T-0010(21 天) | 无 |
| troubleshooting | 2 | ~115 | T-0020(21 天) | 无 |
| admin | 1 | ~90 | T-0030(21 天) | 无 |
| lifecycle | 1 | ~85 | T-0040(21 天) | 无 |

- 孤儿:本次解除 1 个(`session.intent` 已被 T-0010/T-0030 反向指入)。
- 过期:无(全部 21 天,未达 60 天告警)。

## 待人工处置

- [ ] **存量锚点约定收敛**:`related_intents` 的 INV 名锚点(`#xxx`)与 intent-lang 真实块名
      不一致 + D2 剥离项已无对应块。需定一个锚点约定(任务侧改用 `#BlockName`,或 intent 侧
      统一登记 INV 名锚点),再批量对账 6 个 task 的存量断链 → 走 intent-spec-writer + 维护者。
- [ ] **`contracts.intent` 第 62 行正文矛盾**:`session_store_interface ... [Awaiting ADR-0003]`
      与上方 goal 已 `[ADR-0003 Accepted 2026-05-13]` 自相矛盾。属 `.intent` 正文,living-doc-author
      不改 → 走 intent-spec-writer 同步内部契约位状态。
- [ ] **T-0002 / T-0021 last_verified 可回填**(2fa-required = TwoFactorStaysOn verified),
      本次限定 session 范围未动;下次跑 `--target last-verified` 全量回填。可选:T-0011 OAuth 登录
      也签发 session,是否补 `session.intent#session-isolation` 关联由 prd-writer 判定(属新增链接)。

---

## 检查清单（提交前勾完）

- [x] 四类 drift 信号都已扫描并列出
- [x] 刷新动作每条都对应真实文件改动
- [x] last_verified 只回填了 verified 的 task（T-0010 / T-0030;skipped/D2 项保持 ~）
- [x] 健康度快照数字与刷新后的 README 一致
- [x] 所有越界 drift 已转「待人工处置」,未擅自改 .intent 正文 / 任务正文
- [x] frontmatter 计数与正文一致（scanned 14 / signals 4 / refreshed 3 / followups 3）
