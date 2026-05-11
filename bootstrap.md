# Bootstrap Guide — intent-coder Module

You are migrating a legacy codebase into the Intent-Driven Development workflow. This module provides the **archaeology and gating skills** that bridge "raw legacy code" and "spec-driven IDD pipeline".

The IDD adoption journey is:

```
legacy code
    │
    ▼
[fact-extractor]                ← intent-coder
    │  produces structured fact-base
    ▼
[product-debate → prd-writer]   ← popsicle-spec-development
    │  produces PRD grounded in extracted facts
    ▼
[arch-debate → rfc-writer + adr-writer]
    │  produces technical design + decision records
    ▼
[spec-writer → safety-spec → invariant-spec]   ← popsicle-intent-development
    │  produces .intent files
    ▼
[intent-consistency-check]      ← intent-coder
    │  Z3-gates the .intent files
    ▼
[implementation → unit-test-codegen]
    │  ...
    ▼
[living-doc-author]             ← intent-coder
    keeps PRD/RFC honest as code evolves
```

---

## Skill Mapping Guide

| Skill | Use When |
|---|---|
| **fact-extractor** | The very first step on any legacy codebase. It produces dependency-graph, public-API contracts, unsafe/risk inventory, and tech-debt list. **Every downstream skill (PRD writer, RFC writer, spec writer) consumes its output.** |
| **intent-consistency-check** | After any pipeline stage that produces or modifies `.intent` files. Acts as a Z3 gate — non-zero exit blocks downstream stages. |
| **living-doc-author** | After the first migration slice is complete and you have a PRD/RFC. Re-runs whenever the underlying code or intents change, keeping the "current state" sections honest. |

---

## When to Run fact-extractor

**Run it once at project bootstrap, then re-run when**:
- The codebase undergoes major restructuring (new modules, large deletions)
- A new domain is added (new bounded context)
- Before any large-scale refactor (so you have a "before" baseline)

**Don't re-run it on every PR** — it's a baseline tool, not a CI gate.

---

## Pipeline Selection Guide

When the user is starting a fresh migration of a legacy repo, use **`migration-bootstrap`**:

```
fact-extractor
  → product-debate
  → prd-writer
  → arch-debate
  → rfc-writer + adr-writer
  → intent-consistency-check (gate)
```

If the user already has PRD/RFC/intent files and only wants the gate, use the upstream **`spec-and-verify`** pipeline from `popsicle-intent-development` — `intent-consistency-check` will be auto-included once it's available as a stage skill.
