---
paths:
  - "**/*.go"
---

## Go Idioms and Patterns

Go favors simplicity, explicitness, readability. Intentionally small language — resist importing patterns from other languages. Boring and obvious = idiomatic.

> Scope: Go-specific coding idioms. File layout: `@.gemini/skills/project-structure-go/SKILL.md`. Test naming: GEMINI.md § Testing Strategy. Logging: `@.gemini/skills/logging-and-observability-principles/SKILL.md`.

### Error Handling

1. **Always return errors — never panic in library/business code.** `panic` reserved for unrecoverable states. `recover` only at top-level goroutine boundaries.

2. **Wrap with `%w`:**
   ```go
   // ✅ Preserves the error chain for errors.Is / errors.As
   return fmt.Errorf("creating task for user %s: %w", userID, err)

   // ❌ Loses the error chain
   return fmt.Errorf("creating task: %v", err)
   ```

3. **Sentinel errors for expected branches:**
   ```go
   var ErrNotFound = errors.New("not found")
   var ErrUnauthorized = errors.New("unauthorized")

   if errors.Is(err, ErrNotFound) {
       // handle
   }
   ```

4. **Typed errors for rich domain errors:**
   ```go
   type ValidationError struct {
       Field   string
       Message string
   }
   func (e *ValidationError) Error() string {
       return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
   }

   var ve *ValidationError
   if errors.As(err, &ve) {
       // access ve.Field, ve.Message
   }
   ```

5. **Handle at right level** — propagate until enough context to act. Never swallow or re-wrap same error twice.

### Interfaces

1. **Small — one or two methods ideal:**
   ```go
   // ✅ Focused, composable
   type Reader interface { Read(p []byte) (n int, err error) }
   type Writer interface { Write(p []byte) (n int, err error) }

   // ❌ Monolithic
   type FileManager interface {
       Read(...); Write(...); Delete(...); List(...); Stat(...)
   }
   ```

2. **"Accept interfaces, return structs"** — params accept interfaces for flexibility, return concrete structs.

3. **Define where used, not where implemented:**
   ```go
   // ✅ Defined in consumer package (task feature)
   // task/storage.go
   type Storage interface {
       GetByID(ctx context.Context, id string) (*Task, error)
   }

   // postgres.go implements Storage — does NOT define it
   ```

4. **Implicit satisfaction** — no `implements` keyword needed. Any matching method set satisfies automatically.

### Goroutines and Channels

> General concurrency: `@.gemini/skills/concurrency-and-threading-principles/SKILL.md`. Go-specific mechanics below.

1. **Always pass `context.Context` first:**
   ```go
   // ✅
   func (s *Service) GetTask(ctx context.Context, id string) (*Task, error)

   // ❌ — no cancellation/deadline propagation
   func (s *Service) GetTask(id string) (*Task, error)
   ```

2. **Never start goroutine without knowing how it stops:**
   ```go
   go func() {
       for {
           select {
           case <-ctx.Done():
               return
           case item := <-ch:
               process(item)
           }
       }
   }()
   ```

3. **`errgroup` for concurrent fan-out:**
   ```go
   g, ctx := errgroup.WithContext(ctx)
   g.Go(func() error { return fetchUsers(ctx) })
   g.Go(func() error { return fetchOrders(ctx) })
   if err := g.Wait(); err != nil { ... }
   ```

4. **Channels for ownership transfer; mutexes for shared state.** Close channels from sender only.

### Naming

1. **Receivers:** short, first letter of type. `(s *Service)` not `(svc *Service)` or `(self *Service)`.
2. **Packages:** short, lowercase, no underscores, no plurals. `package task` not `package tasks`.
3. **Acronyms:** all caps or all lowercase. `userID` not `userId`. `HTTPClient` not `HttpClient`.
4. **Unexported:** omit type name (it's private, keep terse).
5. **No stutter:** `task.Task` fine; `task.TaskService` not.

### Idiomatic Patterns

1. **Functional options:**
   ```go
   type Option func(*Service)

   func WithTimeout(d time.Duration) Option {
       return func(s *Service) { s.timeout = d }
   }

   func NewService(store Storage, opts ...Option) *Service {
       s := &Service{store: store, timeout: 30 * time.Second}
       for _, o := range opts { o(s) }
       return s
   }
   ```

2. **`defer` — always error-checked closures.** Never bare `defer X.Close()`.

   ```go
   // ❌ NEVER: Error silently discarded
   defer rows.Close()

   // ✅ ALWAYS: Error-checked closure with structured logging
   rows, err := db.QueryContext(ctx, query)
   if err != nil { return fmt.Errorf("querying tasks: %w", err) }
   defer func() {
       if err := rows.Close(); err != nil {
           slog.Warn("failed to close rows", "error", err, "operation", "ListTasks")
       }
   }()
   ```

   Transaction rollback:
   ```go
   // ❌ NEVER
   defer tx.Rollback()

   // ✅ ALWAYS: Guard against sql.ErrTxDone
   defer func() {
       if err := tx.Rollback(); err != nil && !errors.Is(err, sql.ErrTxDone) {
           slog.Error("failed to rollback transaction", "error", err, "operation", "CreateOrder")
       }
   }()
   ```

   HTTP response body:
   ```go
   // ❌ NEVER
   defer resp.Body.Close()

   // ✅ ALWAYS: Drain then close
   defer func() {
       if _, err := io.Copy(io.Discard, resp.Body); err != nil {
           slog.Warn("failed to drain response body", "error", err)
       }
       if err := resp.Body.Close(); err != nil {
           slog.Warn("failed to close response body", "error", err)
       }
   }()
   ```

3. **Avoid `init()`** — implicit, hard to test. Prefer explicit init in `main` or constructors.
4. **Struct embedding** for code reuse only when true "is-a" relationship.
5. **Named return values** only for documentation or defer cleanup. Never naked returns in non-trivial fns.

### Testing

> Test naming, pyramid: GEMINI.md § Testing Strategy. Go-specific tooling below.

1. **Table-driven tests (default):**
   ```go
   func TestCalculateDiscount(t *testing.T) {
       tests := []struct {
           name     string
           input    float64
           expected float64
           wantErr  bool
       }{
           {"zero items", 0, 0, false},
           {"negative input", -1, 0, true},
       }
       for _, tt := range tests {
           t.Run(tt.name, func(t *testing.T) {
               got, err := calculateDiscount(tt.input)
               if tt.wantErr {
                   require.Error(t, err)
                   return
               }
               require.NoError(t, err)
               assert.Equal(t, tt.expected, got)
           })
       }
   }
   ```

2. **`testify`** — `require` (fatal) + `assert` (non-fatal).
3. **Race detector in CI** — `go test -race ./...`
4. **`httptest.NewRecorder()`** for handler tests. No live server.
5. **Test behavior, not implementation** — assert outputs/side effects, not internal fields.

### Formatting and Static Analysis

Must pass zero warnings/errors before commit. See GEMINI.md § Code Completion Mandate.

| Tool | Purpose | Command |
|---|---|---|
| `gofumpt` / `goimports` | Canonical formatting | `gofumpt -l -w .` |
| `go vet` | Correctness checks | `go vet ./...` |
| `staticcheck` | Advanced static analysis | `staticcheck ./...` |
| `gosec` | Security scanning | `gosec -quiet ./...` |
| `golangci-lint` | Aggregated linter (CI) | `golangci-lint run` |
| `govulncheck` | Dependency CVE scanning | `govulncheck ./...` |

- Never disable linter without comment explaining why.
- **`//nolint:errcheck` NEVER acceptable.** Handle errors — even in defer. Use error-checked closure. #1 audit finding source.
- Other `//nolint:` require rationale comment AND code review approval.
- Dev iteration: `go vet ./...` for type-checking; reserve `golangci-lint` for pre-commit.

> Never `fmt.Println`/`log.Printf` in production. Use `log/slog` (Go 1.21+) or project adapter. See `@.gemini/skills/logging-and-observability-principles/SKILL.md`.

### Related
- Code Idioms and Conventions GEMINI.md § Code Idioms and Conventions
- Project Structure — Go Backend @.gemini/skills/project-structure-go/SKILL.md
- Testing Strategy GEMINI.md § Testing Strategy
- Error Handling Principles GEMINI.md § Error Handling Principles
- Concurrency and Threading Principles @.gemini/skills/concurrency-and-threading-principles/SKILL.md
- Logging and Observability Principles @.gemini/skills/logging-and-observability-principles/SKILL.md
- Dependency Management Principles @.gemini/skills/dependency-management-principles/SKILL.md
