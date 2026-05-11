# Unsafe & Risk Report — {project_name}

> Companion to `{slug}.fact-extraction-report.md`. Records every construct in the codebase that **bypasses the language's safety net** or **carries known risk**. Consumed by safety-spec and invariant-spec to know what invariants are most worth proving.

> **Scope**: this report records **what exists**, not **what should change**. Recommendations belong in adr-writer or rfc-writer artifacts downstream.

---

## Memory / Type-Safety Bypass

### `unsafe` blocks (Rust)

| File:Line | Context | Surrounding comment | Reason (if stated) |
|---|---|---|---|
| `src/auth/state.rs:22` | `static mut SESSIONS` | `// SAFETY: only accessed under MUTEX` | yes — mutex justification |
| `src/util/cast.rs:18` | `transmute::<u64, *const Header>` | (no comment) | (none — needs human input) |

### FFI / cgo / NAPI calls

| File:Line | Function | Library | Pre/Post conditions documented? |
|---|---|---|---|
| `src/z3/binding.rs:55` | `Z3_mk_solver` | libz3 | partial (return-value handling, no error path) |

### Raw pointer arithmetic

| File:Line | Pointer source | Operation |
|---|---|---|
| `src/serde_fast/parse.rs:312` | `slice.as_ptr().add(i)` | offset within bounds-checked length |

---

## Failure-Mode Hotspots

### `.unwrap()` / `.expect()` call sites

| File:Line | Expression | Reachable from public API? | Note |
|---|---|---|---|
| `src/payment/process.rs:115` | `gateway.charge(amount).unwrap()` | yes | Panics on network error |
| `src/parse/lexer.rs:88` | `iter.next().unwrap()` | no (internal) | Loop invariant guards |

> Group "reachable from public API" first; that's where panics translate to user-visible crashes.

### `panic!` / `unreachable!` / `todo!`

| File:Line | Macro | Message |
|---|---|---|
| `src/lifecycle.rs:142` | `panic!` | `"unhandled phase transition: {:?} → {:?}"` |
| `src/codegen/emit.rs:401` | `todo!` | `"emit array literal"` |

### Dynamic evaluation / shell

| File:Line | Construct | Input Source |
|---|---|---|
| `src/templates/render.rs:55` | `evaluate(user_template)` | user-uploaded template |
| `src/scripts/run.rs:18` | `Command::new("sh").arg("-c").arg(cmd)` | config file |

---

## Concurrency Risk

### Shared mutable state

| Construct | File:Line | Synchronization | Notes |
|---|---|---|---|
| `lazy_static! HASHES: Mutex<HashMap<…>>` | `src/auth/state.rs:14` | Mutex | Held across await? — needs check |
| `static ATOMIC_COUNTER: AtomicU64` | `src/metrics.rs:8` | atomic | OK |

### `await`-holding locks

| File:Line | Lock type | Awaited expression |
|---|---|---|
| `src/cache/tiered.rs:88` | `tokio::sync::Mutex` | `db.fetch(key).await` |

---

## Cryptographic / Secret Handling

| Construct | File:Line | Algorithm / Library | Notes |
|---|---|---|---|
| Password hashing | `src/auth/hash.rs:12` | `argon2id` (argon2 v0.5) | OK — modern KDF |
| Token signing | `src/auth/token.rs:34` | `hmac-sha256` | Key loaded from `SECRET_KEY` env var |
| TLS | `src/http/server.rs:8` | `rustls 0.22` | OK |

### Secret-in-source scan

| Match | File:Line | Likely False Positive? |
|---|---|---|
| `AKIA...` regex hit | `tests/fixtures/example.json` | yes — fixture file |
| `password = "..."` | `src/dev/seed.rs:22` | yes — dev seed only |

---

## Network / IO Risk

| Endpoint | File:Line | Validation | Timeout |
|---|---|---|---|
| Stripe `/v1/charges` | `src/payment/gateway.rs:42` | amount > 0 not checked | 30s |
| GitHub `/repos/.../releases` | `src/release/check.rs:18` | none | 10s |

---

## Risk Density by Module

> Aggregate counts; helps prioritize the first migration slice.

| Module | unsafe | unwrap | panic | dyn-eval | Total |
|---|---|---|---|---|---|
| `auth` | 2 | 5 | 1 | 0 | 8 |
| `payment` | 0 | 12 | 0 | 0 | 12 |
| `storage` | 0 | 3 | 0 | 0 | 3 |
| ... | ... | ... | ... | ... | ... |

---

## Extraction Checklist

- [ ] Every entry has a file:line citation
- [ ] Every entry quotes the surrounding comment (or says `(no comment)`)
- [ ] `unsafe` blocks: `Reason (if stated)` is filled with `yes — <one-line>`, `no`, or `(none — needs human input)`
- [ ] `.unwrap()` table separates "reachable from public API" from "internal-only"
- [ ] Concurrency section reviewed even if empty (state `(none found)` rather than omit)
- [ ] Cryptographic section reviewed even if empty
- [ ] Risk-density table totals match individual section counts
- [ ] No sentence contains "should be removed" / "should be replaced" (editorializing check)
