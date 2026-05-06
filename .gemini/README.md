# Rugged Gemini — Internal Documentation

Originally adapted from [Awesome AGV](https://github.com/irahardianto/awesome-agv). Purpose-built for [Gemini CLI](https://github.com/google-gemini/gemini-cli).

## Installation

Copy this directory's contents into your project's `.gemini/` directory:

```bash
# From your project root
cp -r path/to/rugged-gemini/* .gemini/
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
- **`settings.json`** — MCP servers (Pathfinder, Playwright). Update commands to match your local installations.

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

## Architecture Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Rules location | `GEMINI.md` (single file) | Auto-loaded every session; zero setup overhead |
| Commands format | `.toml` | Gemini CLI native format |
| Agent tool access | `tools:` allowlist | Explicit grants over implicit denials |
| Lang-specific rules | `skills/` (on-demand) | Reduces context noise; loaded only when relevant |
| MCP config | `settings.json` | Gemini CLI standard location |

## MCP Server Notes

Update `settings.json` MCP server commands to match your actual installations:

```json
"pathfinder": {
  "command": "pathfinder",
  "args": ["--workspace", "."]
}
```

If using npm-installed MCP servers, use the `npx` command pattern shown in `settings.json`.
