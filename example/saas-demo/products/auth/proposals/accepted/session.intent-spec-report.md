---
artifact: intent-spec-report
slug: session-storage
generated_by: intent-spec-writer
target_product: auth
source_seed: session-storage.contracts-unlocked.intent
output_file: products/auth/intents/session.intent
verified: true
last_updated: 2026-05-13
query_anchors:
  - "session-isolation 收紧成什么了？"
  - "为什么 s 用 unprimed、r 用 primed？"
---

# Intent 收紧报告 — session-storage

> intent-spec-writer 把 adr-writer 解锁的契约逻辑收紧成可验证 intent-lang（dogfood）。

## Summary

| 指标 | 值 |
|---|---|
| 来源种子 | session-storage.contracts-unlocked.intent（[ADR-0003 Accepted]）|
| 产出文件 | products/auth/intents/session.intent（新建）|
| 收紧的 safety | 1（SessionResolvesToOwner）|
| 触发 intent | 2（VerifySession / RejectRevokedSession）|
| 自验结果 | **2 verified, exit 0** |

## 分层归位

| 契约逻辑 | 归到哪层 | 落点 | 理由 |
|---|---|---|---|
| session-isolation（防串号）| safety（不变量类）| **独立 session.intent** | 不能混进 invariants.intent 的 2FA 域——vcgen 按参数名把 safety 注入同文件所有 intent，混域会误判 |

## 剥离的 D2 约束（登记到 RFC/SLO，不进 intent）

- 校验延迟 p95 ≤ 50ms → RFC § Quality Attributes，压测守护。
- 即时撤销「生效时延」→ e2e 测试（RejectRevokedSession 只验逻辑上 revoked ⇒ invalid）。

## 四条硬规则审查

- [x] **规则1（后态 primed）**：产出 `r` 全用 primed；safety 用 `r.valid'` 等后态形式。
- [x] **规则2（一文件一作用域）**：本文件只放 session 校验域，独立于 2FA / acceptance。
- [x] **规则3（无 frame）**：`s` 是只读输入，safety 与 ensure 对 `s` 用 unprimed，
      不引入 `s.*'` 自由变量（否则会误 FAIL）；产出 `r` 三字段都被 ensure 显式约束，无自由量。
- [x] **规则4（纯 require+ensure 才 trivial）**：本文件有 safety + primed 不变量 → 非平凡，
      Z3 真正生成并求解 VC。

## 去重 / 冲突检查

- 与 invariants.intent（2FA 域）：类型 `Session`/`SessionResolution` 不与 `User` 重名，无冲突。
- 与 acceptance.intent（含 AdminRevokeTargetSession）：本文件不重复吊销操作语义，
  只补「校验解析」的隔离不变量，互补不重叠。

## 验证结果

```
$ intent check products/auth/intents/session.intent
  ✅ intent VerifySession — verified
  ✅ intent RejectRevokedSession — verified
  exit 0
```

反向验证（注入串号 `ensure r.resolvedUserId' == s.accountId`）：
**FAILED + 反例（s.accountId=1 ≠ s.userId=0）** —— 证明 safety 真在守护。

## 合并计划

- [x] 新建 `products/auth/intents/session.intent`
- [x] `contracts.intent` 的 goal 注释解锁 [Awaiting ADR-0003] → [ADR-0003 Accepted]，
      并指向 session.intent
- [x] 全 auth intents 复跑：13 verified + 1 skipped + 0 failed

## 检查清单

- [x] 分层归位有依据（作用域约定）
- [x] D2 约束已剥离并登记去向
- [x] 四条硬规则逐条审查
- [x] 去重 / 冲突已查
- [x] 交付前 intent check 自验通过
