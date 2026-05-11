# intent-coder

Popsicle module that helps **migrate legacy codebases into the IDD (Intent-Driven Development) pipeline**: spec-driven, intent-verified, living-doc maintained.

This module lives at the front of the IDD adoption journey. It bridges "我有一份老代码" and "我准备好走 spec-and-verify pipeline 了" by first **extracting the facts** (what the code actually does today), then **gating downstream specs against verified intents**, then **keeping product/tech docs alive** as code evolves.

---

## Skills

| Skill | Status | Purpose |
|-------|--------|---------|
| **project-init** | ✅ shipped | Bootstrap a new IDD repo: pin legacy as a submodule, scaffold the per-product 4-piece doc structure (`PRODUCT.md` / `ARCHITECTURE.md` / `intents/` / `decisions/`+`proposals/`), promote the doc-architecture charter, install module deps |
| **fact-extractor** | ✅ shipped | Read a legacy codebase and emit a structured fact-base (dependency graph, public-API contracts, unsafe/risk inventory, tech-debt list) — the input every PRD/RFC/intent writer downstream needs |
| **intent-consistency-check** | ⏳ planned | Thin popsicle wrapper around `intent --format json check`; turns the Z3 verdict into a stage gate |
| **living-doc-author** | ⏳ planned | Re-runs PRD/RFC sections whose source artifacts (code, intents, ADRs) have changed; keeps the "current state" doc honest |

The four are intentionally ordered by dependency:
1. `project-init` runs once at repo birth; produces the directory stage every other skill writes into.
2. `fact-extractor` runs on the pinned legacy submodule; produces the evidence base.
3. `intent-consistency-check` needs `.intent` files to exist (so it presupposes some spec writing).
4. `living-doc-author` needs PRD/RFC documents to maintain (so it presupposes the first migration slice).

## Pipelines

| Pipeline | Purpose |
|----------|---------|
| **migration-bootstrap** (planned) | `project-init → fact-extractor → product-debate → prd-writer → arch-debate → rfc-writer + adr-writer + intent-consistency-check`. Run once per legacy codebase. |

## Usage

In your new project:

```bash
mkdir new-project && cd new-project && git init
popsicle init
popsicle module add /path/to/intent-coder
popsicle module add /path/to/popsicle-spec-development     # for prd/rfc/adr writers
popsicle module add /path/to/popsicle-intent-development   # for intent/safety writers

# 1. Bootstrap the doc structure (interactive — names the products, picks the first slice)
popsicle skill start project-init

# 2. After approval, scan the pinned legacy submodule for facts
popsicle skill start fact-extractor --source legacy/<your-legacy-name>

# 3. Run the full migration pipeline (when shipped)
# popsicle pipeline run migration-bootstrap
```

## Why a separate module?

The 4 skills here are **migration-time tools**, not ongoing IDD tools. Splitting them off means:

- you can adopt IDD on a new project **without** dragging migration scaffolding into your day-to-day pipelines;
- legacy archeology (fact-extractor) is reusable on any codebase, regardless of whether you eventually use intent-lang;
- the boundary makes the IDD contract crisper: "after migration-bootstrap finishes, you owe one PRD/RFC/ADR set + at least one passing `.intent` file".

## License

MIT
