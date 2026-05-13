# Traceability — saas-demo

> 把 legacy 模块映射到新 product 的「翻译表」。
>
> saas-demo 是 **greenfield**，**没有 legacy**——本表演示该表的标准结构，全部
> 标 `N/A (greenfield)`。

---

## Legacy Module → New Product Map

| Legacy Module | LoC (估) | Primary Owner Product | Risk Notes |
|---|---|---|---|
| `N/A (greenfield)` | —— | —— | —— |

---

## 用法说明（真实项目）

当 saas-demo 有 legacy 时，本表应该这样填：

```markdown
| Legacy Module | LoC | Primary Owner Product | Risk Notes |
| `legacy/login` | 1,200 | auth | unsafe access; needs PDR-XXXX |
| `legacy/payment` | 800 | billing | depends on global `db_pool` |
| `legacy/admin` | 600 | admin-console | tight coupling with `legacy/login` session |
```

并配合每次 `prd-writer` 产出在 [progress.md](progress.md) 增加一行。

---

## Decision-Ref

无（greenfield）。
