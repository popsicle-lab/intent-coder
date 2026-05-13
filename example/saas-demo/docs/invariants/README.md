# Global Invariants — saas-demo

> 跨 product 的领域永恒为真规则。本目录下的 `*.intent` 文件**约束所有 product**，
> 修改任一文件需要 **CADR**（Charter Amendment Decision Record）。
>
> Last-Updated: 2026-05-13
> Status: `[TBD]` — 等首切片完成后再识别哪些 invariant 应该全局化

---

## 当前状态

**本目录暂无 `.intent` 文件**——本 demo 中所有 invariants 都先放在各自 product 的
`products/<name>/intents/invariants.intent` 里。

按 IDD 标准做法，bootstrap 阶段不预设全局 invariants——等首切片（auth）完成跑通
后，回看哪些规则是「跨 product 必须保证」的，再升格到本目录。

---

## 候选全局 invariants（待 CADR 落地）

经初步分析，以下规则**可能**需要升格为全局 invariants。落地前需要：
1. 跑一次 charter 级讨论
2. 确认确实跨多个 product
3. 起草 CADR 审批

### 候选 1: User 全局唯一性

> 「一个 email 只能注册一个 User，跨所有 product 不可重复」

- **涉及**: auth（注册）+ billing（账单归属）+ admin-console（用户查询）
- **目前位置**: `products/auth/intents/invariants.intent` § `email-uniqueness`
- **升格理由**: billing 和 admin-console 都依赖「email → user_id 一对一」的假设

### 候选 2: Account 与 Subscription 一对一

> 「同一时刻每个 Account 有且仅有一个 active Subscription」

- **涉及**: billing（订阅）+ admin-console（计费状态查询）
- **目前位置**: `products/billing/intents/invariants.intent` § `single-active-subscription`
- **升格理由**: admin-console 的所有计费相关查询都依赖这条假设

### 候选 3: Audit Log 不可篡改

> 「Audit Log 一旦写入，跨所有 product 都不可修改或删除」

- **涉及**: auth（吊销 session）+ billing（修改 plan）+ admin-console（敏感查询）
- **目前位置**: 暂无（应该由 admin-console 落地后引入）
- **升格理由**: 这是 SOC2 / GDPR 合规底线，不能依赖某个 product 单方面保证

---

## 何时填本目录

参考 `intent-coder/skills/project-init/templates/doc-architecture-charter.md`
§「迁移序列」：**步骤 4 — 加 intent 层**，在首切片完成后跑。

本 demo 因为是早期演示，未到这一步。
