---
Last-Updated: 2026-05-13
Last-Decision-Ref: PDR-0001
Product: auth
Audience: ["工程师", "AI agent"]
---

# Auth — 架构

> 实现视角的活文档。所有「为什么这么实现」类内容**不在本文件**，去对应 ADR。
>
> ⚠️ 本文件大量段落标 `[TBD: Awaiting ADR-XXXX]` —— 这是因为本 demo 故意没装
> arch-debate / rfc-writer 模块，演示「契约位」状态。真实项目里这些 ADR 落地后
> 才会回填本文件。

---

## Module 划分

| 模块 | 职责 | 状态 |
|---|---|---|
| `auth-api` | HTTP 入口（注册 / 登录 / 2FA / OAuth 回调）| `[TBD: Awaiting ADR-0001]` |
| `auth-core` | 业务逻辑（密码哈希 / 2FA 验证 / Session 签发）| `[TBD: Awaiting ADR-0002]` |
| `auth-store` | 持久化（User / Session / OAuth Identity 表）| `[TBD: Awaiting ADR-0003]` |
| `auth-audit` | Audit Log 写入（对接 admin-console）| `[TBD: Awaiting ADR-0004]` |

---

## 关键技术选型

> 标 `[TBD]` 的等 ADR；标了 ADR ID 的引用对应 ADR。

| 选型 | 决策 | 来源 |
|---|---|---|
| 密码哈希算法 | `[TBD: Awaiting ADR-0001]` | —— |
| 2FA TOTP 实现库 | `[TBD: Awaiting ADR-0002]` | —— |
| Session 存储 | `[TBD: Awaiting ADR-0003]` | —— |
| OAuth 客户端实现 | `[TBD: Awaiting ADR-0004]` | —— |
| Audit Log 存储 | `[TBD: Awaiting ADR-0005]` | —— |

---

## 模块间接口（contracts.intent 形式化版）

详见 [`intents/contracts.intent`](intents/contracts.intent)。当前大部分 block 标
`[Awaiting ADR-XXXX]`。

---

## 跨 product 集成点

| 对方 product | 集成方式 | 状态 |
|---|---|---|
| `billing` | 监听 `user.signed_up` 事件以创建 Free 订阅 | `[TBD: Awaiting ADR-XXXX]` |
| `billing` | 提供 `verify_session(token) -> user_id` 同步 RPC | `[TBD: Awaiting ADR-XXXX]` |
| `admin-console` | 提供 `list_sessions(user_id)` / `revoke_session(session_id)` RPC | `[TBD: Awaiting ADR-XXXX]` |
| `admin-console` | 推送 Audit Log 事件 | `[TBD: Awaiting ADR-XXXX]` |

---

## 性能与可用性目标

> 这些数字来自 PRD § Success Metrics；实现层面如何达成由 ADR 决定。

| 指标 | 目标 | Decision-Ref |
|---|---|---|
| 登录 API P95 延迟 | < 200ms | PDR-0001 |
| 2FA 验证 P95 延迟 | < 100ms | PDR-0001 |
| OAuth 回调 P95 延迟 | < 500ms（含外部 IdP 调用）| PDR-0001 |
| 服务可用性 | 99.9% (月)| PDR-0001 |

---

## 安全要求

详细 invariants 见 [`intents/invariants.intent`](intents/invariants.intent)：

- `no-plaintext-password`：密码任何时刻都不以明文形式存在（含日志 / 错误返回 / DB）
- `session-isolation`：不同 User 的 Session 不可互相伪造
- `email-uniqueness`：一个 email 全局对应一个 User
- `2fa-required-after-onboarding`：用户启用 2FA 后不可关闭（除非走「2FA 设备丢失」恢复流程）

---

## 当前已知技术债

- 暂无（项目刚启动）

---

`Decision-Ref: PDR-0001`
