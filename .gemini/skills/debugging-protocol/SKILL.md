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

## Active Debugging Techniques

### Binary Search for Regressions
When a regression has no obvious cause:
1. Identify last known good state (commit, deploy, config change)
2. `git bisect start` → `git bisect bad` (current) → `git bisect good` (known good)
3. At each step: run failing test or reproduce symptom
4. Result: exact commit that introduced the regression
5. Root cause analysis on that commit's diff

### Concurrency Debugging Checklist
For race conditions, deadlocks, and data races:
- [ ] Identify all shared mutable state
- [ ] Map lock acquisition order — look for cycles (deadlock)
- [ ] Check for missing synchronization on shared variables
- [ ] Verify atomic operations are used correctly
- [ ] Test with race detectors (`go test -race`, `-fsanitize=thread`, `TSAN`)
- [ ] Add stress tests: increase parallelism, add random delays
- [ ] Check for time-dependent assumptions (sleep-based synchronization)

### Memory Debugging Patterns
For leaks, corruption, and excessive allocation:
- [ ] Profile heap allocation (pprof, heaptrack, valgrind)
- [ ] Check for unclosed resources (connections, file handles, channels)
- [ ] Verify cleanup in error paths (defer/finally/RAII)
- [ ] Look for unbounded caches or growing maps
- [ ] Check for goroutine/thread leaks (blocked goroutines never cleaned up)

### Postmortem Integration
After resolving a complex bug:
1. Extract regression test — codify the fix as a test that would have caught it
2. Update monitoring — add alert/log pattern to detect symptom earlier
3. **Handoff** — for formal postmortem (P0-P2 incidents), hand off debugging session document to `@incident-responder`

> Full postmortem template and blameless review process: `@.gemini/skills/incident-response/SKILL.md`

## Language Modules

Load from `languages/` for language-specific tools, hypothesis categories, validation strategies:

| Module | Runtime |
|---|---|
| [Rust](languages/rust.md) | cargo, rustc, tokio |
| [Frontend](languages/frontend.md) | Vue 3, React, browser, Vite |

## Compliance
- Error Handling GEMINI.md § Error Handling Principles
- Logging — load `logging-and-observability-principles` skill
- Testing Strategy GEMINI.md § Testing Strategy (regression test for fix)
- Incident Response @.gemini/skills/incident-response/SKILL.md (for postmortem handoff)

