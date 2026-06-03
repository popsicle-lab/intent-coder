---
artifact: rfc-draft
slug: session-storage
topic: "auth session 存储与校验"
target_product: auth
status: draft
generated_by: arch-debate
date: 2026-05-13
adr_candidates: [ADR-0003]
cadr_candidates: []
---

# RFC Draft: Session 存储与校验（Redis 集中式）

> arch-debate 的辩论摘要草稿，非终态 RFC。

## Context

billing / admin-console 每次鉴权都依赖 auth 校验 session。现有 `crates/auth-store`
已依赖 Redis（F-1）。需定下 session 的存储与校验契约，满足即时撤销（D5）与防串号。

## Goals / Non-Goals

**Goals**
- 对外单一校验入口 `verify_session`，下游不得直接读 store。
- session 与归属 user 强绑定，校验**只**解析出该 session 真正归属的 user。
- 支持 admin 即时吊销（置 revoked / 删 key）。

**Non-Goals**
- 不做跨区域 session 复制（后续 RFC）。
- 不在本 RFC 处理 OAuth 外部 token 映射。

## Quality Attributes（NFR）

| 属性 | 目标 | 守护手段 |
|---|---|---|
| 校验延迟 p95 | ≤ 50ms | 压测断言（不进 .intent，D2）|
| 撤销生效 | 即时（下次校验即失效）| e2e 测试 |

## Proposed Design

token → Redis 查 `{user_id, account_id, expires_at, revoked}`。
`verify_session(token)`：命中且未过期未撤销 → `Some(UserContext{user_id, account_id})`，
且 `user_id` 恒等于该 session 持久化的归属 user；否则 `None`。

## Alternatives Considered

| 方案 | 否决理由 |
|---|---|
| B JWT 无状态 | 撤销需黑名单，退回有状态，违背即时撤销 NFR |
| C DB 表 | 热路径打 DB，延迟最差 |

详见 `session-storage.tech-decision-matrix.md`。

## Intent & Decision Mapping ★

| 核心技术声明 | 目标 intent 层 | 决策载体 | contracts goal | 备注 |
|---|---|---|---|---|
| 校验只解析出 session 真正归属的 user（防串号）| `invariants.intent`（session 域）| ADR-0003 | "auth 对外提供可信的会话校验" | 收紧为 safety + primed |
| 下游只能走 verify_session | `contracts.intent` | ADR-0003 | 同上 | goal 登记 |
| 校验 p95 ≤ 50ms | （不进 intent）| RFC § NFR | —— | D2：压测守护 |

**ADR 候选清单**：ADR-0003（Session Storage = Redis 集中式）
**CADR 候选清单**：（无——不触及 charter 铁律 / Layer Map）

## Risks & Mitigations

| 风险 | 触发条件 | 缓解 |
|---|---|---|
| Redis 单点 | 主节点故障 | HA + 哨兵；校验失败 fail-closed |

## Migration / Rollout

灰度：双读（旧 + 新 store）→ 切写 → 下线旧路径。回滚 = 切回旧 store 读。

## Open Questions

- [ ] expires_at 的滑动续期策略（留给 rfc-writer / 后续）。

## References

- **Source Debate**: `session-storage.arch-debate.md`
- **Decision Matrix**: `session-storage.tech-decision-matrix.md`
- **Fact Basis**: fact-extraction-report § Bounded Contexts（Redis 依赖）
