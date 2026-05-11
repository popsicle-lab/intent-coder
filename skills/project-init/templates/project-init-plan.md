# Project Init Plan — {repo_name}

> **Status**: draft → review → approved
> **Created**: {YYYY-MM-DD}
> **Author**: {agent + human-confirmer}

This plan must be reviewed and approved by a human **before** any file is created. Scaffolding is irreversible-ish; product naming bakes into every downstream document.

---

## Repository Identity

| Field | Value |
|---|---|
| Repo name | `{new-intent-lang}` |
| Local path | `{/Users/.../Workspace/github/new-intent-lang}` |
| Issue key prefix | `{INTENT}` (used in BUG-INTENT-1, TC-INTENT-1, …) |
| Default agent target | `{claude}` (one of: claude / cursor / copilot / codex / opencode) |
| License | `{MIT}` (must be compatible with legacy source license) |
| Initial branch | `main` |

---

## Product Inventory

> The most important table in this plan. Once approved, these names become directory paths in `products/<name>/` and are referenced by every downstream doc.

| # | Product | 1-line purpose | Est. LoC | Source (fact-ext §) | Status |
|---|---|---|---|---|---|
| 1 | `{syntax}` | IntentLang parser & AST | {1,200} | Bounded Contexts row 1 | scaffold-only |
| 2 | `{verifier}` | Z3-backed semantic verification | {2,400} | Bounded Contexts row 2 | **slice (first)** |
| 3 | `{cli}` | `intent` command-line entry point | {600} | Bounded Contexts row 3 | scaffold-only |
| 4 | `{ui}` | Tauri desktop visualizer | {1,800} | Bounded Contexts row 4 | scaffold-only |

**Validation against hard rules**:
- [ ] Each name is customer-recognizable (not `core` / `utils` / `common`)
- [ ] Count is between 3 and 7
- [ ] No catch-all "shared" product
- [ ] If fact-extraction-report present, every Bounded Context row maps to exactly one product (or is explicitly merged with reason)

---

## Legacy Source

| Field | Value |
|---|---|
| Repo URL | `{https://github.com/popsicle-lab/intent-lang.git}` |
| Submodule path | `legacy/{intent-lang}` |
| Pinned commit SHA | `{abcdef1234...}` |
| Pin justification | "main HEAD as of init day; will not auto-track" |
| License | `{MIT}` — compatible with new repo |
| Multi-repo? | no (single submodule) |

> If multi-repo: list each submodule on its own row.

---

## First Migration Slice

**Chosen**: `{verifier}`

**Rationale** (cite fact-extraction-report sections):
1. {High churn (47 commits in last 6 months) per Risk Hotspots row 2}
2. {Owns the Z3 binding — the highest-leverage formal-method surface}
3. {Has 3 reverse deps (cli, ui, syntax) — manageable, not isolated}

**Alternatives considered**:

- **`{syntax}`**: lower risk (read-only AST), but few invariants → low IDD ROI for a learning slice
- **`{cli}`**: lowest LoC, but it's an aggregator — its specs are mostly delegations, doesn't exercise the doc system
- **`{ui}`**: too few invariants per source-discussion §六 (visualization product ROI = low)

**What "first slice" means concretely**:
- Only `products/verifier/` gets non-skeleton content during the rollout
- Other products get the full directory tree but every file stays at `[TBD: needs archaeology]` until their turn
- The verifier's complete cycle (PRFC → PDR → PRODUCT.md → contracts.intent → first code in `crates/verifier/`) becomes the playbook

---

## Module Dependencies

| Module | Source | Purpose | Status |
|---|---|---|---|
| `intent-coder` | `/Users/narwal/Workspace/github/intent-coder` | This module — migration toolkit | required |
| `popsicle-spec-development` | `/Users/narwal/Workspace/github/popsicle-spec-development` | PRD/RFC/ADR writers, test specs | recommended |
| `popsicle-intent-development` | `/Users/narwal/Workspace/github/popsicle-intent-development` | intent-lang spec writers, verification skills | recommended |

All paths verified to exist before approval.

---

## Scaffolding Manifest

> Exhaustive list of every directory and file the next state will create. Nothing else may be created. Nothing here may be skipped.

### Repo-level

```
.gitattributes                              # append: *.intent linguist-language=Scala
.gitmodules                                 # auto-managed by `git submodule add`
CONTRIBUTING.md                             # IDD workflow rules (human + AI agent reference)
```

### docs/ (cross-product layer)

```
docs/
├── CHARTER.md                              # promoted from doc-architecture-charter artifact
├── invariants/
│   └── README.md                           # empty index, [TBD: needs archaeology]
├── baseline/
│   └── .gitkeep
├── glossary.md                             # skeleton with empty term table
└── PROJECT_CONTEXT.md                      # skeleton; populated later by living-doc-author
```

### products/ (per-product 4-piece × N)

For each product in the inventory:

```
products/<name>/
├── PRODUCT.md                              # business living doc, [TBD] placeholders
├── ARCHITECTURE.md                         # technical living doc, [TBD] placeholders
├── intents/
│   ├── invariants.intent                   # empty — header comment only
│   ├── contracts.intent                    # empty
│   └── acceptance.intent                   # empty
├── decisions/
│   ├── adr/.gitkeep
│   └── pdr/.gitkeep
└── proposals/
    ├── exploring/.gitkeep
    ├── proposed/.gitkeep
    ├── accepted/.gitkeep
    └── rejected/.gitkeep
```

### migration/ (one-time bootstrap aids)

```
migration/
├── slices/.gitkeep
├── traceability.md                         # legacy-path → new-path + responsible spec
└── progress.md                             # kanban: not-started / in-progress / done per product
```

### legacy/ (submodule — created by `git submodule add`)

```
legacy/<legacy-name>/                       # pinned to {commit-sha}
```

---

## Risks & Open Questions

| Risk / Question | Mitigation / Owner |
|---|---|
| Product naming may need revision after stakeholder review | Re-run project-init; cheap |
| Submodule URL may not be accessible from CI | Document in CONTRIBUTING.md; fall back to vendoring if needed |
| Some bounded contexts merge into one product (or split) | Resolve in this plan, not after scaffolding |

---

## Plan Checklist

- [ ] Repository Identity table fully populated (no placeholders)
- [ ] Product Inventory has 3-7 entries, each customer-recognizable
- [ ] Each product entry shows status (slice / scaffold-only) and 1-line purpose
- [ ] Exactly one product is marked **slice**
- [ ] Legacy Source section has URL + pinned SHA + license check
- [ ] First Migration Slice section names ≥1 alternative with rejection reason
- [ ] Module Dependencies — every path verified to exist
- [ ] Scaffolding Manifest is exhaustive (every file the scaffolding state will create)
- [ ] Risks & Open Questions — every known concern recorded
