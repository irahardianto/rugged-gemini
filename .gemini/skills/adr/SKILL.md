---
name: adr
description: >-
  Document architectural decisions using ADR format. Use during research
  when choosing approaches, introducing deps/patterns, or changing arch.
---

# ADR Skill

Capture the **why**, not just the **what**. Institutional knowledge persists across conversations.

## When to Invoke
- Research phase: significant arch decision
- User requests decision documentation
- Choosing between 2+ viable approaches
- New dependency or pattern
- Changing existing architecture

## Storage
```
docs/decisions/
├── 0001-use-postgresql-for-storage.md
├── 0002-adopt-feature-based-structure.md
└── NNNN-short-title.md
```

## Template

```markdown
# NNNN. Short Title

**Date:** YYYY-MM-DD
**Status:** Proposed | Accepted | Deprecated | Superseded by NNNN

## Context
What issue motivates this decision? Constraints, requirements, context.

## Options Considered

### Option A: {name}
- **Pros:** ...
- **Cons:** ...
- **Effort:** Low/Medium/High

### Option B: {name}
- **Pros:** ...
- **Cons:** ...
- **Effort:** Low/Medium/High

## Decision
We chose **Option X** because...

## Consequences

### Positive
- What becomes easier/possible

### Negative
- What becomes harder
- Tech debt (if any)

### Risks
- What could go wrong
- Mitigation strategies

## Related
- Links to rules, ADRs, resources
```

## Guidelines
1. Number sequentially (check existing)
2. Short, descriptive titles
3. Status: Proposed → Accepted → Deprecated/Superseded
4. Never delete — mark `Superseded by NNNN`
5. Use `sequential-thinking` for complex trade-offs

## Compliance
- Architectural Patterns GEMINI.md § Architectural Patterns
- Core Design GEMINI.md § Core Design Principles
- Project Structure GEMINI.md § Project Structure
