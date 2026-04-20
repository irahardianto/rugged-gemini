# geminicli-minimal

Ported from `claude-minimal`. Same engineering excellence, adapted for [Gemini CLI](https://github.com/google-gemini/gemini-cli).

## Installation

Copy this directory's contents into your project's `.gemini/` directory:

```bash
# From your project root
cp -r path/to/geminicli-minimal/* .gemini/
# Also copy GEMINI.md to project root (auto-loaded by Gemini CLI)
cp .gemini/GEMINI.md ./GEMINI.md
```

### Directory mapping

| Source (this repo) | Target (your project) |
|---|---|
| `settings.json` | `.gemini/settings.json` |
| `GEMINI.md` | `./GEMINI.md` (project root) |
| `agents/*.md` | `.gemini/agents/*.md` |
| `commands/*.toml` | `.gemini/commands/*.toml` |
| `skills/*/SKILL.md` | `.gemini/skills/*/SKILL.md` |

## What's Included

### Core System
- **`GEMINI.md`** — All engineering rules consolidated (security, rugged software, testability, SOLID, testing, logging, pathfinder routing). Auto-loaded every session.
- **`settings.json`** — MCP servers (Pathfinder, Playwright, Supabase, Cloud Run), auto-edit approval mode, checkpointing.

### 11 Specialized Agents
| Agent | Role | Access |
|---|---|---|
| `architect` | System design, ADRs, contracts | Read-only |
| `backend-engineer` | API, business logic, observability | Full |
| `frontend-engineer` | Vue 3 UI, Pinia, a11y | Full |
| `mobile-engineer` | Flutter, Riverpod, offline-first | Full |
| `database-expert` | Schema, migrations, queries | Full |
| `devops-engineer` | CI/CD, Docker, IaC, monitoring | Full |
| `test-automation-engineer` | E2E tests, test infra | Full |
| `technical-writer` | Docs, API docs, changelogs | Full |
| `qa-analyst` | Code review, debugging, quality gates | Read-only |
| `security-engineer` | Threat modeling, vulnerability audit | Read-only |
| `ux-reviewer` | Design heuristics, a11y, responsive | Read-only |

Invoke with `@agent-name` in Gemini CLI.

### 2 Slash Commands
- `/orchestrate` — Full pipeline manager (SCOUT → DESIGN → BUILD → TEST → REVIEW → VERIFY)
- `/refactor` — Safe incremental refactoring workflow

### 38 Skills
Language idioms (Go, TS, Vue, Flutter, Rust, Python), project structures, debugging protocols, code review checklists, performance optimization, CI/CD, accessibility, and more.

## Key Differences from Claude-Code Version

| Aspect | claude-minimal | geminicli-minimal |
|---|---|---|
| Rules location | `rules/*.md` (28 separate files) | `GEMINI.md` (single consolidated file) |
| Commands format | `.md` (markdown) | `.toml` (Gemini CLI native) |
| Agent frontmatter | Claude YAML schema | Gemini `kind: local` + `tools:` allowlist |
| Tool isolation | `disallowedTools:` (blocklist) | `tools:` (allowlist) |
| Lang-specific rules | `rules/` directory | `skills/` directory (on-demand loading) |
| MCP config | `settings.local.json` | `settings.json` |
| Context loading | Implicit (Claude auto-loads rules/) | Explicit (`GEMINI.md` at project root) |

## MCP Server Notes

Update `settings.json` MCP server commands to match your actual installations:

```json
"pathfinder": {
  "command": "pathfinder",
  "args": ["--workspace", "."]
}
```

If using npm-installed MCP servers, use the `npx` command pattern shown in `settings.json`.
