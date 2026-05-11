# project-init — Writing Guide

> Audience: the AI agent (you) running the project-init skill. Read this **and the source discussion at `docs/source-discussions/idd-doc-migration.md` (if present)** before starting.

## Mission

Lay down the **doc-architecture stage** so that downstream skills (fact-extractor, prd-writer, rfc-writer, intent-spec writers) all write into well-defined slots. Get this wrong and every downstream skill silently degrades.

## The One Decision That Matters Most: Product Naming

Everything else can be fixed by re-running a skill. **Product naming cannot** — it bakes into the directory structure, into git history, and into every cross-reference downstream documents make. If you only get one decision right, get this one.

### Hard rules

1. **Products are customer-recognizable**, not internal modules. Customers don't say "I use the `core` crate"; they say "I use the database product."
2. **3 to 7 products**. Fewer than 3 → likely under-decomposed. More than 7 → either you're listing modules instead of products, or the project is genuinely sprawling and you should ask the human to confirm.
3. **No catch-all "common" / "shared" / "utils" products**. If something doesn't belong to any product, it's a candidate for `docs/invariants/` (cross-cutting) or `crates/common/` (code-level shared lib, not a product).
4. **Products map to bounded contexts**, which `fact-extractor` already extracts. Use that table; don't re-derive.

### Soft signals for "is this a product?"

- Does it have its own customer-facing surface (CLI, API, UI)?
- Could you imagine a sales conversation about it standalone?
- Does it have a roadmap distinct from its peers?

If yes to any → it's a product. If no to all → it's a module belonging to some other product.

## The Other Decision That Matters: First Migration Slice

Per the source discussion §六 (具体建议):
- ROI by domain varies wildly (database/network high, simulation medium, viz low)
- Pick a product with **high ROI + medium criticality + few reverse deps**
- Document **at least one alternative** considered, with reasons rejected

The slice you pick will become the **playbook** other products copy. So:
- Don't pick the most critical (too risky for a first run)
- Don't pick the smallest (won't generate enough patterns)
- Don't pick the most isolated (other slices won't be analogous)

## What "Skeleton" Means

**Every** file you create in the scaffolding state contains either:
- The template's default text (placeholders like `{product-name}`)
- `[TBD: needs archaeology]` where a real value would normally go

You **must not** invent content. The temptation is real — you have a fact-extraction-report; you could draft a passable PRODUCT.md from it. **Don't.** That's `prd-writer`'s job. Stay in your lane: structure, not content.

The one exception: `docs/CHARTER.md` is content, but it's content the user has already locked in (the Four Iron Laws). You promote it verbatim from the planning artifact.

## Why the Plan Is Reviewed Before Scaffolding

Two reasons:
1. **Reversibility cliff** — once `git submodule add` runs and `popsicle init` runs, you've made changes that are annoying (not impossible, but annoying) to undo. Catch mistakes in the plan, not in `git rm -rf`.
2. **Product naming is hard** — humans need a chance to look at the proposed product list and say "actually, `database` and `storage` should be one product called `persistence`" before that name is baked into 50 file paths.

If the human approves and then changes their mind after scaffolding: re-run the skill. It's the cheapest skill in the toolkit and produces no consumed-by-others artifacts other than the doc-architecture, which is meant to be revisitable.

## Handling Multi-Repo Legacy Sources

The default assumes one legacy repo. If the user's legacy footprint spans multiple repos:
- Add multiple submodules under `legacy/<name1>/`, `legacy/<name2>/`, etc.
- Document each in the plan's "Legacy Source" section
- fact-extractor will need to be run per-submodule; that's fine

## Handling Greenfield (No Legacy)

Rare but valid. If the user is genuinely starting fresh:
- Skip the submodule step
- Skip the fact-extractor input
- Product inventory comes entirely from human input
- The skill still produces the same scaffold — the doc architecture is value-additive even without migration

## When Things Go Wrong

- **Submodule add fails** (network, auth) → record the failure in the project-init-plan, leave a `MIGRATION_TODO.md` at repo root with the exact command to run later, continue with the rest of the scaffold
- **`popsicle module add` fails for one module** → continue with the others, record the failure
- **The user disagrees with the product list mid-scaffold** → STOP. Roll back to surveying state. Do not partially scaffold.

## Output Format Convention

Both artifacts (`project-init-plan` and `doc-architecture-charter`) end with their respective checklists. The plan's checklist is what the workflow guard verifies before allowing the transition to scaffolding.
