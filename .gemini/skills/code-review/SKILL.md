---
name: code-review
description: >-
  Structured code review protocol: inspect against full rule set.
  Use for audit workflows, code reviews, or when user requests review.
  Produces findings document with severity tags.
---

# Code Review Skill

Systematically review against full rule set. Catches what linters miss: arch violations, missing observability, logic errors, pattern inconsistencies.

## When to Invoke
- Audit workflows (Phase 1: Code Review)
- User-requested review
- Best: fresh conversation (avoid confirmation bias)

## Review Process

### 1. Scope
- Feature review — all files in feature dir
- PR review — changed files only
- Full audit — all features

### 2. Load Rules
Apply rules from `GEMINI.md`. Use Rule Priority (§ Rule Priority) for severity.

### 3. Categories (Priority Order)

**Critical (Must Fix):**
- [SEC] Security — injection, hardcoded secrets, broken auth
- [DATA] Data integrity — missing error handling on writes, no transactions
- [RES] Resource leaks — unclosed connections, missing cleanup

**Major (Should Fix):**
- [TEST] Testability — I/O not behind interfaces, untested error paths
- [OBS] Observability — missing logging, no correlation IDs
- [ERR] Error handling — empty catch, swallowed errors
- [ARCH] Architecture — circular deps, wrong layer access

**Minor (Nice to Fix):**
- [PAT] Pattern consistency — deviation from codebase patterns
- Naming — unclear names
- Organization — fns too long, mixed responsibilities

**Nit:** style (linter catches), missing comments on complex logic.

### 4. Findings Output

```markdown
# Code Review: {Feature/Module Name}
Date: {date}
Reviewer: AI Agent (fresh context)

## Summary
- **Files reviewed:** N
- **Issues found:** N (X critical, Y major, Z minor, W nit)

## Critical Issues
- [ ] **[SEC]** {description} — [{file}:{line}](file:///path)
- [ ] **[DATA]** {description} — [{file}:{line}](file:///path)

## Major Issues
- [ ] **[TEST]** {description} — [{file}:{line}](file:///path)
- [ ] **[OBS]** {description} — [{file}:{line}](file:///path)

## Minor Issues
- [ ] **[PAT]** {description} — [{file}:{line}](file:///path)

## Nit
- [ ] {description} — [{file}:{line}](file:///path)

## Rules Applied
List of rules referenced.
```

### 5. Save Report
Audit workflow: MUST save to `docs/audits/review-findings-{feature}-{YYYY-MM-DD}-{HHmm}.md`
Standalone: saving recommended but optional.

### 6. Severity Tags

| Tag | Category | Source |
|---|---|---|
| [SEC] | Security | GEMINI.md § Security Principles |
| [DATA] | Data integrity | GEMINI.md § Error Handling Principles |
| [RES] | Resource leak | @.gemini/skills/resources-and-memory-management/SKILL.md |
| [TEST] | Testability | GEMINI.md § Architectural Patterns, § Testing Strategy |
| [OBS] | Observability | GEMINI.md § Logging and Observability Mandate |
| [ERR] | Error handling | GEMINI.md § Error Handling Principles |
| [ARCH] | Architecture | GEMINI.md § Architectural Patterns, § Project Structure |
| [PAT] | Pattern consistency | GEMINI.md § Code Organization Principles |
| [INT] | Integration contract | @.gemini/skills/api-design-principles/SKILL.md |
| [DB] | Database design | @.gemini/skills/database-design-principles/SKILL.md |
| [CFG] | Configuration | @.gemini/skills/configuration-management-principles/SKILL.md |

### 7. Language Anti-Patterns

| Language | File |
|---|---|
| Go | `languages/go.md` |
| TypeScript | `languages/typescript.md` |
| Flutter/Dart | `languages/flutter.md` |
| Rust | `languages/rust.md` |

Anti-patterns = auto-fail. Pattern exists → finding.

### 8. Cross-Boundary Checks

State active dimensions at start: "Activating: A, B, C, D, E. Skipping F (no mobile)."

| Dim | When |
|---|---|
| A. Integration Contracts | Frontend + backend |
| B. Database & Schema | Uses DB |
| C. Config & Environment | Always |
| D. Dependency Health | Always |
| E. Test Coverage Gaps | Always |
| F. Mobile ↔ Backend | Mobile + backend |

**A — Integration:**
- [ ] Map every endpoint vs frontend adapter
- [ ] Field names, types, status codes match
- [ ] Centralized API client (no raw fetch)
- [ ] Auth coverage matrix
- [ ] Error contract alignment

**B — Database:**
- [ ] Required base columns (id, created_at, updated_at)
- [ ] FKs have indexes
- [ ] RLS policies on user data tables
- [ ] Model field names vs DB columns (drift)
- [ ] Migrations reversible + additive-first
- [ ] No N+1 queries

**C — Config:** no hardcoded secrets, `.env.template` coverage, fail-fast on missing config, secrets never logged.

**D — Deps:** no unused deps, no circular deps, public API imports only, audit for CVEs.

**E — Tests:** handler test per endpoint, integration test per adapter, error path coverage, E2E for primary journeys.

**F — Mobile:** API version compat, offline sync tested, token refresh flows.

### Zero-Findings Guard

<3 findings → MUST produce Dimensions Covered attestation:

```markdown
## Dimensions Covered
| Dimension | Status | Files Examined |
|---|---|---|
| A. Integration | ✅ / ⏭ Skipped (reason) | e.g., 26 routes vs 11 adapters |
| B. Database | ✅ / ⏭ Skipped | e.g., 8 tables + 4 adapters |
| C. Config | ✅ | scanned for secrets, .env.template |
| D. Deps | ✅ | npm audit, unused check |
| E. Tests | ✅ | handler tests for all endpoints |
| F. Mobile | ⏭ Skipped | no mobile app |
```

## Architecture Fitness Checks

During code review, assess these architecture health indicators:

### Dependency Direction
- [ ] Dependencies flow inward (domain ← application ← infrastructure)
- [ ] No circular dependencies between packages/modules
- [ ] Infrastructure concerns don't leak into domain layer
- [ ] Public API surface is minimal (internal packages/modules used)

### Coupling & Cohesion
| Indicator | Healthy | Unhealthy |
|---|---|---|
| Module dependency count | ≤ 5 direct deps | > 10 direct deps |
| Shotgun surgery frequency | Change touches 1-2 files | Change touches > 5 files |
| Feature leakage | Feature self-contained in directory | Feature scattered across tree |
| Interface width | ≤ 5 methods | > 10 methods (break apart) |

### Technical Debt Scoring

When reviewing, tag debt with impact/effort assessment:

| Severity | Impact | Effort | Action |
|---|---|---|---|
| **[DEBT-H]** | High risk, blocks features | > 1 day | File as priority backlog item |
| **[DEBT-M]** | Moderate risk, slows velocity | < 1 day | Fix in current sprint |
| **[DEBT-L]** | Low risk, cosmetic | < 1 hour | Fix opportunistically |

Include in findings output:
```markdown
## Technical Debt
- [ ] **[DEBT-H]** {description} — Impact: {what it blocks} — [{file}:{line}](file:///path)
- [ ] **[DEBT-M]** {description} — [{file}:{line}](file:///path)
```

### Refactoring Opportunity Flags

Flag code smells that indicate refactoring potential (for `@refactoring-specialist`):

| Flag | Trigger | Recommended Refactoring |
|---|---|---|
| **[REFACTOR-EXTRACT]** | Function > 30 lines or class > 300 lines | Extract method/class |
| **[REFACTOR-SIMPLIFY]** | Cyclomatic complexity > 10 | Simplify conditionals, extract strategy |
| **[REFACTOR-DEDUP]** | >3 instances of similar code | Extract shared function/module |
| **[REFACTOR-INTERFACE]** | Concrete dependency in constructor | Extract interface for testability |
| **[REFACTOR-PATTERN]** | Anti-pattern detected | Apply appropriate design pattern |

When refactoring opportunities are found, recommend: `/refactor <specific target>` for the user.

## Related
- Rule Priority GEMINI.md § Rule Priority
- Security Principles GEMINI.md § Security Principles
- Architectural Patterns GEMINI.md § Architectural Patterns
- Testing Strategy GEMINI.md § Testing Strategy
- Logging Mandate GEMINI.md § Logging and Observability Mandate
- Error Handling GEMINI.md § Error Handling Principles
- Refactoring Patterns @.gemini/skills/refactoring-patterns/SKILL.md (for refactoring execution)

