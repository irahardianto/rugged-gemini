---
name: guardrails
description: >-
  Pre-flight checklists before coding, post-implementation self-review after.
  Catches arch violations, missing observability, security oversights.
---

# Guardrails

## When to Invoke
- **Pre-Flight:** before writing any code
- **Self-Review:** after writing code, before verification

## Pre-Flight

- [ ] Identified applicable rules from `GEMINI.md`
- [ ] Searched for existing patterns (Pattern Discovery — GEMINI.md § Architectural Patterns)
- [ ] Project structure aligned (GEMINI.md § Project Structure)
- [ ] I/O boundaries identified for abstraction
- [ ] Test strategy determined (unit/integration/E2E)
- [ ] Reviewed GEMINI.md § Rule Priority for conflicts

Any item unchecked → STOP and resolve.

## Post-Implementation Self-Review

**Security:**
- [ ] No hardcoded secrets/config
- [ ] User input validated at boundaries
- [ ] Parameterized queries (no SQL concat)

**Testability:**
- [ ] I/O behind interfaces
- [ ] Business logic pure (no side effects in calcs)
- [ ] Dependencies injected

**Observability:**
- [ ] Public ops logged (start/success/failure)
- [ ] Structured logging with correlationId
- [ ] Appropriate log levels

**Error Handling:**
- [ ] Explicit error paths (no empty catch)
- [ ] Errors wrapped with context
- [ ] Resources cleaned up (defer/finally)

**Testing:**
- [ ] Happy path covered
- [ ] ≥2 error paths covered
- [ ] Domain edge cases
- [ ] I/O adapters modified → integration tests
- [ ] UI modified → E2E tests exist/planned

**Consistency:**
- [ ] Follows codebase patterns (>80%)
- [ ] Naming conventions match
- [ ] File org matches GEMINI.md § Project Structure

## Language-Specific

| Language | File |
|---|---|
| Go | `languages/go.md` |
| TypeScript | `languages/typescript.md` |
| Flutter/Dart | `languages/flutter.md` |
| Rust | `languages/rust.md` |

Load only for active languages. Missing file → flag for creation.

## Compliance
- All mandates (always-on rules)
- Architectural Patterns GEMINI.md § Architectural Patterns
- Testing Strategy GEMINI.md § Testing Strategy
- Rule Priority GEMINI.md § Rule Priority
