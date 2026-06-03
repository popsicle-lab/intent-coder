---
artifact: rfc
slug: session-storage
title: "Session 存储与校验（Redis 集中式）"
target_product: auth
status: Proposed
generated_by: rfc-writer
date: 2026-05-13
related_adrs: [ADR-0003]
related_prd: "auth.prd.md"
quality_score: 92
query_anchors:
  - "session 为什么选 Redis 集中式？"
  - "verify_session 的契约是什么？"
  - "session-isolation 怎么保证？"
---

# RFC: Session 存储与校验（Redis 集中式）

> 由 rfc-writer 从 arch-debate 的 rfc-draft 打磨而来，质量评分 92/100。现在时书写。

## Context

billing / admin-console 每次鉴权都依赖 auth 校验 session。`crates/auth-store` 已依赖
Redis（fact-extraction-report § Bounded Contexts）。本 RFC 定下 session 的存储与校验
契约，满足即时撤销（D5，见 acceptance.AdminRevokeTargetSession）与防串号。

## Goals

- 对外单一校验入口 `verify_session`，下游不得直接读 store。
- session 与归属 user 强绑定，校验**只**解析出该 session 真正归属的 user（session-isolation）。
- 支持 admin 即时吊销。

## Non-Goals

- 跨区域 session 复制；OAuth 外部 token 映射（后续 RFC）。

## Quality Attributes (NFR)

> 不进 contracts 种子（D2）。

| 属性 | 目标 | 守护手段 |
|---|---|---|
| 校验延迟 p95 | ≤ 50ms | 压测断言 |
| 撤销生效 | 即时 | e2e 测试 |

## Proposed Design

token → Redis 查 `{user_id, account_id, expires_at, revoked}`。
`verify_session(token) -> Option<UserContext{user_id, account_id}>`：命中且未过期未撤销
→ `Some(...)`，且返回的 `user_id` 恒等于该 session 持久化的归属 user；否则 `None`。

## Alternatives Considered

| 方案 | 否决理由 |
|---|---|
| B JWT 无状态 | 撤销需黑名单，退回有状态，违背即时撤销 NFR |
| C DB 表 | 热路径打 DB，延迟最差 |

详见 `../exploring/session-storage.tech-decision-matrix.md`。

## Intent & Decision Mapping

| 核心技术声明 | 目标 intent 层 | 决策载体 | contracts goal | 备注 |
|---|---|---|---|---|
| 校验只解析出 session 真正归属的 user | `invariants.intent`（session 域）| ADR-0003 | "auth 对外提供可信的会话校验" | 收紧为 safety + primed |
| 下游只能走 verify_session | `contracts.intent` | ADR-0003 | 同上 | goal 登记 |
| 校验 p95 ≤ 50ms | （不进 intent）| RFC § NFR | —— | D2 |

## Risks & Mitigations

| 风险 | 触发条件 | 缓解 |
|---|---|---|
| Redis 单点 | 主节点故障 | HA + 哨兵；校验 fail-closed |

## Migration / Rollout

双读 → 切写 → 下线旧路径。回滚 = 切回旧 store 读。

## File Manifest

### ARCHITECTURE.md 顶层增量
- [ ] `products/auth/ARCHITECTURE.md` § Session Layer — Redis 集中式 store + verify_session 契约

### Intent 文件
- [ ] `products/auth/intents/contracts.intent` 的 goal "auth 对外提供可信的会话校验"
      解锁（[Awaiting ADR-0003] → Accepted）
- [ ] `products/auth/intents/session.intent`（新建）追加 session-isolation safety
      （待 ADR-0003 解锁后由 intent-spec-writer 收紧）

### 决策记录
- [ ] `products/auth/decisions/adr/ADR-0003-session-storage.md`（Proposed → adr-writer 固化）

## Quality Checklist

- [x] 四维度已评分，总分 92 ≥ 90
- [x] `session-storage.contracts.intent` 能 `intent check`（goal 块合法、0 VC）
- [x] 无性能/时延误塞进 contracts（D2）
- [x] File Manifest 与 ADR Consequences 镜像一致
- [x] Intent & Decision Mapping 每行都有决策载体

## References

- **Source RFC Draft**: `../exploring/session-storage.rfc-draft.md`
- **Source Debate**: `../exploring/session-storage.arch-debate.md`
- **Decision Matrix**: `../exploring/session-storage.tech-decision-matrix.md`
