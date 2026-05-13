# intent-coder

一个 popsicle 模块，帮你把**遗留代码库迁移到 IDD（Intent-Driven Development，意图驱动开发）工作流**里：spec 驱动、intent 验证、活文档持续更新。

本模块站在 IDD 工作流的最前端，连接「我有一坨老代码」和「我准备好按 spec-and-verify 工作了」之间的鸿沟——先**抽取事实**（这份代码今天到底在干什么）、再**用形式化 intent 把闸**（拦住不一致的下游 spec）、再**持续保活产品/技术文档**（让它们随代码演进而不腐烂）。

本模块**自带** `.intent` DSL 与 Z3 验证能力，**自带**产品讨论 + PRD 起草这一对前段技能（含 PDR + acceptance.intent 种子产出）。模块的剩余「契约位」（架构辩论 / RFC / ADR writer / intent spec writer 等）保持开放，你自己挑或自己写。

## v0.3 范式升级：从功能树到任务图

`prd-writer` 起草产物从「单一 PRD 文件 + Feature 列表」升级为**任务图**——PRD 拆解为按 **5 个固定用户旅程阶段**（`onboarding` / `daily-ops` / `troubleshooting` / `admin` / `lifecycle`）组织的 task chunk，每个 task 一份独立 `.md` 文件，自带 YAML frontmatter（audience / journey_stage / prerequisites / query_anchors 等），可被 AI Copilot 独立召回精准回答用户问题。

这是 AI 时代产品文档的「极致结构化 + 以任务和意图为中心 + Chunking-friendly」范式落地。详见 [`skills/prd-writer/references/task-organization.md`](skills/prd-writer/references/task-organization.md)。

---

## Skills

| Skill | 状态 | 用途 |
|-------|------|------|
| **project-init** | ✅ 已交付 | 给新仓库铺骨架：把 legacy pin 成 git submodule、为每个 product 铺 4 件套目录（`PRODUCT.md` / `ARCHITECTURE.md` / `intents/` / `decisions/` + `proposals/`）、把 doc-architecture charter 落地到 `docs/CHARTER.md` |
| **fact-extractor** | ✅ 已交付 | 读遗留代码，输出结构化事实基（dependency graph、public-API contracts、unsafe/risk 清单、tech-debt 清单）——下游所有 writer 都消费这份事实 |
| **product-debate** | ✅ 已交付 | 多角色产品辩论模拟器：用 4-6 个角色（PM / UXR / GROWTH / ENGLD / BIZ）就一个 product slice 充分辩论方案空间。**消费 fact-extractor 的事实基**作为辩论 ground truth。**v0.3：Phase 4 PM 强制做 task 识别 + intent 层归类 + User Intents Catalog 起草**。产出 task-centric PRD 草稿 + 决策矩阵 + 辩论纪要，喂给 prd-writer |
| **prd-writer** | ✅ 已交付（v0.2 任务图） | 把辩论产出（或直接需求）打磨成 IDD 任务图五件套：**PRD overview**（PRODUCT.md 顶层增量）+ **N 份 task 文件**（按 5 个旅程阶段归类）+ **tasks/README.md**（索引）+ **acceptance.intent 种子**（单文件多 block，与 task_id 双射）+ **PDR 骨架**（Consequences 精确到文件级）。强制贯彻 charter 四条铁律 + AI 时代任务图范式，质量评分（5 维度 100 分）≥ 90 才放行 |
| **intent-consistency-check** | ⏳ 规划中 | intent-coder 自带的 Z3 闸：跑 `intent --format json check`，把 SMT 判决转成 pipeline 的 stage gate |
| **living-doc-author** | ⏳ 规划中 | 重跑那些上游产物（代码、intent、ADR）已变但 `PRODUCT.md` / `ARCHITECTURE.md` 没跟上的章节，保证活文档永远代表「现在」 |

这 6 个 skill 的依赖顺序是刻意安排的：

1. `project-init` —— 仓库出生时跑一次，铺出后续所有 skill 写入的目录舞台。
2. `fact-extractor` —— 在 pinned 的 legacy submodule 上跑，产出证据基。
3. `product-debate` —— 在一个 product slice 上跑，输入 fact-extractor 的事实基，输出辩论产物。
4. `prd-writer` —— 输入 product-debate 的产物（或直接用户需求），输出三联体（PRD + intent 种子 + PDR）。
5. `intent-consistency-check` —— 在 intent 种子被合并到正式 `.intent` 文件后跑（前置依赖：已有 intent spec writer 把种子收紧）。
6. `living-doc-author` —— 在首个迁移切片完成后跑，保活下游文档。

## Pipelines

| Pipeline | 用途 |
|----------|------|
| **migration-bootstrap**（规划中） | `project-init → fact-extractor → product-debate → prd-writer → 【你的 arch-debate / RFC / ADR writer】→ 【你的 intent spec writer】→ intent-consistency-check`。每个 legacy 代码库跑一次。中间技术侧的 writer 阶段 intent-coder 不内置——见下方「与外部 writer 集成」。 |

## 使用

在新项目里：

```bash
mkdir new-project && cd new-project && git init
popsicle init
popsicle module add /path/to/intent-coder

# 如果你已经有自己的 arch-debate / RFC / ADR / intent spec writer 模块，
# 也一并 add 进来：
# popsicle module add /path/to/your-tech-writers

# 1. 铺出文档骨架（交互式 —— 命名 products、挑首个迁移切片）
popsicle skill start project-init

# 2. 通过审批后，扫遗留 submodule 抽事实
popsicle skill start fact-extractor --source legacy/<your-legacy-name>

# 3. 对首切片 product 开一场多角色产品辩论
popsicle skill start product-debate

# 4. 把辩论产出打磨成 PRD 三联体（PRD + acceptance.intent 种子 + PDR）
popsicle skill start prd-writer

# 5. 你自己的 arch-debate / RFC / ADR writer 收敛技术方案（intent-coder 不内置）

# 6. 你自己的 intent spec writer 把种子收紧成正式 .intent 文件（intent-coder 不内置）

# 7. 过 intent 闸（一旦上线）
# popsicle skill start intent-consistency-check

# 8. 跑完整 migration pipeline（上线后）
# popsicle pipeline run migration-bootstrap
```

## 与外部 writer 集成

intent-coder **只内置产品侧（PRD/PDR）+ 基础设施侧（骨架/事实/校验/活文档保活）**，技术侧（架构 / 模块契约 / intent 语法收紧）保持开放。具体边界：

- `PRODUCT.md` / `ARCHITECTURE.md` 的**骨架**由 `project-init` 铺出（含 `[TBD: needs archaeology]` 占位符）。
- `PRODUCT.md` 的**内容**由 `prd-writer` 产出（基于 `product-debate` 的辩论摘要）。
- `ARCHITECTURE.md` 的**内容**由**你的** RFC writer 填（intent-coder 不内置——通常配套一场 arch-debate）。
- `decisions/pdr/*.md` 的**目录结构**由 `project-init` 铺出，**内容**由 `prd-writer` 产出（PDR 骨架）。
- `decisions/adr/*.md` 的**目录结构**由 `project-init` 铺出，**内容**由**你的** ADR writer 填。
- `intents/acceptance.intent` 的**种子**由 `prd-writer` 产出（宽松 DSL），**最终语法收紧**由**你的** intent spec writer 完成。
- `intents/invariants.intent` 和 `intents/contracts.intent` 的**空壳**由 `project-init` 铺出，**内容**由你的 intent spec writer 填。
- intent-coder 负责**校验**所有 intent 之间是否一致（`intent-consistency-check` skill，规划中）。

intent-coder 对外部 writer 的唯一要求：写入的活文档（`PRODUCT.md` / `ARCHITECTURE.md`）每次 edit 都要带 Decision-Ref，这条由 doc-architecture charter 的「四条铁律」约束。本模块自带的 `prd-writer` 已经天然满足这条；外部 RFC/ADR writer 需要自己遵守。

## 为什么单独成一个模块

这 4 个 skill 都是**迁移期工具**，不是 IDD 日常工具。拆开成独立模块的好处：

- 想给一个全新项目上 IDD 时，**不用把迁移脚手架硬塞进日常 pipeline**；
- 遗留考古能力（fact-extractor）在任何代码库上都能复用，跟你最终用不用 `.intent` 无关；
- 这个边界让 IDD 契约更清晰：「migration-bootstrap 跑完后，你欠 1 套 PRD/RFC/ADR + 至少 1 个跑通的 `.intent` 文件」。

## License

MIT
