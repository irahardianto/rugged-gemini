---
paths:
  - "**/*.go"
---

## Go Backend Layout

Vertical slice — features are packages, not layers.

```
apps/
  backend/
    cmd/
      api/
        main.go                     # Entry point: wire deps, router, start server
    internal/                       # Private (Go compiler enforced)
      platform/                     # Foundation (database, server, logger)
        database/
        server/
        logger/
      features/                     # Business vertical slices
        task/
          service.go                # Public API (Service struct)
          handler.go                # HTTP handlers
          handler_test.go           # Component tests (httptest + mock)
          logic.go                  # Pure business logic
          logic_test.go             # Unit tests (mock storage)
          models.go                 # Domain structs
          errors.go                 # Feature-specific errors
          storage.go                # Storage interface
          storage_pg.go             # PostgreSQL implementation
          postgres_integration_test.go  # Integration (Testcontainers)
          storage_mock.go           # Mock implementation
        order/
          handler.go
          logic.go
          storage.go
```

Key: `cmd/` = entry points (separate binaries). `internal/` = private packages. Tests co-located (`_test.go`). Single `go.mod` at root.

### Related
- Project Structure GEMINI.md § Project Structure
- Go Idioms and Patterns @.gemini/skills/go-idioms/SKILL.md
