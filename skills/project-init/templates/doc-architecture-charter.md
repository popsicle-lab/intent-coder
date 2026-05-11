# Doc Architecture Charter — {repo_name}

> **Promoted to**: `docs/CHARTER.md` after scaffolding
> **Status**: foundational — changing this charter requires its own ADR

This charter defines the non-negotiable rules of how documentation is organized, written, and changed in this repository. Every contributor — human and AI agent — reads this before touching `docs/`, `products/*/PRODUCT.md`, `products/*/ARCHITECTURE.md`, or any decision file.

---

## The Four Iron Laws of the Doc System

1. **Living docs have no version number** — only `Last-Updated` and `Last-Decision-Ref`. They always represent NOW. Outdated content is fixed in place; no historical narrative.
2. **The decision archive is append-only** — ADR/PDR files are never modified after `Status: Accepted`. Wrong decisions are corrected by writing a NEW decision that Supersedes them.
3. **Every living-doc edit must reference a decision ID** (except typo / link / wording fixes). CI enforces this on PRs touching `PRODUCT.md` or `ARCHITECTURE.md`.
4. **One change can touch many living docs** — the triggering ADR/PDR's `Consequences` section MUST list every living-doc section it forces an update to; the PR must update them all in one go.

---

## Layer Map

The doc system has 7 layers, distinguished by **what they constrain** and **how often they change**.

| Layer | Document | Constraint Object | Change Frequency | Owner | intent-lang |
|---|---|---|---|---|---|
| L0 | `docs/CHARTER.md`, `docs/charter.md` | The product's reason to exist; absolute floors | yearly | founders / arch committee | natural language only |
| L1 | `docs/invariants/*.intent`, `products/*/intents/invariants.intent` | Domain rules of nature | quarterly | PM + architect | ✅ core |
| L2 | `products/*/PRODUCT.md` + `acceptance.intent` | User-visible behavior | monthly | PM | ✅ acceptance fragment |
| L3 | `products/*/decisions/{adr,pdr}/*.md` | Why a specific choice was made | frozen at decision time, never modified | architect / PM | ❌ |
| L4 | `products/*/ARCHITECTURE.md` + `contracts.intent` | How a module is implemented | proposal-mutable, frozen on landing | tech lead | ✅ contracts fragment |
| L5 | `migration/slices/*.md`, change PRs | One specific change | one-shot | developer | ✅ as diff |
| L6 | `crates/`, `src/`, `tests/` | Machine behavior | continuous | AI + human | — |

> Misuse to avoid: a "PRD" that mixes L1 invariants, L2 product spec, and L4 contracts in one document is the #1 cause of large-project doc rot. Each layer changes at a different rate; mixing them forces all of them to the highest rate.

---

## Per-Product 4-Piece Set

Every product (under `products/<name>/`) has exactly four kinds of artifact. Each has a defined audience, a defined update method, and **a defined list of what NOT to write here**.

| Piece | View | Audience | Update Method | What NOT to write here |
|---|---|---|---|---|
| `PRODUCT.md` | business | PM, sales, customer success, AI | direct edit (small) or PDR-triggered (large) | implementation details, technical selection rationale |
| `ARCHITECTURE.md` | implementation | engineers, AI | direct edit (small) or ADR-triggered (large) | business strategy, pricing, customer segmentation |
| `intents/*.intent` | formal | LLM, Z3, CI | follows PRODUCT.md / ARCHITECTURE.md changes | natural-language narrative, rationale (those go in PDR/ADR) |
| `decisions/{adr,pdr}/*` | history | anyone tracing a decision | append-only | "current state" descriptions (those go in living docs) |

---

## Three-Layer Intent Hierarchy

| Scope | Path | Owner | Triggered By |
|---|---|---|---|
| **Cross-product (global)** | `docs/invariants/*.intent` | charter-level decisions | "constitutional" PDR (rare) |
| **Single product** | `products/<name>/intents/invariants.intent` | product team | PDR within that product |
| **Single feature / acceptance** | `products/<name>/intents/acceptance.intent` | product team | PDR within that product |
| **Single contract / module API** | `products/<name>/intents/contracts.intent` | architect | ADR within that product |

> Rule: every PDR/ADR's `Intent Impact` section must name which layer of intent it modifies. CI rejects decisions that don't.

---

## Proposal & Decision Lifecycles

### Technical side (RFC → ADR)

```
┌─────────┐    accept    ┌──────────┐
│Proposed │ ───────────► │ Accepted │ (immutable)
└─────────┘              └────┬─────┘
     │                        │ supersede
     │ reject                 ▼
     ▼                   ┌─────────────┐
┌──────────┐             │ Superseded  │
│ Rejected │             └─────────────┘
└──────────┘
```

### Product side (PRFC → PDR)

```
┌──────────┐  ready   ┌──────────┐  accept  ┌──────────┐
│Exploring │ ───────► │Proposed  │ ───────► │ Accepted │
└──────────┘          └──────────┘          └────┬─────┘
     │ deadline             │                    │ supersede
     │ exceeded             │ reject             ▼
     ▼                      ▼              ┌─────────────┐
┌──────────┐          ┌──────────┐         │ Superseded  │
│ Rejected │          │ Rejected │         └─────────────┘
└──────────┘          └──────────┘
```

> **Why the product side has `Exploring` and the tech side does not**: technical RFCs are submitted when the author already knows the direction and is selecting between alternatives. Product proposals often start as "should we even do this?" and need a research period (user interviews, data analysis, A/B tests) before a real proposal can be written. `Exploring` carries an `Decision-Deadline:` field — at deadline it MUST advance to `Proposed` or `Rejected`. This prevents two failure modes: (a) embryonic ideas polluting the proposal queue if forced to start as `Proposed`, (b) loss of audit trail if exploratory work happens entirely off-record.

---

## Forbidden Phrases in Living Docs

If a living doc (`PRODUCT.md`, `ARCHITECTURE.md`) contains any of these phrases, the PR fails review:

- "We originally used X..."
- "Previously, ..."
- "曾经..." / "之前..."
- "We migrated from X to Y because..."
- "Was: X. Now: Y."

These phrases describe **history**, which is the job of ADR / PDR / `git log`. Living docs use **present tense only**.

✅ `"Replication: Multi-Paxos [ADR-0015]"`
❌ `"We initially used Raft, then changed to Multi-Paxos in 2026 to reduce cross-region latency"`

---

## Anti-Patterns (and how to detect them)

| Anti-pattern | Detection | Mitigation |
|---|---|---|
| **Living docs as wiki** — anyone adds a section, structure rots | template-conformance check on PRs touching PRODUCT.md / ARCHITECTURE.md | every living-doc has a fixed top-level outline; new sections require a charter amendment |
| **ADR/PDR as diary** — daily standup notes filed as decisions | reviewer asks "would this still matter in 1 year?" before merge | `decisions/` folder PRs need 2 reviewers; trivia goes in issue/PR description |
| **Per-microservice splits** — one product becomes 20 separate `products/` dirs | repo-wide check: a product with > 5 sub-products is split | products are drawn at customer-recognizable boundaries, not code-module boundaries |
| **Roadmap as wishlist** — "Committed Roadmap" lists ideas without PDR | grep `PRODUCT.md`'s Roadmap section for entries lacking `[PDR-XXXX]` | roadmap entries without a Decision ID are invalid; ideas live in `proposals/exploring/` |
| **Doc–code drift** — living docs go stale | CI checks `Last-Updated` field; warns at N days; PRs touching `crates/<X>/` must confirm whether `products/<X>/ARCHITECTURE.md` needs an update | linter; explicit PR template question |

---

## Migration Sequence (for adopting this charter on a legacy project)

> Replicates the source-discussion §七 sequence; do not re-invent.

| Step | Action | When complete |
|---|---|---|
| 0 | Build empty 4-piece scaffold for each product | this skill (`project-init`) |
| 1 | Write current snapshot of each `PRODUCT.md` + `ARCHITECTURE.md` (with `[TBD: needs archaeology]` markers) | first slice's PM/tech-lead pair, ~2-3 weeks |
| 2 | Lazily backfill ADR/PDR — only when a `[TBD]` is hit or a new decision touches old territory; `Status: Reconstructed` | continuous |
| 3 | From a fixed cutover date: every living-doc PR requires a Decision ID | CI enforced |
| 4 | Add intent layer — each PDR/ADR gains an `Intent Impact` section | post-charter, parallel to step 3 |
| 5 | Acceptance/contracts intents grow per product; first slice produces the playbook | continuous |

---

## Charter Self-Reference

This charter is itself a living doc. Changes to it require a special class of decision: a **CADR** (Charter Amendment Decision Record) at `docs/decisions/cadr/`. CADRs are subject to the Four Iron Laws like any other decision file.
