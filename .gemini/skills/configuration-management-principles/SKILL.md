---
name: configuration-management-principles
description: >-
  Config management patterns: env vars, config files, settings structs,
  secrets injection. Validation, layering, fail-fast, twelve-factor.
user-invocable: false
---

## Configuration Management Principles

### Config vs Code

Config: env-specific values (URLs, creds, timeouts), changes between envs, changes without deploy.
Code: business logic, same across envs, requires deploy.

Never hardcode: ❌ `const DB_URL = "postgresql://prod:5432"` → ✅ `const DB_URL = process.env.DATABASE_URL`

### Validation (at startup)

Fail fast if required config missing/invalid. Clear error: "DATABASE_URL required."

Checks: type, format (URL/email/path), range (port 1-65535), dependencies (feature X → config Y required).

### Hierarchy (highest → lowest)

1. CLI args — override everything (testing/debug)
2. Env vars — override config files
3. Config files — env-specific (config.prod.yaml)
4. Defaults — reasonable fallback

### Organization

**Hybrid:** config files for structure + .env for secrets.

**.env files:** secrets + env-specific values only. Never committed except `.env.template`.
- `.env.template` — blank values (COMMIT)
- `.env.development` — local creds (DO NOT COMMIT)

```
DEV_DB_HOST=123.45.67.89
DEV_DB_USERNAME=prod_user
DEV_DB_PASSWORD=a_very_secure_production_password
```

**Feature files:** grouped by function. Primary method for non-secret settings.

```
default: &default
  adapter: postgresql
  pool: 5
development:
  <<: *default
  host: localhost
  database: myapp_dev
  username: <%= ENV['DEV_DB_USERNAME'] %>
  password: <%= ENV['DEV_DB_PASSWORD'] %>
production:
  <<: *default
  host: <%= ENV['PROD_DB_HOST'] %>
  database: myapp_prod
  username: <%= ENV['PROD_DB_USERNAME'] %>
  password: <%= ENV['PROD_DB_PASSWORD'] %>
```

> Feature flags are a distinct, PRD-gated concern — see @.gemini/skills/feature-flags-principles/SKILL.md.

### Related
- Security Mandate @.gemini/rules/security-mandate.md
- Security Principles @.gemini/rules/security-principles.md
- Feature Flags @.gemini/skills/feature-flags-principles/SKILL.md
