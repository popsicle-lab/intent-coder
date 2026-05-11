# API Contracts — {project_name}

> Companion to `{slug}.fact-extraction-report.md`. Records the **public surface** of the project: what callers can rely on. Grouped by bounded context.

> **Scope rule**: only items marked `pub` (Rust) / `export` (TS/Go) / module-level non-underscore (Python) belong here. Internal helpers go in `dependency-graph.md` if they cross module boundaries, otherwise are out of scope.

---

## Bounded Context: {auth}

> **Path**: `src/auth/`
> **Purpose** (from module doc-comment or README): {"Identity, sessions, and role-based access control."}

### Public Functions

| Signature | File:Line | Mutates | Notes |
|---|---|---|---|
| `pub fn login(user: &str, pass: &str) -> Result<Token, Error>` | `src/auth/login.rs:42` | global session table | Panics if hash store is poisoned |
| `pub fn logout(token: Token) -> Result<(), Error>` | `src/auth/login.rs:88` | global session table | — |
| `pub fn check(token: &Token, role: Role) -> bool` | `src/auth/check.rs:12` | (read-only) | — |

### Public Types

| Type | Kind | File:Line | Public Fields / Variants |
|---|---|---|---|
| `Token` | struct | `src/auth/types.rs:8` | `value: String`, `expires_at: SystemTime` |
| `Role` | enum | `src/auth/types.rs:34` | `Admin`, `Editor`, `Viewer`, `Guest` |
| `Error` | enum | `src/auth/types.rs:60` | `BadPassword`, `Locked`, `Expired`, `Internal(String)` |

### Public Traits

| Trait | File:Line | Implementors |
|---|---|---|
| `SessionStore` | `src/auth/store.rs:5` | `MemoryStore`, `RedisStore` |

### HTTP/gRPC Endpoints (if any)

| Method | Path | Handler | Auth Required |
|---|---|---|---|
| POST | `/api/v1/login` | `auth::http::login_handler` (`src/http/auth.rs:12`) | No |
| POST | `/api/v1/logout` | `auth::http::logout_handler` (`src/http/auth.rs:48`) | Yes (Token) |

### Behavioral Notes (only what's documented or test-encoded)

- **Lockout policy** (encoded in `tests/auth/lockout.rs`): 5 consecutive failures within 15 minutes → user locked for 30 minutes.
- **Token TTL** (encoded in `src/auth/login.rs:55`): 24 hours.

> Do NOT add behavioral notes that are inferred. Only quote what tests assert or what the code literally does.

---

## Bounded Context: {payment}

> **Path**: `src/payment/`
> **Purpose**: {"…"}

### Public Functions

| Signature | File:Line | Mutates | Notes |
|---|---|---|---|
| ... | ... | ... | ... |

(repeat sections per context)

---

## Cross-Cutting Public APIs

> APIs that don't belong to a single bounded context (e.g. logging, error types, common traits).

| Item | File:Line | Notes |
|---|---|---|
| `pub trait Telemetry` | `src/common/telemetry.rs:1` | Used by all contexts |

---

## Stability Markers

> Anything explicitly marked `#[deprecated]`, `@deprecated`, `// EXPERIMENTAL`, etc.

| Item | Marker | Reason | File:Line |
|---|---|---|---|
| `pub fn legacy_login_v1` | `#[deprecated(since = "0.4.0")]` | Use `login` | `src/auth/legacy.rs:8` |

---

## Extraction Checklist

- [ ] Every bounded context from `fact-extraction-report.md` has its own section
- [ ] Every signature includes its file:line
- [ ] Every "Mutates" cell is filled (use `(read-only)` where applicable; do not leave blank)
- [ ] HTTP/gRPC endpoints section either populated or explicitly says `(none — library only)`
- [ ] Behavioral notes section contains only test-asserted or literally-encoded behavior (no inference)
- [ ] Cross-cutting APIs section either populated or says `(none)`
- [ ] Stability markers section either populated or says `(no deprecated/experimental markers found)`
