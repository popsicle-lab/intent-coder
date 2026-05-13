# saas-demo — intent-coder v0.3 端到端示例

这是 [intent-coder](../../README.md) v0.3 任务图范式的**完整可读样本**：一个典
型 SaaS 应用的早期 IDD 工作流产出，含登录、订阅计费、管理后台三个 product。

本项目**没有真实代码**——所有 `.md` / `.intent` / `.yaml` 文件都是 intent-coder
四个核心 skill（`project-init` / `product-debate` / `prd-writer`）按 IDD charter
落地的「活文档」演示。任何想看「v0.3 任务图范式在真实项目里长什么样」的读者，
都可以直接读本目录。

---

## 项目快照

| 字段 | 值 |
|---|---|
| 项目名 | `saas-demo` |
| Products | `auth`（登录注册）/ `billing`（订阅计费）/ `admin-console`（管理后台）|
| **首切片** | `auth` —— 它是 onboarding 必经，反向依赖少 |
| Issue key 前缀 | `SAAS` |
| Greenfield？ | 是（无 legacy submodule）|
| License | MIT |

---

## 目录导览

```
saas-demo/
├── README.md                          # 你正在读这一份
├── module.yaml                        # popsicle 模块元数据
├── CONTRIBUTING.md                    # IDD 工作流贡献指南（人 + AI 共读）
├── PROJECT-INIT-PLAN.md               # project-init skill 的产出留档
│
├── docs/                              # 跨 product 层
│   ├── CHARTER.md                     # ⭐ 文档体系四条铁律（不可妥协）
│   ├── PROJECT_CONTEXT.md             # 项目全景上下文
│   ├── glossary.md                    # 全局术语表
│   ├── invariants/README.md           # 全局 invariants 索引（[TBD]）
│   ├── user-journeys/                 # ⭐ v0.3 任务图：跨 product 旅程的全局层
│   │   ├── README.md
│   │   └── J-0001-new-user-signup-and-first-payment.md
│   └── baseline/                      # fact-extractor 基线快照位置（greenfield 暂空）
│
├── products/                          # 三个 product，每个一份「4+1 件套」
│   ├── auth/                          # ⭐ 首切片，完整填充
│   │   ├── PRODUCT.md                 # 顶层索引（Tasks Catalog + Intents Catalog）
│   │   ├── ARCHITECTURE.md            # 技术活文档（含若干 [TBD]）
│   │   ├── tasks/                     # ⭐ v0.3 任务图：5 个固定旅程阶段
│   │   │   ├── README.md
│   │   │   ├── onboarding/            # T-0001, T-0002
│   │   │   ├── daily-ops/             # T-0010, T-0011
│   │   │   ├── troubleshooting/       # T-0020, T-0021
│   │   │   ├── admin/                 # T-0030
│   │   │   └── lifecycle/             # T-0040
│   │   ├── intents/
│   │   │   ├── acceptance.intent      # 单文件，与 task_id 双射的 block
│   │   │   ├── invariants.intent
│   │   │   └── contracts.intent       # 大部分待 ADR-XXXX 填
│   │   ├── decisions/
│   │   │   ├── pdr/PDR-0001-auth-strategy.md
│   │   │   └── adr/                   # 空，留给 arch-debate
│   │   └── proposals/{4 阶段桶}
│   │
│   ├── billing/                       # 部分填充（演示跨 product 协调）
│   │   └── ...（同 auth 结构，task 较少）
│   │
│   └── admin-console/                 # 仅占位（演示 scaffold-only）
│       └── ...（每个文件都是 [TBD: needs archaeology]）
│
└── migration/                         # 迁移期工具（greenfield 几乎空）
    ├── slices/
    ├── traceability.md
    └── progress.md
```

---

## 阅读路径建议

### 想看任务图范式怎么落地 → 看 auth

1. [`products/auth/PRODUCT.md`](products/auth/PRODUCT.md) —— 顶层概述 + User Intents Catalog
2. [`products/auth/tasks/README.md`](products/auth/tasks/README.md) —— 5 个旅程阶段的 task 列表
3. [`products/auth/tasks/onboarding/T-0001-first-time-signup-with-email.md`](products/auth/tasks/onboarding/T-0001-first-time-signup-with-email.md)
   —— 单个 task chunk 的标准长相
4. [`products/auth/intents/acceptance.intent`](products/auth/intents/acceptance.intent)
   —— acceptance block 与 task_id 双射演示

### 想看 PDR + Charter 合规 → 看 PDR

5. [`products/auth/decisions/pdr/PDR-0001-auth-strategy.md`](products/auth/decisions/pdr/PDR-0001-auth-strategy.md)
   —— Consequences 精确到文件级；Intent Impact 三层覆盖完整

### 想看跨 product 旅程 → 看 Journey

6. [`docs/user-journeys/J-0001-new-user-signup-and-first-payment.md`](docs/user-journeys/J-0001-new-user-signup-and-first-payment.md)
   —— 从注册到首次付费跨 auth + billing 的 6 个 stage

### 想看 charter 怎么写 → 看 CHARTER

7. [`docs/CHARTER.md`](docs/CHARTER.md) —— 四条铁律 / Layer Map / 反模式

---

## 这个 demo 跟过哪些 skill

| Skill | 产出物 | 落在哪 |
|---|---|---|
| `project-init` | 完整目录骨架 + CHARTER + PROJECT-INIT-PLAN | 仓库根 + `docs/CHARTER.md` + `PROJECT-INIT-PLAN.md` |
| `fact-extractor` | （未跑——greenfield 项目没有 legacy 源码可扫）| —— |
| `product-debate` × 2 | auth 辩论纪要 + billing 辩论纪要 | `migration/slices/*-product-debate.md` |
| `prd-writer` × 2 | auth 五件套 + billing 五件套 | `products/auth/**` + `products/billing/**` |

注：本 demo 没有 `.popsicle/artifacts/<run-id>/` 这一层暂存目录——所有产出都直
接落到了它们最终该去的位置（这是 PDR Accepted 之后的状态）。真实工作流里
prd-writer 会先把产出放进 `.popsicle/artifacts/`，PDR Accepted 才合并到这里。

---

## 这个 demo **不**做什么

为了控制篇幅，本 demo **省略**了：

- ❌ 真实代码（`crates/` / `src/` 等都不存在）—— 演示只到 spec 层
- ❌ 完整的辩论 transcript（只留摘要纪要）—— 真实辩论应该几千字
- ❌ admin-console 的 task 内容（保留占位 `[TBD]`）—— 演示 scaffold-only 状态
- ❌ ADR 文件（演示「contracts.intent 等 ADR-XXXX」的 hand-off 状态）

---

## 重跑 demo

如果你想用 intent-coder 重新生成本 demo，跑：

```bash
cd example/saas-demo
popsicle init
popsicle module add ../..
popsicle skill start project-init   # 输入 demo 的 Product Inventory
popsicle skill start product-debate # 对 auth 跑一次
popsicle skill start prd-writer     # 把辩论产出升级为五件套
popsicle skill start product-debate # 对 billing 跑一次
popsicle skill start prd-writer     # 同上
```

---

## License

MIT (跟随 intent-coder 主仓库)
