# Contributing to saas-demo

> 读者：**人类贡献者**和 **AI agent**。本文件由 CI 强制读取——任何修改 `docs/` /
> `products/*` 的 PR 都会跑 charter 合规检查，不通过自动驳回。

## 黄金法则

读 [`docs/CHARTER.md`](docs/CHARTER.md) 的「文档体系四条铁律」。它们**永不妥协**。

1. **活文档没有版本号**——只有 `Last-Updated` 和 `Last-Decision-Ref`
2. **决策档案只追加**——PDR / ADR 一旦 Accepted 永不修改；纠正请写新决策标
   `Supersedes`
3. **每次活文档编辑必须引用 Decision-Ref**——错别字 / 链接 / 措辞修复除外
4. **一次变更可能波及多份活文档**——触发它的 PDR/ADR 的 Consequences 必须列出
   所有强制更新的活文档段落；PR 必须在同一次提交里全部更新

---

## 我想加一个用户可见的行为，从哪开始？

按 v0.3 任务图范式，**不要**直接改 `products/<product>/PRODUCT.md`。流程：

```
1. 跑 product-debate                    # 多角色辩论，识别 task
   ↓
2. 跑 prd-writer                        # 产出五件套（含 PDR）
   ↓
3. PDR 评审通过 → Status: Accepted     # 决策档案落地
   ↓
4. 合并 task 文件到 products/<p>/tasks/{journey}/
   合并 acceptance 种子到 products/<p>/intents/acceptance.intent
   更新 PRODUCT.md 顶层 Catalog
   ↓
5. 跑 intent-consistency-check (Z3 闸)  # 上线后
   ↓
6. 开始写代码
```

**不允许**：
- ❌ 跳过 PDR 直接改 PRODUCT.md
- ❌ 在一份大 PRD 里塞「Functional Requirements / Core Features」（旧范式，已废）
- ❌ 引入第 6 个旅程阶段目录（5 个固定：onboarding / daily-ops / troubleshooting /
  admin / lifecycle）
- ❌ Task 标题写功能名（「重置密码功能」）—— 必须写用户原话（「我忘记密码后 5
  分钟内重新登录」）

---

## Task 文件的硬约束

详见 intent-coder 主仓库的 [`skills/prd-writer/references/task-organization.md`](../../skills/prd-writer/references/task-organization.md)。
要点：

- **文件名**：`T-{nnnn}-{kebab-slug}.md`（4 位 0 填充 task_id，永不变）
- **目录**：5 个固定旅程阶段之一
- **h1 标题**：完整人话句子，第一人称（「我……」）
- **frontmatter**：8 个必填字段（task_id / slug / title / journey_stage /
  audience / task_type / decision_ref / last_updated）
- **「本 task 可解答」**：≥ 3 个用户原话问句（query 锚点）
- **完成路径**：3-7 个 happy-path 步骤，分支拆到 `troubleshooting/`
- **长度**：≤ 250 行（硬上限）/ ≤ 150 行（软上限）
- **末尾**：`Decision-Ref: PDR-XXXX`

---

## PR 模板要求

每个修改活文档 / 决策档案的 PR 必须在 description 包含：

```markdown
## Decision-Ref

PDR-XXXX 或 ADR-XXXX 或 CADR-XXXX

## Charter Compliance Checklist

- [ ] 所有活文档修改 cite Decision-Ref
- [ ] 无历史/未来叙事短语（曾经/originally/previously/将会/计划于）
- [ ] PDR Consequences 列出的所有文件，本 PR 都已更新
- [ ] Task 文件 ≤ 250 行
- [ ] Intent Mapping 表与 acceptance.intent block 一一对应
```

---

## AI agent 贡献者额外纪律

如果你是 AI agent（Claude / Cursor / Copilot / Codex / OpenCode），还要：

1. **读 intent-coder 的 skills/<skill>/guide.md 全文**，再开始任何任务
2. **不跨 product 拼凑 PRD**——一次 prd-writer 锁定一个 target_product
3. **不替 PM 做 task 识别**（除非用户显式授权）——task 识别是 PM 在 product-debate
   Phase 4 的强制职责
4. **不修改已 Accepted 的 PDR/ADR**——发现错误请新写决策并标 `Supersedes`
5. 起草 task 文件时**先回答 3 个问题**（用户原话标题 / 5 选 1 旅程阶段 / 大小是
   否超 250 行），通不过就退到 product-debate 重新做 task 识别

---

## License

MIT
