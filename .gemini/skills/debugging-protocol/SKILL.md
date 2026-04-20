---
name: debugging-protocol
description: >-
  Structured debugging: hypothesis generation, validation, root cause analysis.
  For complex bugs, flaky tests, intermittent failures.
---

# Debugging Protocol

Rigorous framework: beyond ad-hoc troubleshooting to structured hypothesis → validation.

## Workflow

### 1. Initialize
Copy template `assets/debugging-session-template.md` → `docs/debugging/{issue}-{YYYY-MM-DD}-{HHmm}.md`

### 2. Define Problem
- **Symptom:** observable behavior vs expected
- **Scope:** which components involved

### 3. Hypotheses
List distinct, testable hypotheses. Avoid vague guesses. Differentiate layers (frontend vs backend). Example: "race condition in UI state" vs "DB schema misconfiguration."

### 4. Validation Tasks
Per hypothesis:
- **Objective:** prove/disprove what
- **Steps:** precise, reproducible
- **Code pattern:** exact command/query/script
- **Success criteria:** what output confirms hypothesis

### 5. Execute & Document
Per task: status (✅ VALIDATED / ❌ FAILED / ⚠️ INCONCLUSIVE), findings (evidence), conclusion.

### 6. Root Cause
Synthesize: primary root cause, confidence level, proposed fixes.

## Best Practices
- Be specific ("grep 'Error 500' in `/var/log/nginx/access.log`" not "check logs")
- Isolate variables: one change at a time
- Validate assumptions first (config, versions)
- Preserve evidence (trace IDs, timestamps, repro scripts)

## Language Modules

Load from `languages/` for language-specific tools, hypothesis categories, validation strategies:

| Module | Runtime |
|---|---|
| [Rust](languages/rust.md) | cargo, rustc, tokio |
| [Frontend](languages/frontend.md) | Vue 3, React, browser, Vite |

## Compliance
- Error Handling @.gemini/rules/error-handling-principles.md
- Logging — load `logging-and-observability-principles` skill
- Testing Strategy @.gemini/rules/testing-strategy.md (regression test for fix)
