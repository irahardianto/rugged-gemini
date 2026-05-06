---
name: git-workflow
description: >-
  Git conventions: conventional commits, branch naming, PR hygiene,
  release tagging.
user-invocable: false
---

## Git Workflow

### Commits — Conventional Format

```
<type>(<scope>): <description>

[optional body]
[optional footer]
```

| Type | Purpose |
|---|---|
| feat | New feature |
| fix | Bug fix |
| docs | Documentation |
| style | Formatting |
| refactor | No feature/fix change |
| test | Adding/updating tests |
| chore | Maintenance, deps |
| perf | Performance |
| ci | CI/CD config |

Rules: imperative mood ("add" not "added"), scope = feature area, <72 chars, body explains WHY.

### Branch Naming

Format: `<type>/<ticket-or-desc>` (kebab-case)
```
feat/task-crud-api
fix/auth-token-expiry
refactor/storage-layer
chore/update-deps
```

### Commit Hygiene
- One logical change per commit
- Never commit broken tests
- No debug code (console.log, TODO hacks)
- No secrets (use .gitignore + env vars)

### PR Size
- Ideal: <400 lines. Acceptable: 400-800. Too large: >800 (split).

### Merge Strategy
- Feature → main: squash merge
- Release: merge commit
- Hotfix: cherry-pick

### Checklist
- [ ] Branch named with type prefix
- [ ] Conventional commit format
- [ ] No debug code or secrets
- [ ] Tests pass before commit
- [ ] PR <400 lines (or justified)
- [ ] Messages explain WHY

### Related
- Code Completion Mandate GEMINI.md § Code Completion Mandate
- Testing Strategy GEMINI.md § Testing Strategy
- Security Mandate GEMINI.md § Security Mandate
