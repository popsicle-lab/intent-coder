# User Journeys — saas-demo

> 跨 product 的用户旅程。**单 product 内**的 task 串联用各自 product 内 task 的
> `related_next_tasks` 字段，**不**进本目录。
>
> Last-Updated: 2026-05-13

---

## 索引

| Journey | 涉及 product | Status | 负责 PDR |
|---|---|---|---|
| [J-0001 新用户注册并完成首次付费](J-0001-new-user-signup-and-first-payment.md) | auth + billing | Accepted | auth/PDR-0001 + billing/PDR-0001 |

---

## 何时创建新 Journey

满足以下**全部**条件再创建：

1. 涉及 ≥ 2 个 product
2. 用户能用一句完整原话讲出整个旅程（如「我注册并完成首次付费」）
3. 旅程的 Stage 顺序对用户体验至关重要（顺序变了体验就崩了）

不满足 1 → 用单 product 内 task 的 `related_next_tasks` 即可
不满足 2 → 这是「随便排列的任务集合」不是旅程
不满足 3 → 顺序无关的话用 product `PRODUCT.md` 的 User Intents Catalog 即可

---

## 命名规则

- **Journey ID**: `J-{nnnn}` 全局递增（不分 product），4 位 0 填充
- **文件名**: `J-{nnnn}-{kebab-slug}.md`
- **目录**: 本目录单层，**不**按阶段分类（journey 本身已阶段无关）
