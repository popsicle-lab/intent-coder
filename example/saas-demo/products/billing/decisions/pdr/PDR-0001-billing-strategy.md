# PDR-0001 — Billing Strategy: Stripe + 三档定价 + 周期末取消

- **Status**: Accepted
- **Date**: 2026-05-13
- **Deciders**: PM / Tech Lead / 财务 (all demo)
- **Supersedes**: ——
- **Related ADRs**: ADR-XXXX (Stripe Webhook / Subscription Storage / Prorate)

---

## Context

product-debate 留档 [migration/slices/2026-05-13-billing-product-debate.md](../../../../migration/slices/2026-05-13-billing-product-debate.md)
中讨论了三对核心选择：

1. **支付通道**：自建 vs Stripe vs Stripe + Adyen 备份
2. **定价档位**：单一价 vs 两档 vs 三档 vs 自由套餐
3. **取消策略**：立即生效 + 按比例退款 vs 周期末生效不退款

每对都做出了倾向 B2B SaaS 行业惯例 + 减少争议的取舍。

---

## Decision

| ID | 决策 | 理由 | 替代方案 / 否决理由 |
|---|---|---|---|
| **D1** | 支付通道**仅** Stripe | 行业标准；webhook 模型成熟；PCI DSS 由 Stripe 负责 | 自建：PCI 合规成本巨大；Stripe + Adyen：复杂度 vs 单点故障收益不成比例 |
| **D2** | 三档定价：Free / Pro / Team | 覆盖 95% B2B SaaS 客户场景；选项过多决策摩擦 | 单一价：不适配团队规模；自由套餐：UX 复杂 |
| **D3** | 卡号永不入本 SaaS 服务器 | PCI DSS 简化 + 安全底线 | "代理收卡号转给 Stripe"：触发 PCI Level 1 全套审计 |
| **D4** | Dunning：1/3/7 天三次重试，第 3 次失败自动降级 Free（不删数据）| 给用户充分恢复窗口；不立即停服防误伤 | "立即停服"：客户体验差；"无限重试"：财务记账复杂 |
| **D5** | Seat 调整按当前周期剩余天数 prorate；周期内增减对称 | 公平；与 Stripe 默认行为一致 | "按月一次性结算"：不符合用户预期 |
| **D6** | 取消订阅默认 = 周期末生效，不退款 | 行业惯例；减少滥用（订一个月用一个月再退）| "立即生效退余款"：财务复杂；用户也不期待 SaaS 这么做 |

---

## Intent Impact

| Intent Layer | File | New / Modified Entries |
|---|---|---|
| Acceptance | `products/billing/intents/acceptance.intent` | + 7 个 block，与 T-0001~T-0040 双射 |
| Invariants | `products/billing/intents/invariants.intent` | + `single-active-subscription`<br>+ `no-double-charge`<br>+ `prorate-symmetric`<br>+ `no-plaintext-card-number` |
| Contracts | `products/billing/intents/contracts.intent` | + `consume_verify_session`（消费 auth）<br>+ `consume_user_signed_up_event`<br>+ `stripe_webhook_handler` [Awaiting ADR]<br>+ `subscription_store_interface` [Awaiting ADR]<br>+ `prorate_calculator` [Awaiting ADR] |

---

## Consequences

### Task File Updates (新增)

- `products/billing/tasks/onboarding/T-0001-upgrade-from-free-to-pro.md`
- `products/billing/tasks/daily-ops/T-0010-view-current-invoice.md`
- `products/billing/tasks/daily-ops/T-0011-update-payment-method.md`
- `products/billing/tasks/troubleshooting/T-0020-handle-payment-failure.md`
- `products/billing/tasks/admin/T-0030-manage-team-seats.md`
- `products/billing/tasks/lifecycle/T-0040-cancel-subscription-with-prorate.md`

### 其它活文档

- `products/billing/PRODUCT.md` —— 全文新建
- `products/billing/ARCHITECTURE.md` —— Module 骨架
- `products/billing/tasks/README.md` —— 首次生成
- `docs/user-journeys/J-0001-new-user-signup-and-first-payment.md` —— Stage 4-6 由本 PDR 落地

### 后续待跟进

- **ADR-XXXX** Stripe Webhook 签名验证 + Idempotency 策略
- **ADR-XXXX** Subscription Storage（Postgres? Mongo?）
- **ADR-XXXX** Prorate 计算精度（货币最小单位）
- **CADR-XXXX** 把 `single-active-subscription` 升格为全局 invariants

---

## Acceptance Criteria

1. 所有 Decision 表条目在 acceptance.intent 对应 block
2. `single-active-subscription` 在 Z3 可证（待 intent-spec-writer 收紧）
3. PDR Consequences 列出的所有文件本次 PR 已更新
4. J-0001 Stage 4-6 与本 PDR 决策一致

---

## Validation Plan

### 上线后监控

| 指标 | 目标 | 来源 |
|---|---|---|
| Free → Pro 转化率 | 行业基线 2-5% | 后端事件流 |
| 升级后 30s 内生效完成率 | > 99% | webhook 到 DB 延迟监控 |
| Dunning 第 1 次重试成功率 | > 50% | Stripe API metric |
| 重复扣费工单 | 0 | 客户支持 ticket |
| Seat prorate 对称性偏差 | < 0.01% | 财务对账 |

### AI 反馈闭环指标

| 指标 | 目标 |
|---|---|
| "扣费失败" 类 query 命中 T-0020 的召回率 | > 80% |
| Task 跨产品引用率（被 auth/T-0040 提及的次数）| 自然增长 |
