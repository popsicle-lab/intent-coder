# Tech-Debt Inventory — {project_name}

> Companion to `{slug}.fact-extraction-report.md`. Records **explicit debt markers** in the codebase: TODO, FIXME, HACK, XXX, NOTE comments + deprecated API usage + dead-code candidates.

> **Scope rule**: only items the codebase **itself flags** as debt. Inferred debt (e.g. "this looks complicated") is NOT recorded — that's editorializing.

---

## Annotation Counts

| Marker | Count | Source |
|---|---|---|
| `TODO` | {n} | `rg -i 'TODO:'` |
| `FIXME` | {n} | `rg -i 'FIXME:'` |
| `HACK` | {n} | `rg -i 'HACK:'` |
| `XXX` | {n} | `rg -i 'XXX:'` |
| `NOTE` | {n} | `rg -i 'NOTE:'` |

---

## TODO / FIXME (with age)

> Sorted by age descending. Age = months since the comment was first introduced (`git log --diff-filter=A -S "TODO: …"`). Aged > 12 months = candidates for promotion to GitHub issues + ADR.

| File:Line | Marker | Comment | Author | First seen | Age |
|---|---|---|---|---|---|
| `src/payment/process.rs:108` | TODO | `validate amount > 0 before going to prod` | {alice} | {2023-04-12} | {24mo} |
| `src/auth/login.rs:55` | FIXME | `token TTL hard-coded — make configurable` | {bob} | {2024-01-08} | {16mo} |
| `src/cache/tiered.rs:88` | HACK | `holding mutex across await — fix when tokio fixes #5454` | {carol} | {2025-02-20} | {3mo} |

> If git blame is unavailable, omit Author / First seen / Age and flag the section `[reduced fidelity — git history unavailable]`.

---

## Deprecated API Usage

> Internal code calling APIs marked `#[deprecated]` (Rust), `@deprecated` (TS/Java), etc.

| Caller (file:line) | Deprecated API | Suggested replacement | Marked since |
|---|---|---|---|
| `src/auth/legacy.rs:42` | `crypto::md5_hash` | `crypto::sha256_hash` | 0.3.0 |

---

## Dead-Code Candidates

> Items the compiler / linter flags as unused. Only includes signals from the build (e.g. `cargo build` warnings, `rustc -W dead_code`, `unused-vars` from ESLint, etc.). Does NOT include speculative dead-code claims.

| Item | File:Line | Tool | Confidence |
|---|---|---|---|
| `pub fn old_login` | `src/auth/legacy.rs:8` | rustc dead_code | medium (still pub — could have external users) |

---

## Disabled Tests

> Tests marked `#[ignore]` (Rust), `it.skip` / `xit` (TS), `@unittest.skip` (Python), `t.Skip` (Go).

| Test | File:Line | Skip reason (from comment) | Skipped since |
|---|---|---|---|
| `test_payment_idempotency` | `tests/payment.rs:142` | "flaky — retried 5 times in CI, see #421" | {2024-09} |

---

## Configuration Smells

> Magic numbers / hard-coded URLs / hard-coded paths that probably shouldn't be hard-coded. Detected by simple grep heuristics; **flag with confidence level**.

| File:Line | Construct | Confidence |
|---|---|---|
| `src/payment/gateway.rs:8` | `https://api.stripe.com/v1` (hard-coded URL) | high — should be config |
| `src/cache/tiered.rs:14` | `LIMIT = 1024` (no comment) | low — could be intentional |

---

## Build Warnings (snapshot)

> Captured at extraction time. Full output in appendix B if relevant.

| Warning | Count | Worst offender |
|---|---|---|
| `unused_variables` | 12 | `src/codegen/emit.rs` |
| `dead_code` | 4 | `src/auth/legacy.rs` |
| `clippy::too_many_arguments` | 6 | `src/payment/process.rs` |

---

## Summary by Module

| Module | TODO | FIXME | HACK | Deprecated calls | Dead code |
|---|---|---|---|---|---|
| `auth` | 3 | 2 | 1 | 1 | 1 |
| `payment` | 4 | 1 | 0 | 0 | 0 |
| ... | ... | ... | ... | ... | ... |

---

## Extraction Checklist

- [ ] Every TODO/FIXME table row has file:line, marker, comment, age
- [ ] If git history was unavailable, age column shows `?` and section is flagged `[reduced fidelity]`
- [ ] Deprecated API section either populated or `(none found)`
- [ ] Dead-code section sourced ONLY from compiler/linter output, not speculation
- [ ] Disabled tests section reviewed even if empty
- [ ] Build warnings snapshot present (or marked `(build did not run)`)
- [ ] Module summary totals match individual section counts
- [ ] No sentence contains "should be done", "ought to fix", "is bad" (editorializing check)
