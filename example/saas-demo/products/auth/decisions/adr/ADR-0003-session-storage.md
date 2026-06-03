# ADR-0003: Session 存储采用 Redis 集中式 store

> **Status**: Accepted
> **Date**: 2026-05-13
> **Target Product**: `auth`
> **Decision Type**: Architecture Decision Record (ADR)
> **Supersedes**: ——
> **Related PDRs**: `PDR-0001`
> **Related RFC**: `proposals/proposed/session-storage.rfc.md`
> **Related Journey**: `J-0001`

---

## Decision Context

### 触发因素

billing / admin-console 每次鉴权都依赖 auth 校验 session，但 session 存储形态一直
`[Awaiting ADR-0003]`，导致 contracts.intent 的「会话校验」契约无法收紧。本 ADR 定形以解锁。

### 多角色辩论摘要

继承 `proposals/exploring/session-storage.arch-debate.md`。

**参与角色**: ARCH, SEC, PERF, OPS, DEV
**关键分歧**: SEC（即时撤销）vs PERF（最低延迟）→ 撤销是硬 NFR，JWT 出局。
**核心事实引用**: F-1 `crates/auth-store` 已依赖 Redis。

### 备选方案

| 方案 | 否决理由 |
|------|---------|
| JWT 无状态 | 撤销需黑名单，退回有状态，违背即时撤销 NFR |
| DB session 表 | 热路径打 DB，延迟最差 |

---

## Decision

auth 采用 Redis 集中式 session store；对外只通过
`verify_session(token) -> Option<UserContext{user_id, account_id}>` 暴露校验，下游不得
直接读 store；session 与归属 user 强绑定，校验只解析出该 session 真正归属的 user。

---

## Consequences

### ARCHITECTURE.md Updates
- [ ] `products/auth/ARCHITECTURE.md` § Session Layer — Redis 集中式 store + verify_session 契约

### Intent Updates
- [x] `products/auth/intents/contracts.intent` 的 goal "auth 对外提供可信的会话校验"
      解锁（[Awaiting ADR-0003] → [ADR-0003 Accepted]）
- [x] `products/auth/intents/session.intent`（新建）追加 session-isolation safety
      （由 intent-spec-writer 收紧）

### Code Updates (informational, not enforced)
- 模块 `crates/auth-store/`：实现 Redis session store + `verify_session`

### Risk Side-Effects
| Risk | 触发条件 | 缓解 |
|------|---------|------|
| Redis 单点 | 主节点故障 | HA + 哨兵；校验 fail-closed |

---

## Intent Impact

| Intent 层 | 修改类型 | 涉及 block | 备注 |
|-----------|---------|----------|------|
| `intents/contracts.intent` | 解锁 | goal "auth 对外提供可信的会话校验" | 已登记 [ADR-0003 Accepted] |
| `intents/session.intent` | 新增 | safety `SessionResolvesToOwner` + intent `VerifySession` | intent-spec-writer 已收紧 |
| `docs/invariants/*.intent` (全局) | 无影响 | —— | 不触及全局，无需 CADR |

> 不触及 charter 四铁律 / Layer Map → 普通 ADR 即可。

---

## Validation Plan

### Contracts 验证
- `intent check products/auth/intents/session.intent`：`SessionResolvesToOwner` verified。

### 质量属性验证（NFR）
- 校验 p95 ≤ 50ms（压测，D2：不进 intent）。

### 回滚条件
若 Redis 校验路径稳定性劣化，回滚到双读旧 store。回滚新建 ADR 标 `Supersedes: ADR-0003`，
**不修改本 ADR**。

---

## Approval

- **Status**: Accepted
- **Approved by**: dogfood（adr-writer 固化演示）
- **Approval date**: 2026-05-13

---

## References

- **Source RFC**: `proposals/proposed/session-storage.rfc.md`
- **Source Debate**: `proposals/exploring/session-storage.arch-debate.md`
- **Decision Matrix**: `proposals/exploring/session-storage.tech-decision-matrix.md`

---

*本 ADR 由 rfc-writer 起草为 Proposed，adr-writer 固化为 Accepted。Charter 第 2 条
铁律：Accepted 之后永不修改；纠正错误请新建 ADR 并标注 Supersedes。*
