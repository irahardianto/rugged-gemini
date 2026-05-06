---
paths:
  - "**/*.rs"
---

## Rust Idioms and Patterns

Rust type system + ownership model = primary correctness tools. Lean into compiler — strongest ally. Idiomatic, safe, expressive.

> Scope: Rust coding idioms. Layout: `@.gemini/skills/project-structure-rust/SKILL.md`. Test naming: GEMINI.md § Testing Strategy. Logging: `@.gemini/skills/logging-and-observability-principles/SKILL.md`.

### Ownership and Borrowing

1. **Prefer borrowing (`&T`, `&mut T`) over cloning.** Never `.clone()` to silence borrow checker without `// CLONE:` comment. Use `Cow<'_, T>` when may/may not need ownership. Prefer `&str` over `String`, `&[T]` over `Vec<T>` in params.

2. **Minimize owned data in structs.** References + lifetimes for short-lived. Owned types when struct outlives inputs.

3. **Avoid unnecessary `Arc<Mutex<T>>`:** channels for one-directional, `RwLock` for read-heavy, `Arc<T>` (no lock) for immutable-after-init.

### Error Handling

1. **`?` for propagation — never `unwrap()` in production.** Acceptable only in tests, infallible ops with `// SAFETY:` comment, CLI `main()` with `expect("reason")`.

2. **Error crates by context:** library = `thiserror` (typed enums), application = `anyhow` (ergonomic chaining). Never mix: libs must not depend on `anyhow`.

3. **Error type design:**

```rust
// ✅ Typed, matchable
#[derive(Debug, thiserror::Error)]
pub enum PathfinderError {
    #[error("file not found: {path}")]
    FileNotFound { path: PathBuf },
    #[error("AST parse failed: {0}")]
    ParseError(String),
    #[error(transparent)]
    Io(#[from] std::io::Error),
}

// ❌ Stringly-typed, unmatchable
fn do_thing() -> Result<(), String> { ... }

// ✅ Force callers to handle
#[must_use]
pub fn create_task(req: CreateTaskRequest) -> Result<Task, TaskError> { ... }
```

### Async and Concurrency

1. **`tokio` runtime.** `#[tokio::main]`/`#[tokio::test]`. `tokio::spawn` over `std::thread::spawn`. `tokio::select!` for racing futures.

2. **Cancellation safety:** prefer `tokio::sync::mpsc` over broadcast. Document cancellation on async fns holding resources across `.await`. Use `CancellationToken` for shutdown.

3. **Blocking ops:** never blocking I/O in async. Use `tokio::task::spawn_blocking`. Use `tokio::fs` not `std::fs` in async.

### Unsafe Code

1. **Zero `unsafe` except FFI boundaries** (tree-sitter C bindings etc). Every `unsafe` needs `// SAFETY:` comment.

2. **Minimize surface:** encapsulate in safe wrapper. Public API safe from any context. Test boundary conditions.

3. **Never `unsafe` to bypass borrow checker** — restructure instead.

### Lifetimes and Generics

1. **Prefer `'_` elision.** Named lifetimes only when required/clarifying. `'a` for single, descriptive (`'input`, `'query`) for multiple.

2. **Simple generic bounds.** Concrete for prototyping, generics when pattern stabilizes. `impl Trait` for simple, `where` clauses for complex.

3. **Avoid lifetime gymnastics.** Complex annotations -> restructure to owned data or `Arc`. Consider "split borrow" pattern.

### Idiomatic Patterns

1. **Builder pattern** — `Self` return for chaining, `build()` returns `Result<T, BuildError>`.
2. **Newtype** — wrap primitives: `struct UserId(u64)`. `Deref` only for true "is-a".
3. **Typestate** — different states = different types. Invalid transitions = compile errors.
4. **`From`/`Into`** — implement `From<A> for B` (never `Into` directly). Use `thiserror`'s `#[from]`.

### Testing

1. **Organization (Rust-specific):**
   - **Unit:** `#[cfg(test)] mod tests` at bottom of each `.rs` file. Access private fns via `use super::*`. Stripped from production. Never create `*_test.rs` files.
   - **Integration:** `tests/` at crate root (separate crates). Public API only. Shared helpers: `tests/common/mod.rs` (NOT `tests/common.rs`).
   - `#[tokio::test]` for async.

2. **Naming:** `fn test_<function>_<scenario>_<expected>()` (snake_case).
3. **Assertions:** `assert_eq!`/`assert_ne!` over `assert!(a == b)`. `assert!(matches!(result, Ok(_)))` for variants.
4. **Property testing:** `proptest` or `quickcheck` for wide input spaces.

### Clippy and Formatting

1. **`cargo check`** for fast iteration. `cargo clippy` before commit. `cargo build` only for artifacts. Never `cargo build` during TDD.

2. **`cargo clippy` zero warnings.** `#[allow(clippy::...)]` only with `// ALLOW:` comment.

3. **`cargo fmt` non-negotiable.**

4. **Recommended config:**
```toml
[lints.clippy]
pedantic = "warn"
unwrap_used = "deny"
expect_used = "warn"
```

### Dependency Management
- Minimize count — each dep = attack surface + compile cost.
- Pin major versions: `dep = "1"` not `dep = "*"`.
- `cargo audit` regularly.
- Prefer well-maintained crates (downloads, last commit, issue tracker).

### Related
- Error Handling Principles GEMINI.md § Error Handling Principles
- Concurrency and Threading Principles @.gemini/skills/concurrency-and-threading-principles/SKILL.md
- Concurrency and Threading Mandate GEMINI.md § Concurrency and Threading Mandate
- Performance Optimization Principles @.gemini/skills/performance-optimization-principles/SKILL.md
- Resource and Memory Management Principles @.gemini/skills/resources-and-memory-management/SKILL.md
- Security Mandate GEMINI.md § Security Mandate
- Code Idioms and Conventions GEMINI.md § Code Idioms and Conventions
- Testing Strategy GEMINI.md § Testing Strategy
- Dependency Management Principles @.gemini/skills/dependency-management-principles/SKILL.md