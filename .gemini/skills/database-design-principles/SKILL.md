---
name: database-design-principles
description: >-
  Database design: schemas, migrations, queries, indexes, transactions.
  Normalization, query optimization, migration safety.
user-invocable: false
---

## Database Design Principles

### Schema

**Normalization:** start 3NF, denormalize only with measured perf need. One entity per table. No derived data unless explicit cache.

**Naming:** tables=plural snake_case (`users`, `task_assignments`). Columns=singular snake_case. FKs=`{table_singular}_id`. Indexes=`idx_{table}_{cols}`. Constraints=`{type}_{table}_{cols}`.

**Required columns:** `id` (UUID preferred), `created_at` (immutable), `updated_at` (auto-update).

### Migrations

Safety: never drop prod columns without deprecation. Never rename directly (add new → migrate → drop old). Always reversible (up + down). Test on prod data copy.

Strategy: 1. additive → 2. backfill → 3. update code → 4. drop old in future migration.

### Queries

**Performance:** parameterized only (no SQL concat). Index WHERE/JOIN/ORDER BY columns. No SELECT *. Watch N+1 (use JOINs/batch). Set timeouts.

**Transactions:** for multi-row/table ops. Keep short. Retry on deadlock. Never hold during user interaction or external calls.

### Checklist
- [ ] Naming conventions followed
- [ ] All tables have id, created_at, updated_at
- [ ] FKs have constraints + indexes
- [ ] Parameterized queries (no injection)
- [ ] Indexes for freq query patterns
- [ ] Migrations reversible
- [ ] Transactions short + focused
- [ ] N+1 avoided

### Related
- Security Principles GEMINI.md § Security Principles
- Performance @.gemini/skills/performance-optimization-principles/SKILL.md
- Error Handling GEMINI.md § Error Handling Principles
