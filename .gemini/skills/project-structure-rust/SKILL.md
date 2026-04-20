---
paths:
  - "**/*.rs"
  - "**/Cargo.toml"
---

## Rust/Cargo Layout

Vertical slice — features are modules, not layers.

### Single Binary Crate

```
project-root/
  Cargo.toml
  build.rs                          # Optional (FFI, codegen)
  src/
    main.rs                        # Entry point: CLI, server, wire deps
    lib.rs                         # Public modules (enables integration testing)
    config.rs                      # Config loading
    error.rs                       # App-wide errors (thiserror)
    telemetry.rs                   # Tracing setup

    features/
      mod.rs
      task/
        mod.rs                     # Public API, re-exports
        service.rs                 # Business logic orchestration
        logic.rs                   # Pure rules (no I/O)
        models.rs                  # Domain structs
        error.rs                   # Feature errors
        repository.rs              # Storage trait
        postgres.rs                # PostgreSQL impl
        mock.rs                    # Mock impl

    handlers/
      mod.rs
      task_handler.rs              # HTTP handlers (axum/actix)
      auth_handler.rs
    router.rs                      # Routes + middleware

  tests/                           # Integration tests (separate crate, pub API only)
    common/mod.rs                  # Shared fixtures
    task_api_test.rs
  benches/
    task_benchmark.rs
```

Key: `lib.rs` + `main.rs` enables integration testing. `mod.rs` = feature boundary. `error.rs` per feature with `thiserror` + `#[from]`. No top-level `controllers/`/`services/`.

### Library Crate

```
project-root/
  Cargo.toml
  src/
    lib.rs                         # Public API surface
    parser.rs
    ast.rs
    error.rs
    internal/
      mod.rs
      optimizer.rs
      cache.rs
  examples/
    basic_usage.rs
  tests/
    parsing_test.rs
  benches/
    parser_benchmark.rs
```

### Cargo Workspace (Multi-Crate)

```
project-root/
  Cargo.toml                        # [workspace]
  Cargo.lock                        # Committed for binaries

  crates/
    myapp/
      Cargo.toml
      src/
        main.rs
        lib.rs
        config.rs
        error.rs
        api/
          mod.rs
          routes.rs
          handlers.rs

    myapp-parser/
      Cargo.toml
      build.rs
      src/
        lib.rs
        parser.rs
        transform.rs
        cache.rs
      tests/
        parser_integration.rs

    myapp-client/
      Cargo.toml
      src/
        lib.rs
        client.rs
        lifecycle.rs
        retry.rs

    myapp-search/
      Cargo.toml
      src/
        lib.rs
        search.rs
        filter.rs
      tests/
        search_integration.rs

    myapp-common/
      Cargo.toml
      src/
        lib.rs
        types.rs
        error.rs
      tests/
        common_integration.rs

  tests/
    integration/
      full_pipeline_test.rs

  config/
    myapp.config.default.json
```

Key differences: `crates/` = Cargo workspace. Each crate own `Cargo.toml` (scoped deps, separate compilation). `build.rs` for FFI. `Cargo.lock` committed for binaries. Feature flags via `[features]` in `Cargo.toml`.

### Test Organization (Rust-Specific)

**Unit — inline** (not separate files): `#[cfg(test)] mod tests` at bottom of each `.rs`. Conditionally compiled. Access private via `use super::*`. Idiomatic — do NOT create `*_test.rs`.

**Integration — `tests/` dir**: each file = separate crate, public API only. No `#[cfg(test)]` needed. Shared helpers: `tests/common/mod.rs` (NOT `tests/common.rs`).

**Workspace-level:** `tests/` at workspace root or dedicated test crate.

```
crates/pathfinder-search/
  Cargo.toml
  src/
    lib.rs                  # Contains #[cfg(test)] mod tests { ... }
    search.rs               # Contains #[cfg(test)] mod tests { ... }
    filter.rs               # Contains #[cfg(test)] mod tests { ... }
    mock.rs
  tests/
    search_integration.rs
    common/
      mod.rs
```

### Related
- Project Structure @.gemini/rules/project-structure.md
- Rust Idioms and Patterns @.gemini/rules/rust-idioms-and-patterns.md
