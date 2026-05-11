# fact-extractor — Writing Guide

> Audience: the AI agent (you) producing the five fact-extractor artifacts. Read this before writing any artifact.

## Mission

Turn a legacy codebase into a **fact-base** that downstream IDD skills can cite. You are an archaeologist with a notebook and a tape measure, **not** a designer or a critic.

## The Three Anti-Patterns That Ruin a Fact-Base

These are the failure modes that make downstream PRD/RFC writers hallucinate. Avoid them religiously.

### 1. **Editorializing**

❌ "The auth module is poorly designed; it mixes concerns."
✅ "Module `auth` (src/auth/, 1,243 LoC) imports from `db`, `crypto`, and `http`. It exports 14 public functions; 7 of them mutate global state via `lazy_static` (src/auth/state.rs:22)."

The first sentence is an opinion masquerading as fact. The second is fact.

### 2. **Inferring intent**

❌ "Function `process_payment` should validate the amount before charging."
✅ "Function `process_payment(amount: u64) -> Result<Receipt, Error>` (src/payment/process.rs:108): no precondition checks on `amount`; first call is `gateway.charge(amount)` at line 115. TODO comment above says `// TODO: validate amount > 0 before going to prod`."

The first sentence invents a requirement. The second records what the code does *and* records the TODO that hints at the missing requirement — without claiming the TODO is correct.

### 3. **Approximating numbers**

❌ "Around 30% of the code is in the `core` crate."
✅ "tokei reports `core` crate at 4,127 LoC out of 13,508 total (30.5%); see appendix A.1."

Approximations strip downstream skills of the precision they need to scope work.

## What Each Artifact Is For (so you know what depth to write at)

| Artifact | Consumed By | Implication |
|---|---|---|
| `dependency-graph.md` | rfc-writer (designs new module boundaries) | Must be machine-readable — include adjacency tables, not just diagrams |
| `api-contracts.md` | prd-writer (writes "what does this product *do* today?") | Must group by bounded context, not by file |
| `unsafe-risk-report.md` | safety-spec, invariant-spec | Every entry needs file:line + the surrounding 1-line comment if any |
| `tech-debt-inventory.md` | adr-writer (records "why we have this debt"), bug-tracker | Each item needs estimated age (use git blame) |
| `fact-extraction-report.md` | All of the above; entry point | Cross-links to the four detail artifacts; carries the executive summary |

## Quoting Code

When quoting code, use fenced blocks tagged with the actual language and **always include the file:line above the block**:

````markdown
**src/auth/login.rs:42-50**
```rust
pub fn login(user: &str, pass: &str) -> Result<Token, Error> {
    let hash = HASHES.lock().unwrap().get(user).cloned();
    // ...
}
```
````

Quote at most 10 lines per block. If something needs more, link to the file instead.

## When You Don't Know Something

Write **`(unknown — needs human input)`** verbatim. Do **not** guess. Downstream skills will treat that string as a flag and ask the human. Examples where this is the right answer:

- The intended SLA of an API endpoint
- Whether a `panic!` is reachable in production
- The business reason for a specific magic number

## When a Tool Fails

Some tools won't be available in every environment (e.g. `cargo metadata` requires Rust toolchain). When a tool fails:

1. Note the failure in the report (`tool: cargo metadata — not available, dependency graph extracted via Cargo.toml inspection only`).
2. Fall back to a less precise method (e.g. parse Cargo.toml directly).
3. Flag the resulting artifact section as `[reduced fidelity]`.

Never silently substitute one tool for another — downstream consumers need to know what data they're working with.

## Iteration

`fact-extractor` is meant to run once per major codebase change. The artifacts it produces are dated baselines, not living documents (that's `living-doc-author`'s job). When in doubt about whether to include something, ask: "would a human reading this 3 months from now still find it useful as a baseline reference?" If yes, include it; if no, drop it.

## Output Format Convention

Every artifact must end with an `## Extraction Checklist` section. The top-level `fact-extraction-report.md` checklist is the one the workflow guard checks. The four detail artifacts have their own checklists for self-review.
