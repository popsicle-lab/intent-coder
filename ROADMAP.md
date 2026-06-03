# intent-coder 到生产路线图

> 本文件是**活文档**（现状 + 计划），随进展更新；不写历史叙事，过期内容就地修正。
> 方向性决策固化在 §2，后续如需更严格可抽成独立 ADR。
>
> Last-Updated: 2026-06-02

---

## 1. 现状快照

### 1.1 已经有什么

| 资产 | 状态 | 位置 |
|---|---|---|
| **intent-lang** | ✅ 已进仓库（0.1.0，Rust + Z3 可运行）| `intent-lang/`（crates: `intent-syntax` / `intent-core` / `intent-cli`）|
| 4 个已实现 skill | ✅ | `skills/{project-init,fact-extractor,product-debate,prd-writer}/` |
| 任务图范式（v0.3）| ✅ | `prd-writer` 产出「五件套」 |
| 端到端 demo | ✅（但 `.intent` 语法待对齐）| `example/saas-demo/` |
| popsicle 工作流引擎 | ✅ | `popsicle/`（intent-coder 是它的一个 module）|

### 1.2 intent-lang 真实形态（关键事实）

- **可运行实现**：lexer + parser + 类型检查 + VC 生成 + Z3，**非纯文档**。
- **验证入口**：`intent check --format json <file>` → 调系统 `z3` → exit 0（全过）/ 1（任一失败，含反例）。
- **真实语法**：`type` / `enum` / `function` / `intent`（`require` / `ensure` / `invariant`）/ `safety` / `theorem` / `axiom` / `goal` / `coverage`，Hoare 风格 + 后置态 `x'`，支持 `forall` / `exists` / `==>`。生命周期标注 `@tobe` / `@asis`。
- **明确不支持**：时序逻辑（LTL/CTL）、状态机、时间/性能 DSL（`within` / `eventually` / `never` 等）。POSITIONING 建议这类需求用 TLA+。
- **成熟度**：Alpha（核心链路通，测试偏少，`intent generate` / `fmt` / `intent-llm` 未实现）。

### 1.3 核心矛盾

仓库里**并存两套互不兼容的 `.intent` 语法**：

| | intent-lang（真实）| intent-coder 产出（saas-demo + prd-writer 模板）|
|---|---|---|
| 顶层构造 | `intent` / `safety` / `theorem` / `goal` | `acceptance` / `invariant` / `contract` |
| 行为描述 | `require` / `ensure` / `invariant` | `given` / `when` / `then` |
| 时间/性能约束 | **不支持** | 大量使用（`5秒内`、`P95<200ms`、`within()`）|
| 能否被 `intent check` 解析 | ✅ | ❌ **一行都过不了** |

**能力边界错位**：intent-lang 擅长「逻辑一致性 / 不变量」；而 saas-demo 的 `acceptance.intent` 塞满了 intent-lang 明确拒绝的「时间性能约束」。

---

## 2. 关键决策（固化）

| ID | 决策 | 理由 | 否决的替代 |
|---|---|---|---|
| **D1** | **路线甲**：intent-coder 适配 intent-lang 真实语法，**不**扩展 intent-lang 去吃 IDD 的 `given/when/then` | 尊重 intent-lang 的 POSITIONING；工作量集中在改模板/demo，可控 | 路线乙（扩 intent-lang 加时序/时间算子）：违背其定位、工作量大、污染语言 |
| **D2** | 时间 / 性能类约束**降级为测试断言**，写进 task 文件「可观察的成功标志」，**不进 Z3** | intent-lang 不做时序；这类约束本就该由 benchmark / e2e 测试守，不是 SMT | 硬塞进 `.intent`：制造「假形式化」，Z3 根本消费不了 |
| **D3** | `intent-spec-writer`（种子 → 正式 intent）**从外部契约位收回内置** | intent-lang 已进仓库，这一步直接决定能否喂给 Z3，是闭环必经环节，不该外包 | 保持外部：闭环断在仓库外，无法 dogfood |
| **D4** | popsicle 简化（RFC D2）**解耦出主线、痛点驱动、延后**；现在只做 RFC 步骤 1（加 `ContentProvider`，零破坏）| RFC 本身是 dogfood 驱动；闭环验证优先级更高；切割是半年宽限的渐进重构 | 现在就砍：过早优化，停下闭环去重构平台 |

> 闭环的第一刀切在 **invariants 层**，不是 acceptance 层——因为 `invariants.intent` 正好是 intent-lang 的主场。

---

## 3. 三层 intent ↔ intent-lang 构造映射

| IDD 三层（intent-coder）| 映射到 intent-lang | 能否进 Z3 |
|---|---|---|
| `invariants.intent` | `safety` / `invariant` | ✅ 完美契合 |
| `contracts.intent` | `intent`（`require` / `ensure` 模块操作契约）| ✅ 契合 |
| `acceptance.intent`（逻辑类，如「2FA 启用后不可关闭」「recovery code 一次性」）| `intent` / `theorem` | ✅ 可映射 |
| `acceptance.intent`（时间/性能类，如「P95<200ms」「90 秒内」）| —— | ❌ 不进 Z3，降级为测试断言（D2）|

**路线甲对 saas-demo 的直接后果**——`acceptance.intent` 会「瘦身 + 分流」。以 auth 为例：

| 原 acceptance block | 处理 |
|---|---|
| `T-0001 signup ≤ 90s` / `T-0010 P95<200ms` / `T-0011 OAuth<500ms` | 移出 `.intent` → task「可观察的成功标志」当测试断言 |
| `T-0002 recovery codes = 10 且只显示一次` | 保留 → intent-lang `intent`/`ensure` |
| `T-0021 recovery code 一次性` | 保留 → intent-lang `intent` |
| `T-0010 错误信息不区分用户/密码` | 保留 → intent-lang `intent` |

结果：`acceptance.intent` 从「一堆伪形式化时间约束」收敛成「少量逻辑契约」，**invariants.intent 成为 Z3 验证主力**。

---

## 4. 主线：打通「意图 → 机器判定」闭环

### Phase 0 — 地基对齐 + 第一个真实 PASS/FAIL ⭐ 最高优先级

| # | 任务 | 产出 |
|---|---|---|
| 0.1 | 把 intent-lang 封装成 popsicle tool | `tools/intent-validate/tool.yaml`（`command: intent check --format json {{path}}`）|
| 0.2 | 新建最小 `intent-consistency-check` skill，调用 0.1 的 tool、解析 JSON、**先观察不阻塞** | `skills/intent-consistency-check/` |
| 0.3 | 用 intent-lang 真实语法重写 `auth/invariants.intent`，跑出第一个真实 PASS | 改写后的 `.intent` |
| 0.4 | 改 `prd-writer` 种子模板，今后产出 intent-lang 合法语法（invariants/contracts 层）；时间约束写进 task 备注 | `skills/prd-writer/templates/acceptance-intent-seed.intent` |

**验收**：
- `intent check` 在 `auth/invariants.intent` 上输出真实 PASS；
- 故意注入矛盾（如同时要求 `no-self-revoke` 和一条允许自我吊销的规则）→ 输出 FAIL + 反例；
- `prd-writer` 不再产出无法解析的伪形式化文本。

**✅ Phase 0 已完成（2026-05-13）**

| # | 产出 | 实证 |
|---|---|---|
| 0.1 | `tools/intent-validate/{tool.yaml,guide.md}` | 封装 `intent check`；PATH 无 `intent` 时回退仓库内 `intent-lang/target/release/intent-cli`，并校验 z3。渲染命令实跑通过 |
| 0.2 | `skills/intent-consistency-check/`（skill.yaml + guide + report 模板）| observe 模式：枚举 `.intent` → 调 tool → 解析 JSON → 出报告，不阻塞；guard 已对照 popsicle 源码核对 |
| 0.3 | 重写 `auth/invariants.intent` | `EnableTwoFactor` / `ResetVia2FARecovery` **verified**，`EmailUniqueness` theorem skipped，`exit 0` |
| 0.4 | 重写 `acceptance-intent-seed.intent` + 同步 quality-rubric | 实测 2 个 intent verified、exit 0；rubric 去掉 `within()` 等伪算子 |
| 验证 | 注入矛盾（2FA 启用却 `totpEnabled=false`）| **FAILED + Z3 反例** `u.totpEnabledAt=0, u.totpEnabled=false` |

**关键 dogfood 发现（已写进各 guide，后续 Phase 必须遵守）**：

1. **后态须 primed**：`safety`/`invariant` 子句写 unprimed 只验「旧态」→ 假通过。约束操作**结果**必须用 `x'`（见 `vcgen.rs`：`ensure` 进 assumes，safety goal 直接取用户写的表达式）。
2. **一文件 = 一验证作用域**：vcgen 把每条 `safety` **无条件注入**文件内所有 intent，且靠**参数名**绑定。不相关操作放各自文件，否则自由变量会误判 FAIL。→ 约定：`acceptance.intent` 只放操作 intent，保持型不变量进 `invariants.intent`。
3. **struct-typed `forall` theorem 当前 skipped**（未实现）：email/session 唯一性这类「双实体关系」暂只能**声明**不能**验证**，记入 intent-lang 待补清单。
4. **工具链坑**：tool 需 `popsicle tool install ./tools/intent-validate` 装到 `.popsicle/tools/`；`tool run --format json` 会**双层包裹**（intent JSON 在内层 `stdout`）；FAIL 时 `exit 1` 会让 `tool run` 报错，observe 模式须当数据处理。

### Phase 1 — 「种子 → 正式 intent」内置化

| # | 任务 |
|---|---|
| 1.1 ✅ | 新建 `intent-spec-writer` skill：把 prd-writer 的种子收紧成 intent-lang 合法语法（D3）|
| 1.2 ✅ | 完善 `intent-consistency-check`：反例展示、「观察 → 门禁」退出判据（连续 N 迭代零偏差才升级为闸门）|
| 1.3 ✅ | 把 saas-demo 全部 9 份 `.intent`（auth / billing / admin-console 三件套）迁到 intent-lang 真实语法 |

**✅ Phase 1 已完成（2026-05-13）**：

- **1.1 `intent-spec-writer`**（skill.yaml + guide + 2 模板）：承接 prd-writer 种子，做
  五件事——分层归位（acceptance/invariants/contracts）、剥离 D2 约束（登记到 task）、
  四规则审查、与现有 `.intent` 去重查冲突、交付前 `intent check` 自验。`formal-acceptance.intent`
  模板实跑 **verified、exit 0**。定位刻意保持薄：只做语法收紧与冲突检查，不发明语义。
- **1.2 `intent-consistency-check` 收紧**：明确 **observe = skill 行为 / gate = CI 行为**
  （gate 不是 skill 状态，而是 CI 跑 `intent-validate` tool 靠 exit code 拦合并）。
  新增量化退出判据 `Gate Readiness`：`consecutive_clean_runs >= 3` 且本次 pass → `gate_ready`，
  附 CI 开闸 YAML 片段；强化反例展示（要求写「哪个字段=什么值违反哪条约束」+ 常见根因）。
  guard 已加 `Gate Readiness` section 校验。

**✅ Phase 1.3 已完成（2026-05-13）**：saas-demo 9 份 `.intent` 全部真实语法、`intent check` 全绿——**17 intent verified，4 theorem skipped，0 failed**。分层落地：

- `acceptance.intent` = **操作后置规约**（require/ensure，trivial verified）：auth 9 + billing 3 + admin 1。
- `invariants.intent` = **Z3 真验证的安全不变量**（safety + primed + 完整 ensure）：auth「2FA 保持」、billing「降级清零 seats」、admin「跨账户查询必审计」各 1 条 verified。
- `contracts.intent` = **契约位**（`goal` 块声明意图 + 注释登记 `[Awaiting ADR]`，0 VC）。
- 聚合类（`single-active` / `no-double-charge` 的 `count`）与双实体关系（`prorate-symmetric` / 唯一性）→ struct-forall theorem，当前 **skipped**（intent-lang 待补）。
- 时间 / 性能 / 运行时事实（p95、retry 间隔、卡号不入库、明文密码不入日志）→ 全部剥离到 task「可观察的成功标志」（D2）。

**补充 dogfood 发现（已写进各文件头 + guide）**：

5. **无 frame 假设**：intent-lang **不**默认「未提及字段不变」。要声明操作不改某字段，必须显式 `ensure x' == x`，否则该 primed 字段自由 → 若被 invariant/safety 约束则必 FAIL。
6. **纯 require+ensure = trivial verified**：只有 `invariant`/`safety` 子句产生 goals，`ensure` 只是假设。所以「操作规约」本身不被证伪，真正的一致性验证来自 invariants 的 safety。
7. **不支持聚合**：无 `count` / `where`；集合基数约束只能转写成双实体关系 theorem，且当前 skipped。

### Phase 2 — 文档保活 + 编排

| # | 任务 |
|---|---|
| 2.1 ✅ | 新建 `living-doc-author` skill（多处模板已挂钩 `--target tasks-index` 等，防 doc-code drift）|
| 2.2 ✅ | 写 `migration-bootstrap` pipeline YAML，串联 4 + 新增 skill 成 DAG，结束手动逐个 `skill start` |

**✅ Phase 2 已完成（2026-05-13）**：

- **2.1 `living-doc-author`**（skill.yaml + guide + sync-report 模板）：保活/对账 skill，
  扫四类 doc-code drift（过期 / 断链 / 孤儿 / 未验证），按 `--target`
  （tasks-index / task-backrefs / last-verified / product-context / all）刷新活文档
  元数据、`tasks/README` 健康度、反向引用、`last_verified` 回填。红线：**只对账不创作正文**，
  正文 drift 转「待人工处置」走 prd-writer + PDR。
- **2.2 `migration-bootstrap` pipeline**（`pipelines/migration-bootstrap.pipeline.yaml`）：
  按 popsicle `PipelineDef` 真实 schema 串起全部 7 个自带 skill 成 DAG——
  `init → facts → debate → prd → intent-spec → intent-check → living-docs`，
  4 个审批点（init/debate/prd/living-docs）。已校验：字段合法、依赖指向存在的 stage/skill、
  拓扑无环。**关键变化**：`intent-spec-writer` Phase 1 起内置，不再是 bootstrap.md 里
  的外部契约位【】；只剩架构 ADR / `contracts.intent` 最终形式留给 Phase 3 外部 writer。

### Phase 3 — 技术侧 writer（已决定：内置）

| # | 任务 |
|---|---|
| 3.1 ✅ | `arch-debate` / `rfc-writer` / `adr-writer`：决定 `contracts.intent` 的最终形式 |

**✅ Phase 3 已完成（2026-05-13，决策：全内置）**：技术侧三件套与产品侧完全对称落地——

- **`arch-debate`**（skill.yaml + guide + tech-roles + 3 模板）：product-debate 的技术侧
  对称体。技术角色 ARCH（主持）/ SEC（代言攻击者）/ PERF / OPS / DATA / DEV，复用
  setup→debating→concluding 四 Phase + 强制暂停机制。消费 PRD § Intent Mapping 标
  `contracts` / [ADR 候选] 的行，产 rfc-draft + tech-decision-matrix + 辩论纪要。
- **`rfc-writer`**（skill.yaml + guide + 3 模板）：prd-writer 的对称体。产 RFC（含
  ARCHITECTURE.md 增量 File Manifest）+ contracts.intent 种子（`[Awaiting ADR]`，
  goal 块，实跑 `intent check` 0 VC exit 0）+ ADR 骨架（Proposed）。质量门 ≥ 90。
- **`adr-writer`**（skill.yaml + guide + 2 模板）：技术决策审批闸。固化 ADR
  Proposed→Accepted（不可变），解锁 contracts 种子的 `[Awaiting ADR]`，列收紧工单交
  intent-spec-writer。刻意保持薄（不发明内容），对称 intent-spec-writer 的「固化+解锁」定位。
- **职责不重叠的关键设计**：arch-debate 辩论 → rfc-writer 起草骨架 → adr-writer 固化+解锁
  → intent-spec-writer 收紧 → intent-consistency-check 验证。固化（不可变审批）与起草
  （可反复改）分离，且 adr-writer 兼做 contracts 解锁触发器。
- **pipeline 扩成 10 stage DAG**：在 prd 与 intent-spec 间插入 arch-debate→rfc→adr
  技术侧支线（无 contracts 候选时整段可 skip，popsicle 视 skipped 为依赖已满足）。
  已校验：字段合法、依赖指向存在的 stage/skill、拓扑无环、7 个审批点。
- **D2 一致贯彻**：性能/时延/容量不进 contracts goal（写 RFC § Quality Attributes，
  压测守护）；契约逻辑前后置待 ADR Accepted 后才收紧进 acceptance/invariants。
- bootstrap.md 流程图的两个外部虚线框（架构/RFC/ADR + intent spec writer）已改为内置实心；
  README skill 表 6→10、依赖顺序重排、pipeline 链路与使用步骤更新。

---

## 5. 支线：popsicle 简化（RFC D2）

参考 `popsicle/docs/rfc-workflow-only-core.md`。

- **立即做**：RFC 步骤 1 —— 在 `popsicle-core` 加 `ContentProvider` trait + 默认文件实现（纯加法、零破坏）。
- **延后做**：步骤 2–5（搬 doc 能力到 `popsicle-doc-helpers` crate）等 Phase 0–2 积累出真实痛点再动；步骤 6（真切割）留半年宽限。
- **Phase 0–2 期间**：维护一份 popsicle 痛点清单（做 tool / promote / doc 落点时遇到的耦合点），作为 D2 落地的依据。

---

## 6. 明确不做（范围边界）

- ❌ **代码生成 / 测试生成**：按 `bootstrap.md` 明确在 module 边界外。
- ❌ **扩展 intent-lang 支持时序/时间逻辑**：见 D1/D2，那是 TLA+ 的领域。
- ❌ **现在就砍 popsicle**：见 D4，痛点驱动、延后。

---

## 7. 风险与未决

| 风险 / 未决 | 缓解 |
|---|---|
| intent-lang 是 Alpha，表达力可能撑不起真实 invariant（如 billing `prorate-symmetric`「增减对称」）| Phase 0 用 saas-demo 真实约束做压力测试；表达不了的暴露出来 = intent-lang 的待补清单 |
| `intent check` 依赖系统 PATH 上有 `z3` | tool/skill 启动时检查 z3 可用性，缺失给明确报错 |
| 路线甲会让 acceptance.intent 大幅瘦身，部分「可验证性」转移到测试侧而测试侧暂无 skill | 接受现状；测试生成在边界外，靠 task「可观察的成功标志」+ 人工/CI 守 |
| `intent-spec-writer` 内置化增加 intent-coder 重量 | 保持薄：只做「种子语法收紧 + 冲突检查」，不做语义发明 |
| popsicle 简化与闭环并行可能分散精力 | D4 已锁定：支线只做非破坏的步骤 1，主线优先 |

---

## 8. 一句话状态

> Phase 0-3 全部闭环：intent-lang 封装为 `intent-validate` tool；**10 个自带 skill**
> （project-init / fact-extractor / product-debate / prd-writer / **arch-debate / rfc-writer /
> adr-writer** / intent-spec-writer / intent-consistency-check / living-doc-author）产品侧 +
> 技术侧全内置，并由 `migration-bootstrap` 10-stage pipeline 串成带依赖 + 7 审批点的 DAG
> （一键 `popsicle pipeline run`，技术侧支线无契约时可整段 skip）；saas-demo 9 份 `.intent`
> 真实语法全绿（17 verified / 4 skipped / 0 failed），注入矛盾跑出 FAIL + Z3 反例。整条
> 「需求 → PRD 任务图 → 架构辩论 → RFC/ADR → intent 收紧 → 机器验证 → CI 闸门 → 活文档保活」
> 链全程走通。**剩余仅边界外项**：代码/单测生成（明确不做）、popsicle 简化支线（D2，痛点驱动）、
> intent-lang 待补能力（struct-forall theorem / 聚合，记在 §7 风险）。
