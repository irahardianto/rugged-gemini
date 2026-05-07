# Project Engineering Standards

> This file consolidates all rules from the minimal configuration. Gemini CLI auto-loads this as persistent context. All rules are always active — language-specific rules auto-apply based on file extensions in scope.

---

## Rule Priority (Conflict Resolution)

### Priority Order (highest to lowest)
1. **Security Mandate** — always wins
2. **Rugged Software Constitution** — foundational defensibility
3. **Code Completion Mandate** + **Logging Mandate** — validation and instrumentation non-negotiable
4. **Testability-First Design** — maintainability enables future
5. **Feature-specific** — language idioms, concurrency, CI/CD. Higher-priority rules win on conflict.
6. **PRD-gated** — feature-flags, gitops-kubernetes. Only when PRD explicitly requires. Confirm before activating.
7. **YAGNI / KISS** — only when no security/reliability/maintainability trade-off

### Common Conflicts

| Conflict | Resolution |
|---|---|
| YAGNI vs Security | Security wins. Input validation always needed. |
| KISS vs Testability | Testability wins. Interfaces enable testing. |
| Perf vs YAGNI | Measure first. Optimize only after profiling. |
| DRY vs Clarity | Clarity wins until 3+ duplications (Rule of Three). |
| Speed vs Logging | Logging wins. Silent failures = enemy. |
| YAGNI vs PRD-gated | PRD wins if explicitly required. |

When in doubt: *"Which choice = more defensible and maintainable?"* If equal -> simpler one (KISS).

---

## Security Mandate

Security = foundational requirement, not feature.

### Universal Principles
1. **Never trust user input** — validate all data from users/APIs/external sources server-side
2. **Deny by default** — explicit permission grants, never assume access
3. **Fail securely** — fail closed (deny) on errors, never open
4. **Defense in depth** — multiple layers, never single control

For implementation details (auth, validation, queries): see Security Principles section below.

---

## Rugged Software Constitution

### Core Philosophy
"I recognize that my code will be attacked." Generate defensibility, not just functionality.

### Commitments
1. **Responsible** — no happy-path-only code. Every input assumed malformed/malicious. Error handling = first-class feature.
2. **Defensible** — validate own state/inputs (Paranoid Programming). Fail securely (closed). Verify assumptions explicitly.
3. **Maintainable** — write for next year's reader, not today's compiler. Clarity over cleverness. Isolate complexity.

### 7 Rugged Habits
1. **Defense-in-depth** — validate at every boundary (API, DB, fn call). Never single-layer protection.
2. **Instrument for awareness** — code signals attacks/failures. Silent failures = enemy #1.
3. **Reduce attack surface** — remove unused code/deps/endpoints. Minimum public interface (Least Privilege).
4. **Design for failure** — assume DB down, network timeout, disk full. Circuit breakers, fallbacks.
5. **Clean up** — own acquired resources, ensure release. No TODO for security holes; fix or document risk.
6. **Verify defenses** — test unhappy paths as rigorously as happy.
7. **Adapt to ecosystem** — battle-tested libraries over custom. Community conventions for maintainability.

### Code Generation Rules
- **Refuse** insecure patterns (SQLi, hardcoded secrets, shell injection) even if asked.
- **Proactively** add validation, error handling, timeout logic even if not requested.
- **Explain** why defensive measures added.

---

## Code Completion Mandate

**Before marking any code task complete, run automated quality checks and remediate all issues.**

### Completion Workflow
1. **Generate** — write code
2. **Validate** — run language-appropriate quality checks
3. **Remediate** — fix all issues
4. **Verify** — re-run checks
5. **Deliver** — mark complete only after all checks pass

Never skip validation "to save time." Validation IS the work.

### Quality Commands per Language

| Language | Section |
|---|---|
| Go | `go vet ./...`, `staticcheck ./...`, `go test ./...` |
| TypeScript / Vue | `npx tsc --noEmit`, `npx eslint .`, `npm test` |
| Flutter / Dart | `dart analyze`, `flutter test` |
| Rust | `cargo clippy -- -D warnings`, `cargo test` |
| Python | `ruff check .`, `mypy .`, `pytest` |

### Failure Protocol
1. Read error output completely
2. Fix identified issues
3. Re-run failing command
4. Do not proceed until all checks pass

Never disable a lint rule or suppress a warning to pass. Fix root cause.

---

## Core Design Principles

### SOLID
- **SRP** — one reason to change per class/module/fn. If description needs "and" -> violates SRP.
- **OCP** — open for extension, closed for modification. Use composition + DI.
- **LSP** — subtypes substitutable for base types without breaking correctness.
- **ISP** — many small focused interfaces over one monolithic.
- **DIP** — depend on abstractions, not concretions. Core principle for testability-first.

### Essential Practices
- **DRY** — single authoritative representation. No duplicate logic/algorithms/business rules.
- **YAGNI** — no speculative features. Build for today, refactor when needs change.
- **KISS** — simple (easy to maintain) over clever. Complexity justified by actual requirements only.
- **Separation of Concerns** — distinct sections, minimal overlap, isolated modules/layers.
- **Composition over Inheritance** — delegation over class hierarchies. Interfaces/traits for polymorphism.
- **Least Astonishment** — follow established conventions. No surprising behavior.

---

## Architectural Patterns — Testability-First Design

### Core Principle
All code independently testable without running full application or external infra.

### Rule 1: I/O Isolation
Abstract ALL I/O behind interfaces/contracts: db queries, HTTP calls, file system, time/randomness, message queues.

### Rule 2: Pure Business Logic
Extract calculations, validations, transformations into pure fns: Input -> Output, no side effects, deterministic.

### Rule 3: Dependency Direction
Dependencies point inward toward business logic.

```
Infrastructure (DB, HTTP, Files, External APIs)
  ↓ depends on
Contracts/Interfaces (abstract ports, no implementation)
  ↓ depends on
Business Logic (pure fns, domain rules, NO infra deps)
```

### Pattern Discovery Protocol (MANDATORY before implementing ANY feature)
1. Search for: `Interface`, `Repository`, `Service`, `Store`, `Mock`
2. Examine 3 existing modules for consistency (db access, pure fns, testing patterns)
3. Document pattern (over 80% consistency required): "Following pattern from [module] modules"
4. If under 80% consistency: STOP and report fragmentation to human.

---

## Code Organization Principles

- Small focused fns (10-50 lines), single purpose
- Cognitive complexity under 10 for most fns
- Clear layer boundaries (presentation, business logic, data access)
- Design for testability from start, avoid tight coupling
- Naming conventions reveal intent without comments

### Module Boundaries
Feature-based organization with clear public interfaces:
- One feature = one directory
- Each module exposes public API (exported fns/classes)
- Internal implementation private
- Cross-module calls only through public API

---

## Error Handling Principles

1. **Never fail silently** — all errors handled explicitly (no empty catch). Catch = do something (log, return, transform, retry).
2. **Fail fast** — detect/report errors early. Validate at boundaries before processing.
3. **Provide context** — error codes, correlation IDs, actionable messages.
4. **Separate concerns** — different handlers for different types.
5. **Resource cleanup** — always clean up on error (close files, release connections, unlock).
6. **No information leakage** — sanitize for external consumption. No stack traces to users.

---

## Logging and Observability Mandate

### Every Operation Entry Point MUST Include Logging

**Operations (mandatory logging):**
API endpoints, background jobs, queue workers, event handlers, scheduled tasks, CLI commands, external service calls, database transactions.

**NOT operations (no direct logging):**
Pure business logic fns, utility/helper fns, data transformations/validators.

### Minimum 3 Log Points
1. **Start** — correlationId, userId, operation name
2. **Success** — duration, result identifiers
3. **Failure** — correlationId, error details, stack trace

### Mandatory Context
`correlationId` (UUID), `operation` (clear name), `duration` (ms), `userId` (when applicable), `error` (full context on failures).

---

## Concurrency and Threading Mandate

### When to Use
- **I/O-bound** — async I/O, event-driven, coroutines for network/file/db waits
- **CPU-bound** — OS threads or thread pools for heavy computation

### When NOT to Use
- Simple synchronous operations
- No measurable performance benefit

Concurrency adds significant complexity (races, deadlocks, debugging). Profile first — only add when measurable benefit exists.

---

## Testing Strategy

### Test Pyramid
- **Unit (70%)** — domain logic in isolation, mocked deps. Fast (under 100ms). Coverage over 85%.
- **Integration (20%)** — adapters against real infra (Testcontainers). Medium (100ms-5s).
- **E2E (10%)** — complete user journeys through all layers. Slow (5-30s).

### TDD: Red-Green-Refactor
1. Red: write failing test
2. Green: minimal code to pass
3. Refactor: clean up, tests stay green

### Test Organization

| Language | Unit | Integration |
|---|---|---|
| TS/JS | `*.spec.ts` | `*.integration.spec.ts` |
| Go | `*_test.go` | `*_integration_test.go` |
| Dart/Flutter | `*_test.dart` in `test/` | `*_integration_test.dart` |
| Python | `test_*.py` | `test_*_integration.py` |
| Rust | `#[cfg(test)] mod tests` inline | `tests/` at crate root |

---

## Security Principles

### OWASP Top 10 Enforcement
- **Broken Access Control** — deny by default. Validate permissions server-side every request.
- **Cryptographic Failures** — TLS 1.2+ everywhere. Encrypt PII/secrets at rest.
- **Injection** — ZERO TOLERANCE for string concatenation in queries. Parameterized queries only.
- **SSRF** — validate user-provided URLs against allowlist.

### Auth
- **Passwords** — Argon2id or Bcrypt (min cost 12). Never plain text.
- **Access Tokens** — short-lived (15-30 min), HS256 or RS256.
- **Refresh Tokens** — long-lived (7-30 days), rotate on use, `HttpOnly; Secure; SameSite=Strict`.
- **Rate Limiting** — strict on public endpoints. 5 attempts / 15 min.
- **RBAC** — permissions mapped to roles, not users. Check at route AND resource level.

### Input Validation
- "All input is evil until proven good."
- Validate against strict schema (Zod/Pydantic) at handler/port boundary.
- Allowlist good characters, never filter bad.

### Secrets
Never commit to git. Use `.env` (local) or Secret Managers (prod — Vault/GSM).

---

## Documentation Principles

### Self-Documenting Code
- Code shows WHAT, comments explain WHY.
- Comment when: complex business logic, non-obvious algorithms, bug workarounds, perf optimizations.

### Documentation Levels
1. **Inline** — explain WHY for complex code
2. **Function/method** — API contract (params, returns, errors)
3. **Module/package** — high-level purpose + usage
4. **README** — setup, usage, examples
5. **Architecture** — system design, component interactions

### AI-Friendly Documentation Lookup
When researching a library or framework:
1. **Check for `llms.txt`** at the library's domain first (e.g., `https://router.vuejs.org/llms.txt`, `https://pinia-colada.esm.dev/llms.txt`)
2. **Follow linked pages** in the `llms.txt` for up-to-date API references
3. **Never hallucinate or guess documentation URLs** — verify they resolve before citing
4. **Verify examples** against documentation before using — AI summaries of docs drift

---

## Code Idioms and Conventions

### Universal Principle
Write idiomatic code for target language. Follow community conventions, not personal preferences.

### Anti-Patterns
- No "Java in Python" or "C in Go"
- No forcing OOP in functional languages
- No avoiding features because "unfamiliar"

### Language-Specific Rules
Load relevant skill when working in that language:

**Opinionated defaults:**
- Go → `.gemini/skills/go-idioms/SKILL.md`
- TypeScript → `.gemini/skills/typescript-idioms/SKILL.md`
- Vue 3 → `.gemini/skills/vue-idioms/SKILL.md`
- Flutter/Dart → `.gemini/skills/flutter-idioms/SKILL.md`
- Rust → `.gemini/skills/rust-idioms/SKILL.md`
- Python → `.gemini/skills/python-idioms/SKILL.md`

**Community languages:**
- Java → `.gemini/skills/java-idioms/SKILL.md`
- C# → `.gemini/skills/csharp-idioms/SKILL.md`
- C++ → `.gemini/skills/cpp-idioms/SKILL.md`
- Swift → `.gemini/skills/swift-idioms/SKILL.md`
- Kotlin → `.gemini/skills/kotlin-idioms/SKILL.md`
- Elixir → `.gemini/skills/elixir-idioms/SKILL.md`
- JavaScript → `.gemini/skills/javascript-idioms/SKILL.md`
- PHP → `.gemini/skills/php-idioms/SKILL.md`
- Ruby → `.gemini/skills/ruby-idioms/SKILL.md`
- SQL → `.gemini/skills/sql-idioms/SKILL.md`

**Frameworks (load alongside language idioms):**
- React → `.gemini/skills/react-idioms/SKILL.md`
- Angular → `.gemini/skills/angular-idioms/SKILL.md`
- Next.js → `.gemini/skills/nextjs-idioms/SKILL.md`
- Django → `.gemini/skills/django-idioms/SKILL.md`
- Laravel → `.gemini/skills/laravel-idioms/SKILL.md`
- Rails → `.gemini/skills/rails-idioms/SKILL.md`
- Spring Boot → `.gemini/skills/spring-boot-idioms/SKILL.md`
- .NET → `.gemini/skills/dotnet-idioms/SKILL.md`

---

## Project Structure

**Philosophy:** Organize by FEATURE, not technical layer. Each feature = vertical slice.

**Universal Rule: Context -> Feature -> Layer**

1. **Level 1: Repository Scope** — root contains `apps/` grouping distinct applications.
2. **Level 2: Feature Organization** — vertical business slices. Anti-pattern: top-level technical layers.

### Adapting for Project Types

| Type | Layout |
|---|---|
| Monorepo (default) | `apps/backend/`, `apps/frontend/`, `apps/mobile/` |
| Single backend | Flatten: `cmd/`, `internal/` (Go) or `src/` (Rust) at root |
| Single frontend | Flatten: `src/` at root |
| Single mobile | Flatten: `lib/` at root |

### Project Structure Layouts
Language-specific project layouts:
- Go → `.gemini/skills/project-structure-go/SKILL.md`
- Vue → `.gemini/skills/project-structure-vue/SKILL.md`
- Python → `.gemini/skills/project-structure-python/SKILL.md`
- Rust → `.gemini/skills/project-structure-rust/SKILL.md`
- Flutter → `.gemini/skills/project-structure-flutter/SKILL.md`

---

## Orchestration Dispatch Protocol

> **Applies when:** Using the `/orchestrate` command or manually dispatching sub-agents.

### Agent Routing

Agents are organized in 4 layers: Research (read-only exploration), Design (read-only decisions/contracts), Builder (write in worktrees), Reviewer (read-only quality gates).

| Primitive | Agent Type | Layer | Rationale |
|-----------|-----------|-------|-----------|
| SCOUT | @scout (general) or domain agent (specialized research) | Research | Read-only codebase exploration, research |
| DESIGN | architect (lead) + optional ux-reviewer, database-expert, security-engineer, performance-engineer | Design | Multi-disciplinary design — architect leads, pulls domain experts from other layers |
| PRE-MORTEM | incident-responder + optional security-engineer, performance-engineer, database-expert | Reviewer | Proactive failure analysis on proposed designs |
| BUILD | Implementation agents | Builder | backend/frontend/mobile per MECE domains |
| TEST | test-automation-engineer | Builder | E2E, integration test infrastructure |
| REVIEW | qa-analyst + security-engineer + optional ux-reviewer, database-expert | Reviewer | Quality gates |
| REMEDIATE | Fix agents | Builder | Matches BUILD agent for the domain |
| OPTIMIZE | performance-engineer | Builder | Profiling, benchmarking, optimization |
| INCIDENT | incident-responder + domain engineers | Reviewer | Reactive: triage, RCA, mitigation, postmortem |
| REFACTOR | refactoring-specialist | Builder | Code smell detection, safe transformation |
| VERIFY | qa-analyst | Reviewer | Full test suite, lint, type check, build |
| DOCUMENT | technical-writer | Builder | Docs, API docs, changelogs |

### Parallel Dispatch

Agents support two parallelism modes:

- **Cross-domain**: Different agent types in parallel (e.g., `@backend-engineer` ∥ `@frontend-engineer`). Always safe — disjoint domains by definition.
- **Intra-domain**: Multiple scoped instances of the same agent type via `@agent-name[scope]` syntax (e.g., `@backend-engineer[auth]` ∥ `@backend-engineer[tasks]`). Requires MECE decomposition.

Intra-domain dispatch uses 4 composable skills from `.gemini/skills/`:
1. `parallel-dispatch-decomposition` — break work into MECE scope cards
2. `parallel-dispatch-dag` — build dependency graph, topological sort into levels
3. `parallel-dispatch-ownership` — validate exclusive write scopes, no file overlap
4. `parallel-dispatch-merge` — sequential merge with quality gates between branches

---

## Pathfinder Tool Routing

> **Applicability:** When Pathfinder MCP tools are available, follow these routing rules. If not available, use built-in tools as normal.

### Core Principle
Pathfinder operates at the **semantic level** (symbols, functions, classes). Built-in tools operate at **text level**. **Always prefer semantic tools for source code.**

### Tool Preference

| Action | Prefer (Pathfinder) | Instead of (Built-in) |
|---|---|---|
| Explore project structure | `get_repo_map` | directory listing |
| Search for code patterns | `search_codebase` | grep |
| Read a function or class | `read_symbol_scope` | read file |
| Edit a function body | `replace_body` | edit file |
| Edit entire declaration | `replace_full` | edit file |
| Batch-edit multiple symbols | `replace_batch` | multiple edits |
| Add code before/after symbol | `insert_before` / `insert_after` | edit file |
| Delete a function or class | `delete_symbol` | edit file |
| Pre-check a risky edit | `validate_only` | no equivalent |
| Create a new file | `create_file` | write file |
| Edit config files | `write_file` | edit file |

### Addressing Rules
Semantic paths MUST include file path and `::`. Example: `src/main.rs::MyClass.my_function`

### Graceful Fallback
If Pathfinder unavailable, fall back to built-in tools transparently. Do not block.
