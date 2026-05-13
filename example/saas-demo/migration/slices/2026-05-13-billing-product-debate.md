# Product Debate Record — billing (2026-05-13)

> **Skill**: `product-debate`
> **Target Product**: billing
> **Slice**: 2（演示跨 product 协调）
> **Outcome**: → PDR-0001-billing-strategy Accepted
>
> 留档摘要。

---

## Phase 1 — Setup

**Inputs**:
- `project-init-plan` + auth/PDR-0001（auth 已经定下 `verify_session` 接口契约）
- 无 fact-extraction-report

**辩论焦点**:
> "SaaS 订阅计费的支付通道 + 定价档位 + 取消策略"

---

## Phase 2 — Divergent

| 角色 | 倾向 |
|---|---|
| PM | 三档（Free / Pro / Team），Stripe 唯一 |
| UXR | 单一价：选项少决策快 |
| GROWTH | 多档（4-5 档）+ 年付优惠 |
| ENGLD | Stripe 唯一（PCI 合规由 Stripe 承担）|
| BIZ | 三档 + 企业版（高价人工询价）|

**质询**:
- ENGLD → 所有: "自建支付的 PCI 合规成本 >> 任何业务价值"
- BIZ → UXR: "单一价无法对应不同客户规模，损失中端市场"
- PM → GROWTH: "4-5 档增加 UI 复杂度；可以将「企业版」放后置入口"

---

## Phase 3 — Convergent

**收敛到**:
- 支付通道: Stripe only
- 定价: Free / Pro / Team (三档；企业版后置)
- 取消: 周期末生效，不退款（行业惯例）

---

## Phase 4 — Conclude

### 候选 Tasks

```
TBD-1 [onboarding]       「我从 Free 升级到 Pro 并完成首次付款」
TBD-2 [daily-ops]        「我查看本周期的发票和下次扣费时间」
TBD-3 [daily-ops]        「我更换绑定的支付方式」
TBD-4 [troubleshooting]  「我处理一次扣费失败」
TBD-5 [admin]            「我作为 Team Admin 增减 Seat」
TBD-6 [lifecycle]        「我取消订阅并按当前周期结束自动降级」
```

### Intent 层归类

| 声明 | 层 |
|---|---|
| 升级 30s 内生效 | acceptance / TBD-1 |
| Dunning 1/3/7 天三次重试 | acceptance / TBD-4 |
| 取消周期末生效不退款 | acceptance / TBD-6 |
| 同时只一个 active subscription | invariants (候选升格) |
| 不重复扣费 | invariants |
| Prorate 对称 | invariants |
| 卡号永不入服务器 | invariants |
| `consume_user_signed_up_event` | contracts（消费 auth）|

---

## 跨 product 协调

辩论中显式确认两个 hand-off:

1. **auth → billing 事件**: `user.signed_up` 用于自动创建 Free 订阅（J-0001 Stage 4）
2. **auth → billing 同步 RPC**: `verify_session` 每次计费操作前调用
3. **auth/T-0040 ↔ billing/T-0040**: 注销账号前**必须**先取消订阅；本流程在 PDR-0001 § Cross-Product Integration 显式声明

→ 这些跨 product 约定**不写入** billing/PDR-0001 单方面决定，而是体现在
J-0001 Journey + 两个 product 的 contracts.intent 里（双方都引用）

---

## Artifacts

- `prd-draft.md` → prd-writer
- 本文件（debate-record）

→ **下一步**: prd-writer 提升 + J-0001 同步落地
