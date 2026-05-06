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

### Monorepo Strategies
When working in monorepos with multiple packages/services:
- **Scope commits to package**: `feat(api): add task endpoint` — scope = package name
- **Changelogs per package**: each package has its own CHANGELOG.md
- **Selective CI**: trigger builds only for affected packages (path filters)
- **Shared deps**: pin at workspace root, override per-package only when needed
- **Atomic changes**: cross-package changes in single commit when they must deploy together

### Semantic Release Automation
For automated versioning and changelog generation:
1. **Conventional commits → version bumps**: `feat` → minor, `fix` → patch, `BREAKING CHANGE` → major
2. **Tooling**: `semantic-release` (Node.js), `release-please` (GitHub), `go-semantic-release` (Go)
3. **CI integration**: run after merge to main, generates tag + changelog + release
4. **Pre-release channels**: `alpha`, `beta`, `rc` for pre-release testing

### Worktree Best Practices
When orchestrator uses git worktrees for parallel agent execution:
- **Naming**: `.wt/<agent-name>` — consistent prefix for cleanup
- **Branch naming**: `wt/<agent-name>-<timestamp>` — unique per dispatch
- **Cleanup**: always remove worktree + branch after merge
- **Conflict resolution**: merge sequentially, resolve conflicts before next merge
- **Never commit directly to main from worktree** — always squash merge

### Checklist
- [ ] Branch named with type prefix
- [ ] Conventional commit format
- [ ] No debug code or secrets
- [ ] Tests pass before commit
- [ ] PR <400 lines (or justified)
- [ ] Messages explain WHY
- [ ] Lock files committed with dependency changes
- [ ] CHANGELOG updated for user-facing changes

### Related
- Code Completion Mandate GEMINI.md § Code Completion Mandate
- Testing Strategy GEMINI.md § Testing Strategy
- Security Mandate GEMINI.md § Security Mandate
- Dependency Management Principles @.gemini/skills/dependency-management-principles/SKILL.md

