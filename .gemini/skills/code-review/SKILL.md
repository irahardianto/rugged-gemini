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
Read from `.gemini/rules/`. Use `rule-priority.md` for severity.

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
| [SEC] | Security | security-principles.md |
| [DATA] | Data integrity | error-handling-principles.md |
| [RES] | Resource leak | resources-and-memory-management |
| [TEST] | Testability | architectural-pattern.md, testing-strategy.md |
| [OBS] | Observability | logging-and-observability-mandate.md |
| [ERR] | Error handling | error-handling-principles.md |
| [ARCH] | Architecture | architectural-pattern.md, project-structure.md |
| [PAT] | Pattern consistency | code-organization-principles.md |
| [INT] | Integration contract | api-design-principles |
| [DB] | Database design | database-design-principles |
| [CFG] | Configuration | configuration-management-principles |

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

## Related
- Rule Priority @.gemini/rules/rule-priority.md
- Security Principles @.gemini/rules/security-principles.md
- Architectural Patterns @.gemini/rules/architectural-pattern.md
- Testing Strategy @.gemini/rules/testing-strategy.md
- Logging Mandate @.gemini/rules/logging-and-observability-mandate.md
- Error Handling @.gemini/rules/error-handling-principles.md
