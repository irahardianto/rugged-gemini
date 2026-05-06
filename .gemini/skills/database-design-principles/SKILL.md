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

**Required columns:** `id` (bigint IDENTITY for single-DB, UUIDv7 for distributed — avoid random UUIDv4), `created_at` (timestamptz, immutable), `updated_at` (timestamptz, auto-update).

**Data type selection:** `timestamptz` over `timestamp`. `text` over `varchar(n)` unless constraint needed. `numeric` over `float` for money. `bigint` over `int` for IDs.

### Migrations

Safety: never drop prod columns without deprecation. Never rename directly (add new → migrate → drop old). Always reversible (up + down). Test on prod data copy.

Strategy: 1. additive → 2. backfill → 3. update code → 4. drop old in future migration.

### Queries

**Design-level:** ensure indexes cover query access patterns. Validate query plans during schema review. For SQL coding patterns (parameterized queries, SELECT hygiene, N+1 avoidance, timeouts), see `@.gemini/skills/sql-idioms/SKILL.md`.

**Transactions:** for multi-row/table ops. Choose appropriate isolation level. Never hold during user interaction or external calls. For locking patterns (deadlock prevention, advisory locks, SKIP LOCKED), see `@.gemini/skills/sql-idioms/SKILL.md` § Concurrency & Locking.

### Partitioning

Tables >100M rows or time-series data → range partition by time. Enables instant partition drops vs slow DELETE.

### Security at Schema Level

**Row-Level Security:** for multi-tenant data, enable RLS + FORCE to enforce database-level tenant isolation. Never rely solely on application-layer filtering. Index columns used in RLS policies.

**Least privilege:** create separate roles (readonly, writer) with specific table grants. Revoke default public access. Never use superuser for application queries.

### Connection Management

Size pool as `(CPU cores × 2) + disk spindles`. Set `idle_in_transaction_session_timeout = 30s`. In transaction-mode pooling, avoid named prepared statements (use unnamed or deallocate).

### Checklist
- [ ] Naming conventions followed
- [ ] All tables have id, created_at, updated_at
- [ ] FKs have constraints + indexes
- [ ] Indexes for freq query patterns
- [ ] Migrations reversible
- [ ] timestamptz used (not timestamp)
- [ ] PK strategy appropriate (IDENTITY vs UUIDv7)
- [ ] Partitioning evaluated for tables >100M rows
- [ ] RLS enabled for multi-tenant tables
- [ ] Least privilege roles configured
- [ ] Connection limits + idle timeouts set

### Related
- Security Principles GEMINI.md § Security Principles
- Performance @.gemini/skills/performance-optimization-principles/SKILL.md
- Error Handling GEMINI.md § Error Handling Principles
