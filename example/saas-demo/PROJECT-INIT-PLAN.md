# Project Init Plan — saas-demo

> **Status**: Approved (demo 留档)
> **Created**: 2026-05-13
> **Author**: intent-coder demo agent + 人类确认者

本计划是 `project-init` skill 的产出留档。真实工作流里，本文件审批通过后才允许
执行 `popsicle init` + `git submodule add` + `mkdir -p` 等不可逆操作。

---

## Repository Identity

| 字段 | 值 |
|---|---|
| 仓库名 | `saas-demo` |
| 本地路径 | `/Users/narwal/Workspace/github/intent-coder/example/saas-demo` |
| Issue key 前缀 | `SAAS`（用于 BUG-SAAS-1、TC-SAAS-1 ...）|
| 默认 agent target | `claude` |
| License | `MIT`（greenfield 自由选）|
| 初始分支 | `main` |

---

## Product Inventory

> 一旦批准，这些名字会成为 `products/<name>/` 目录路径，被所有下游文档引用。

| # | Product | 一行用途 | 估计 LoC | 来源 | 状态 |
|---|---|---|---|---|---|
| 1 | `auth` | 用户注册 / 登录 / 会话 / 2FA / 账号生命周期 | ~3,000（估）| 用户输入 | **slice（首切片）** |
| 2 | `billing` | Stripe 集成 / 订阅 / 发票 / 套餐升降 | ~2,500（估）| 用户输入 | scaffold-only（演示部分填充）|
| 3 | `admin-console` | 内部管理后台 / 用户查询 / 配额审计 | ~2,000（估）| 用户输入 | scaffold-only（仅占位）|

**对照硬规则的校验**：

- [x] 每个名字客户能识别（auth / billing / admin-console 都是 SaaS 行业通用词）
- [x] 数量在 3-7 之间（3 个）
- [x] 没有万能的 "shared" / "common" / "utils" product
- [x] greenfield 项目，无 fact-extraction-report 映射要求

---

## Legacy Source

| 字段 | 值 |
|---|---|
| 类型 | **Greenfield**（无 legacy 仓库）|
| Submodule | —— |
| Pinned commit SHA | —— |
| License 检查 | —— |

> 因为是 greenfield，跳过 `git submodule add` 和 `fact-extractor` skill。
> Bootstrap 流程中下游 writer **没有** fact-extraction-report 可消费——所有
> 「数字 / 模块 / 风险」类陈述都会标 `[未经事实基验证]`。这是 greenfield 项目
> 的天然代价；上线后通过实际运行数据回填。

---

## First Migration Slice

**选中**: `auth`

**理由**:

1. **onboarding 必经**：用户接触产品的第一步永远是注册/登录——auth 是 5 个旅程
   阶段都被高频用的 product
2. **反向依赖少**：billing 和 admin-console 都依赖 auth 的会话 / 用户 ID，反过
   来 auth 不依赖它们 —— 适合作为「playbook product」
3. **安全敏感**：早期把 auth 的 invariants（如「永不存明文密码」「2FA 不可绕过」）
   形式化进 `intents/invariants.intent`，比后期补救成本低

**考虑过的替代**：

- **`billing`**: 商业价值最高，但耦合 Stripe 外部依赖，invariants 受外部 API 制
  约，做不到「自闭环可验证」—— 作为首切片 IDD ROI 反而低
- **`admin-console`**: LoC 最少，但用户量极小（仅内部），首切片产出的 playbook
  对真实用户旅程参考价值不足

**「首切片」对本 demo 的具体含义**：

- `products/auth/` 被完整填充（8 个 task 覆盖 5 个旅程阶段 + 1 个 PDR + acceptance.intent
  含 5+ block）
- `products/billing/` 被**部分**填充（6 个 task + 1 个 PDR，演示跨 product 协调）
- `products/admin-console/` 只铺骨架，每个文件都停在 `[TBD: needs archaeology]`

---

## Module Dependencies

| 模块 | 来源 | 用途 | 状态 |
|---|---|---|---|
| `intent-coder` | `../..`（相对路径，本 demo 在仓库内）| 主模块 —— 含 project-init / fact-extractor / product-debate / prd-writer | 已装 |
| `(your-arch-debate)` | —— | 处理 contracts.intent 条目；本 demo 标 `[ADR-XXXX 待落地]` 跳过 | 未装（演示 hand-off）|
| `(your-intent-spec-writer)` | —— | 把 acceptance.intent 种子收紧；本 demo 直接展示「种子即最终态」简化版 | 未装（演示 hand-off）|

> 因为是 demo，外部 writer 模块**故意不装**，让读者清晰看到「契约位」长什么样
> （contracts.intent 大段标 `# Awaiting ADR-XXXX`）。

---

## Scaffolding Manifest

> 已在本 demo 中**完整铺出**。每个 product 含：
> - PRODUCT.md（auth / billing 已填，admin-console 仅占位）
> - ARCHITECTURE.md（同上）
> - tasks/{onboarding, daily-ops, troubleshooting, admin, lifecycle}/
>   + tasks/README.md
> - intents/{invariants, contracts, acceptance}.intent
> - decisions/{adr, pdr}/
> - proposals/{exploring, proposed, accepted, rejected}/
>
> 仓库级还含 docs/CHARTER.md + docs/user-journeys/ + migration/ 等。

完整目录树见 [`README.md`](README.md) §「目录导览」。

---

## 风险 & 未决问题

| 风险 / 问题 | 缓解 / 负责人 |
|---|---|
| greenfield 无 fact-extraction-report，所有数字类陈述无据可查 | 上线后用真实运行数据回填；接受 PRD 质量评分 IDD 适配度子项扣分 |
| 三个 product 之间的 invariants 边界可能渗漏（如「会话 token 应该只有 auth 持有」）| 通过 `docs/invariants/` 跨 product 层落地 |
| 跨 product 旅程 J-0001（注册→首次付费）涉及 auth 和 billing 双 PDR，PDR 评审顺序敏感 | 先 auth/PDR-0001，再 billing/PDR-0001，最后 J-0001 |

---

## Plan Checklist

- [x] Repository Identity 表全填满
- [x] Product Inventory 有 3 条（在 3-7 范围内）
- [x] 每个 product 条目有状态（slice / scaffold-only）和一行用途
- [x] 正好一个 product 标 **slice**（auth）
- [x] Legacy Source 标记为 Greenfield，跳过 submodule
- [x] First Migration Slice 章节列了 ≥ 1 个替代并说明否决理由（billing / admin-console）
- [x] Module Dependencies 已声明（外部 writer 故意不装，演示 hand-off）
- [x] Scaffolding Manifest 已穷举（见 README §目录导览）
- [x] **v0.2 任务图：每个 product 含 5 个旅程阶段目录**
- [x] **v0.2 任务图：`docs/user-journeys/` 全局层已铺**
- [x] 风险 & 未决问题 已记录
