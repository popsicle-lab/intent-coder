---
artifact: intent-consistency-report
generated_by: intent-consistency-check
scope: products/auth/intents
mode: observe              # observe | gate —— skill 始终 observe；gate 是 CI 用 tool exit code 实现
last_updated: 2026-05-13
overall: pass
total: 14                  # 13 intent + 1 theorem
verified: 13
failed: 0
skipped: 1
consecutive_clean_runs: 1  # 截至本次，连续 overall=pass 的次数（读上次报告 +1；非 pass 归零）
gate_ready: false          # consecutive_clean_runs >= 3 且本次 overall=pass 时为 true
query_anchors:
  - "auth 的 intent 全绿吗？"
  - "session.intent 引入后破坏既有了吗？"
  - "什么时候能从 observe 升级成 gate？"
---

# Intent 一致性报告 — auth

> 由 intent-consistency-check 经 intent-validate tool（Z3）跑出（dogfood Phase 3 验证）。

## Summary

| 指标 | 值 |
|---|---|
| overall | **pass** |
| 文件数 | 4 |
| verified | 13 |
| failed | **0** |
| skipped | 1（已知工具限制）|
| consecutive_clean_runs | 1 |
| gate_ready | false（需连续 ≥3 次零失败）|

一句话结论：Phase 3 技术侧三件套新增的 `session.intent` 全 verified，**未破坏既有 11 个意图**，
auth 验证域整体 pass。

## Per-File Results

| 文件 | verified | failed | skipped |
|---|---|---|---|
| acceptance.intent | 9 | 0 | 0 |
| contracts.intent | 0（纯 goal，0 VC）| 0 | 0 |
| invariants.intent | 2 | 0 | 1 |
| session.intent ★新增 | 2 | 0 | 0 |

## Failures

无。

> 如何读反例（供后续参考）：counterexample 给出使某条 VC 不成立的变量赋值，
> 形如「字段=值」。定位时先看 safety/invariant 的前件是否被满足、后件哪个 primed 字段越界。

## Skipped

| 项 | 文件 | 原因 |
|---|---|---|
| theorem EmailUniqueness | invariants.intent | struct-typed quantifiers 尚未实现（工具限制，非失败）；待 intent-lang 支持后自动启用 |

## Disposition

本次 observe 结论：**放行**（0 failed）。skipped 项是工具能力边界，已在文件内注明意图，无需处置。

### Gate Readiness

| 判据 | 当前 | 达标？ |
|---|---|---|
| 本次 overall = pass | pass | ✅ |
| consecutive_clean_runs ≥ 3 | 1 | ❌ |
| **gate_ready** | **false** | —— |

还需连续 2 次零失败方可建议 CI 开闸。开闸后 CI 直接用 `intent-validate` tool 的 exit code 拦合并：

```yaml
# .github/workflows 片段（gate_ready=true 后启用）
- name: intent gate
  run: ./intent-lang/target/release/intent-cli check products/auth/intents/*.intent
  # 任一 FAILED → exit 1 → 阻断合并
```

## 检查清单

- [x] Summary / Per-File Results / Failures / Skipped / Disposition / Gate Readiness 齐全
- [x] consecutive_clean_runs / gate_ready 已据上次报告推算（首跑=1 / false）
- [x] skipped 原因已注明、与失败区分
