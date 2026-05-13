# Migration Progress — saas-demo

> Last-Updated: 2026-05-13

`saas-demo` 是 greenfield 项目，**没有真实迁移**。本文件用作 IDD 工作流的进度
追踪 + 决策时间线。

---

## 时间线

| 日期 | 事件 | 产出物 |
|---|---|---|
| 2026-05-13 | project-init bootstrap | `docs/CHARTER.md` + 三个 product 骨架 |
| 2026-05-13 | product-debate (auth) | [slices/2026-05-13-auth-product-debate.md](slices/2026-05-13-auth-product-debate.md) |
| 2026-05-13 | prd-writer (auth) → PDR-0001 Accepted | auth 五件套 |
| 2026-05-13 | product-debate (billing) | [slices/2026-05-13-billing-product-debate.md](slices/2026-05-13-billing-product-debate.md) |
| 2026-05-13 | prd-writer (billing) → PDR-0001 Accepted | billing 五件套 |
| 2026-05-13 | 跨 product Journey J-0001 落地 | docs/user-journeys/J-0001-... |

---

## 当前阶段

**Slice 1 (auth) + 一半的 Slice 2 (billing) 完成**。下一步：

1. `arch-debate` / `rfc-writer` 模块装入，产出 ADR-0001~5（auth）
2. `intent-spec-writer` 模块装入，收紧 acceptance.intent 种子
3. `intent-consistency-check` 跑 Z3 闸
4. 实施代码 + 上线
5. Slice 3: admin-console 启动 product-debate

---

## 健康度指标

| 指标 | 当前 | 目标 |
|---|---|---|
| Product 总数 | 3 | 3-7 ✅ |
| 已 Accept PDR | 2 (auth/PDR-0001, billing/PDR-0001) | —— |
| 等候 ADR 数 | ≥ 7（auth 5 + billing 2+） | < 15 |
| 跨 product Journey | 1 (J-0001) | —— |
| Task 文件总数 | 14 (auth 8 + billing 6) | —— |
| `[TBD]` 占位密度 | admin-console 全为 TBD（预期）；auth/billing 0 个 [TBD]（除 contracts.intent）| —— |
