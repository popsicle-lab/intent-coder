# Project Context — saas-demo

> 项目全景上下文。供新加入的人 / AI agent 在 30 秒内对齐项目当前状态。
>
> Last-Updated: 2026-05-13
> Last-Decision-Ref: project-init bootstrap

---

## 一句话

`saas-demo` 是一个用于演示 intent-coder v0.3 任务图范式的样板 SaaS 项目，含登录、
订阅计费、管理后台三个 product。

---

## 当前状态（IDD 工作流位置）

```
project-init           ✅ Complete
fact-extractor         ⏭️ Skipped (greenfield, no legacy)
product-debate (auth)  ✅ Complete (PDR-0001-auth-strategy)
prd-writer (auth)      ✅ Complete (8 tasks + PDR + acceptance.intent)
product-debate (billing) ✅ Complete (PDR-0001-billing-strategy)
prd-writer (billing)   ✅ Complete (6 tasks + PDR + acceptance.intent)
arch-debate            ⏸️ Not run (contracts.intent 大段标 [ADR-XXXX 待])
rfc-writer             ⏸️ Not run
intent spec writer     ⏸️ Not run (acceptance.intent 种子尚未收紧)
intent-consistency-check ⏸️ Not run (intents 未合并)
implementation         ⏸️ Not started
```

---

## Product 状态

| Product | 状态 | Tasks 数 | 备注 |
|---|---|---|---|
| `auth` | ✅ **首切片，完整填充** | 8（覆盖 5 个旅程阶段）| PDR-0001 Accepted |
| `billing` | ✅ 部分填充（演示跨 product 协调）| 6（覆盖 5 个旅程阶段）| PDR-0001 Accepted；依赖 Stripe 外部 API |
| `admin-console` | ⏸️ scaffold-only | 0 | 仅占位，每个文件 `[TBD]` |

---

## 跨 Product Journey

| Journey | 状态 | 涉及 product |
|---|---|---|
| `J-0001 — 新用户注册并完成首次付费` | ✅ Accepted | auth + billing |

---

## 已知盲区 / 未决问题

1. **`docs/invariants/` 全局 invariants 缺失**
   - Auth 和 Billing 都有可能涉及「全局唯一」类的 invariant（如「一个 email 只能
     注册一个 User，跨所有 product 不可重复」）
   - 当前都放在了各自 product 的 `intents/invariants.intent`，可能需要重构到全局
   - **要做**：跑一次 charter 级别讨论，识别哪些 invariant 应该全局化

2. **Contracts.intent 大段空白**
   - 多处标 `# Awaiting ADR-XXXX from arch-debate`
   - 模块间接口的最终形式化定义需要 arch-debate 先收敛技术选型
   - **要做**：装一个 arch-debate / rfc-writer 模块（外部 writer 契约位）

3. **没有真实 fact-extraction-report**
   - greenfield 项目，所有「数字」类陈述都标 `[未经事实基验证]`
   - PRD 质量评分 § IDD 适配度子项扣分（acceptable，因为是 demo）
   - **要做**：上线后用真实运行数据回填 baseline

4. **AI 反馈闭环未启动**
   - 每个 PDR Validation Plan 都预留了 AI 召回热度 / 错答率监控字段
   - 但没有真实平台层遥测——这些字段全部空着
   - **要做**：（项目级）接入 RAG 引擎，遥测 task chunk 引用热度

---

## 重要文件速查

| 想看什么 | 去哪 |
|---|---|
| 四条铁律 | [`docs/CHARTER.md`](CHARTER.md) |
| 全局术语 | [`docs/glossary.md`](glossary.md) |
| 项目初始化决策 | [`../PROJECT-INIT-PLAN.md`](../PROJECT-INIT-PLAN.md) |
| 首切片 auth 概览 | [`../products/auth/PRODUCT.md`](../products/auth/PRODUCT.md) |
| auth 关键决策 | [`../products/auth/decisions/pdr/PDR-0001-auth-strategy.md`](../products/auth/decisions/pdr/PDR-0001-auth-strategy.md) |
| billing 关键决策 | [`../products/billing/decisions/pdr/PDR-0001-billing-strategy.md`](../products/billing/decisions/pdr/PDR-0001-billing-strategy.md) |
| 跨 product 旅程 | [`user-journeys/J-0001-new-user-signup-and-first-payment.md`](user-journeys/J-0001-new-user-signup-and-first-payment.md) |
| 辩论留档 | [`../migration/slices/`](../migration/slices/) |

---

## 联系与归属

| 角色 | 谁 |
|---|---|
| Product Owner | (demo, no owner) |
| Tech Lead | (demo, no owner) |
| AI agent target | claude (按 module.yaml `default_agent` 字段) |
