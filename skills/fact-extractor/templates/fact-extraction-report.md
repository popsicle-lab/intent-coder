# Fact Extraction Report — {project_name}

> **Baseline date**: {YYYY-MM-DD}
> **Source commit**: `{git rev-parse HEAD}`
> **Extracted by**: fact-extractor v0.1.0

This report is the **entry point** to the fact-base for `{project_name}`. It carries the executive summary and links to four detail artifacts. Every claim here is sourced from one of the four; do not introduce facts that aren't traceable to a detail artifact.

---

## Summary

| Metric | Value | Source |
|---|---|---|
| Total LoC | {n} | tokei (appendix A.1) |
| Primary language(s) | {Rust 78%, TS 22%} | tokei |
| Public crates / packages | {n} | dependency-graph.md |
| External direct dependencies | {n} | dependency-graph.md |
| Public API surface (functions + types) | {n} | api-contracts.md |
| `unsafe` blocks | {n} | unsafe-risk-report.md |
| `.unwrap()` / `.expect()` call sites | {n} | unsafe-risk-report.md |
| TODO / FIXME / HACK count | {n} | tech-debt-inventory.md |
| Top-50 commit-hotspot files | listed in §Risk Hotspots | git log |

---

## Bounded Contexts

> Top-level directories that look like cohesive bounded contexts. Each becomes a candidate "subject area" for downstream PRD/RFC writers.

| Context | Path | LoC | Primary types | Notes |
|---|---|---|---|---|
| {auth} | `src/auth/` | {1,243} | `User`, `Session`, `Role` | Owns identity & access |
| {payment} | `src/payment/` | {2,108} | `Charge`, `Refund`, `Receipt` | Calls external Stripe gateway |
| {storage} | `src/storage/` | {876} | `Repo`, `Tx` | Wraps Postgres |
| ... | ... | ... | ... | ... |

If a directory doesn't map cleanly to a context, list it under **§Unclassified** at the bottom of the table — don't force a label.

---

## Domain Glossary

> Terms that recur in code, comments, or doc strings. Used by downstream skills to maintain ubiquitous language.

| Term | First seen | Likely meaning | Confidence |
|---|---|---|---|
| {Tenant} | `src/multitenancy/lib.rs:1` | Customer organization that owns a set of users | high |
| {Pinpoint} | `docs/internal.md:42` | Geo-spatial event with attached metadata | medium |
| {Phase} | `src/lifecycle.rs:18` | One of 4 lifecycle states: pending/active/sealed/archived | high |

Confidence is `high` when the term is defined explicitly (in a struct doc-comment or README); `medium` when meaning is inferred from usage; `low` when usage is inconsistent.

---

## Risk Hotspots

> Files / modules that combine high churn AND high risk constructs. These are the prime candidates for the first migration slice.

| File | Commits/yr | unsafe count | unwrap count | TODO count | Primary risk |
|---|---|---|---|---|---|
| `src/payment/process.rs` | {47} | {0} | {12} | {3} | Panics on edge cases; no input validation |
| `src/auth/session.rs` | {38} | {2} | {5} | {1} | unsafe global state |
| ... | ... | ... | ... | ... | ... |

Each row must reference a specific subsection in `unsafe-risk-report.md` or `tech-debt-inventory.md` for the supporting evidence.

---

## Suggested First Migration Slice

> **Recommendation**, not a decision. Final scoping happens in arch-debate / rfc-writer.

Based on the hotspot table above, the lowest-risk-highest-leverage first slice is **{`src/payment/`}** because:

1. {High churn (47 commits/yr) → frequent re-touch → high benefit from invariants}
2. {No external API consumers visible in `api-contracts.md` → easier to refactor}
3. {Already has TODO comments hinting at desired invariants → spec-writer has a head start}

Alternative slices considered:

- **{src/auth/}**: also high-churn, but `unsafe` global state requires unsafe-removal as a prerequisite — larger scope.
- **{src/storage/}**: low churn, low risk; high effort, low payoff for first slice.

---

## Detail Artifacts

| Artifact | File | Status |
|---|---|---|
| Dependency graph | [{slug}.dependency-graph.md]({slug}.dependency-graph.md) | ✅ |
| API contracts | [{slug}.api-contracts.md]({slug}.api-contracts.md) | ✅ |
| Unsafe / risk report | [{slug}.unsafe-risk-report.md]({slug}.unsafe-risk-report.md) | ✅ |
| Tech-debt inventory | [{slug}.tech-debt-inventory.md]({slug}.tech-debt-inventory.md) | ✅ |

---

## Tool Provenance

| Tool | Version | Used For | Status |
|---|---|---|---|
| tokei | {12.1} | LoC counts, language breakdown | ✅ |
| cargo metadata | {1.0} | Rust dependency graph | ✅ |
| cargo tree | {1.79} | Transitive deps | ✅ |
| ripgrep | {14.1} | Pattern mining (unsafe, TODO, public API) | ✅ |
| ast-grep | {0.20} | Structural matching (optional) | ⚠️ not available |
| git log | — | Churn / hotspot data | ✅ |

If a tool was unavailable, the section that would have used it is flagged `[reduced fidelity]`.

---

## Extraction Checklist

- [ ] All five artifacts produced and cross-linked
- [ ] Every Summary-table value cites a detail artifact
- [ ] Bounded contexts listed (or `Unclassified` populated)
- [ ] Domain glossary contains ≥10 terms with confidence labels
- [ ] Risk hotspots table contains ≥5 entries with evidence pointers
- [ ] Suggested first migration slice has at least one alternative considered
- [ ] Tool provenance table lists every tool actually used
- [ ] No sentence in the report contains the phrases "should", "ought to", "is bad", "is good" (editorializing check)
- [ ] No sentence in the report invents requirements not present in the code
- [ ] Every approximate number replaced with the exact value or removed
