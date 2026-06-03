---
artifact: arch-debate-record
slug: session-storage
topic: "auth 如何存储与校验 session，保证跨产品鉴权可信且 session 不串号"
participants: [ARCH, SEC, PERF, OPS, DEV]
confidence: 4
date: 2026-05-13
query_anchors:
  - "session 为什么选 Redis 集中式而不是 JWT？"
  - "session-isolation 是怎么保证的？"
  - "这个决策走 ADR-0003 吗？"
---

# 架构辩论纪要 — session-storage

> 由 `arch-debate` skill 生成（dogfood 实跑）。来源：PRD-0001 / auth contracts.intent
> 的 "auth 对外提供可信的会话校验" `[Awaiting ADR-0003: Session Storage]`。

## Topic

auth 如何存储与校验 session，使 billing / admin-console 的每次鉴权都可信、可即时撤销，
且**绝不串号**（一个 session 只能解析出它真正归属的 user）。来源：prd-overview §
Intent Mapping → contracts 行「auth 对外提供可信的会话校验」。

## Participants

| 角色 | 立场速写 |
|---|---|
| ARCH | 契约要单一：下游只能走 `verify_session`，不得直接读 store |
| SEC | 必须支持即时撤销 + 防串号；JWT 撤销难是硬伤 |
| PERF | 校验在每次鉴权热路径上，p95 要低 |
| OPS | 要可观测、可回滚；集中式 store 是单点要有 HA |
| DEV | Redis 团队已在用，JWT 轮换密钥的实现成本被低估 |

用户置信度：4/5

## Phase 1 — 技术问题 + 质量属性（NFR）

- 问题：session 存哪、怎么校验、怎么撤销、怎么防串号。
- 硬约束：必须支持 admin 即时吊销 session（D5，见 acceptance.AdminRevokeTargetSession）；
  不得绕过 `verify_session`。
- 质量属性优先级：**安全（撤销 + 防串号）> 校验延迟 > 可运维 > 实现成本**。
- 事实基：F-1 现有 `crates/auth-store` 已依赖 Redis（fact-extraction-report §
  Bounded Contexts）；F-2 admin 吊销路径已在 acceptance.intent 形式化。

## Phase 2 — 方案发散

- **方案 A — Redis 集中式 session store**（ARCH/DEV 提）：token → Redis 查
  `{user_id, account_id, expires_at, revoked}`。撤销 = 置 revoked / 删 key。
- **方案 B — 无状态 JWT**（PERF 提）：token 自带签名 claims，校验无需查库，延迟最低。
- **方案 C — 数据库 session 表**（DATA 视角，DEV 代述）：强一致，但热路径打 DB。

## Phase 3 — 多角色评审

| 方案 | SEC | PERF | OPS | DEV |
|---|---|---|---|---|
| A Redis | 即时撤销✓ 防串号✓（key 绑 user）| 查 Redis ~1ms，OK | 需 HA，但已有 | 复用现有，低 |
| B JWT | **撤销难**（要黑名单，又退回有状态）| 最低延迟 | 无状态易扩 | 密钥轮换成本被低估 |
| C DB 表 | 撤销✓ | 热路径打 DB，最差 | 运维重 | 中 |

## Phase 4 — 收敛与决策

- ARCH 综合：安全（即时撤销 + 防串号）是首要 NFR，JWT 的撤销硬伤直接出局；
  Redis 在延迟与复用现有基建上都优于 DB 表。
- 角色投票：A ×4（ARCH/SEC/OPS/DEV）、B ×1（PERF，但认可撤销硬伤）。
- **用户最终决策**：采用方案 A（Redis 集中式），立 ADR-0003。

## Decision

auth 用 Redis 集中式 session store；对外只通过 `verify_session(token) -> Option<UserContext>`
暴露校验，下游不得直接读 store；session 与其归属 user 强绑定，校验结果只解析出该
session 真正归属的 user（session-isolation）。

## 关键分歧

- SEC vs PERF：即时撤销能力 vs 最低延迟 → 收敛到 A（撤销是硬 NFR，延迟 Redis 可接受）。

## 用户决策点

- [x] 用户决策与多数角色一致（未覆盖），置信度 4。

## 下游接驳建议

- rfc-writer：把本纪要 + rfc-draft 打磨成正式 RFC + contracts 种子 + ADR-0003 骨架。
- 性能目标（校验 p95 ≤ 50ms）进 RFC § Quality Attributes，**不进 .intent**（D2）。

## Output Checklist

- [x] Phase 1-4 小结齐全
- [x] 关键分歧与各方立场已记录
- [x] 用户决策点已显式记录
- [x] 数字/模块名引用可追溯（F-1 Redis 依赖）
- [x] Topic 与另两份 artifact 一致
