# Glossary — saas-demo

> 全局术语表。所有 PRD / task / intent 中出现的专有名词必须出自本文件。
> 在本文件中找不到的术语 = 凭空造词 = PRD 质量评分 § Domain Glossary 一致性扣分。
>
> Last-Updated: 2026-05-13
> Last-Decision-Ref: project-init bootstrap

---

## Identity / Auth 域

| 术语 | 定义 | 同义词（禁用）|
|---|---|---|
| **User** | 一个可以登录本 SaaS 的自然人，由唯一 `user_id` 标识 | account, person |
| **Account** | User 的归属容器；个人版 = 1 User / 1 Account；团队版 = N User / 1 Account | tenant, workspace |
| **Session** | 用户登录后获得的、绑定 IP + UA 的一次会话凭证；过期会自动失效 | login session（口语可用，文档统一 session） |
| **2FA** | Two-Factor Authentication；本 SaaS 实现为 TOTP（基于时间的一次性密码）| MFA（文档统一 2FA） |
| **OAuth Identity** | 来自外部 IdP（Google / GitHub 等）的身份凭证，绑定到 User | OIDC token |
| **Recovery Code** | 用户启用 2FA 时生成的一组一次性备用码，用于 2FA 设备丢失场景 | backup code |
| **Magic Link** | （**本 demo 不采用**——PDR-0001 已显式否决，保留术语用于追溯）| email link login |

## Billing 域

| 术语 | 定义 | 同义词（禁用）|
|---|---|---|
| **Plan** | 订阅档位；本 SaaS 固定 3 档：Free / Pro / Team | tier（口语可用，文档统一 plan） |
| **Subscription** | Account 与 Plan 的关联实例；含计费周期 / 状态 / 续费日 | sub |
| **Invoice** | Stripe 生成的一次扣费凭证；含金额 / 状态 / 期间 | bill, receipt |
| **Seat** | Team 版下一个 User 占用的计费单位；按月折算 | license slot |
| **Payment Method** | 用户挂载在 Stripe 的支付方式（卡 / 支付宝等）| card |
| **Prorate** | 计费周期内升降档时按剩余天数计算的差额 | proration |
| **Dunning** | 扣费失败后的重试 + 通知流程 | retry flow |
| **Webhook** | Stripe 主动推送的事件回调（payment_succeeded 等）| event |

## Admin 域

| 术语 | 定义 | 同义词（禁用）|
|---|---|---|
| **Admin** | 本 SaaS 内部运营人员（**非客户的 Account Admin**）| operator, support |
| **Account Admin** | 客户侧 Team 版 Account 的管理员 User（**非内部 Admin**）| team admin, owner |
| **Audit Log** | 任何「敏感操作」（吊销 session / 修改 plan / 删除 User）的不可篡改记录 | activity log |
| **Quota** | 某个能力的可量化使用上限（如 API 调用次数 / Seat 数）| limit |

## Intent-Lang 域

| 术语 | 定义 |
|---|---|
| **acceptance.intent** | 描述「用户可观察行为」的 intent 文件；与 task 双射 |
| **invariants.intent** | 描述「领域永恒为真」的 intent 文件；跨 task |
| **contracts.intent** | 描述「模块间接口」的 intent 文件；跟随 ADR |
| **Z3 闸** | intent-consistency-check skill 跑的 SMT 求解过程，0 表示通过 |

## v0.3 任务图域

| 术语 | 定义 |
|---|---|
| **Task** | 一个独立的用户意图，能用一句完整用户原话讲出来；落地为 `tasks/{stage}/T-XXXX-*.md` 一份独立文件 |
| **Journey Stage** | 5 个固定的旅程阶段：onboarding / daily-ops / troubleshooting / admin / lifecycle |
| **Journey** | 跨 product 的用户旅程；落地为 `docs/user-journeys/J-XXXX-*.md` |
| **Query Anchor** | task 中「本 task 可解答」的用户原话问句；AI Copilot 的召回锚点 |
