---
name: sequential-thinking
description: >-
  Dynamic, reflective problem-solving through iterative thought chains with
  revision and branching. For complex planning, multi-step analysis, hypothesis
  verification, or problems where scope emerges during analysis.
---

# Sequential Thinking

Structured approach: breaks complex problems into iterative thought steps with revision and course correction.

## When to Use
- Complex problems → manageable steps
- Planning requiring iterative refinement
- Analysis needing mid-stream correction
- Scope emerges during analysis
- Multi-step solutions needing cross-step context
- Hypothesis generation + verification

## Methodology

1. **Initial estimation** — estimate thoughts needed, stay flexible
2. **Iterative analysis** — work sequentially, build context
3. **Revision** — question/revise previous thoughts as understanding deepens
4. **Branch exploration** — explore alternatives when needed
5. **Hypothesis cycle** — generate, verify against chain, repeat
6. **Convergence** — continue until satisfactory solution

## Thought Structure

Each thought includes:
- `thought`: current step content
- `thoughtNumber`: position (1, 2, 3...)
- `totalThoughts`: estimate (adjustable)
- `nextThoughtNeeded`: boolean

Optional: `isRevision`, `revisesThought`, `branchFromThought`, `branchId`, `needsMoreThoughts`.

## Process

**Starting:** estimate, begin thought 1 (context + approach), conservative totalThoughts.

**During:** build on previous, filter irrelevant info, express uncertainty, revise freely, adjust totalThoughts.

**Revision:**
```json
{
  "thought": "On reflection, thought 3's assumption about X was incorrect because Y...",
  "thoughtNumber": 6,
  "totalThoughts": 10,
  "isRevision": true,
  "revisesThought": 3,
  "nextThoughtNeeded": true
}
```

**Hypothesis cycle:** generate → verify against chain → fails? revise/branch → repeat until validated.

**Completion:** `nextThoughtNeeded: false` only when satisfied. Single clear answer addressing original problem.

## Context Management

- Reference previous thoughts by number
- Track valid vs revised thoughts
- Ignore irrelevant details per step
- Complex thought → break into multiple, increase totalThoughts

## Output Format

```
Thought [N/Total]: [content]
[If revision: "Revises thought X because..."]
[If branch: "Branching from thought X to explore..."]

Solution: [Clear, direct answer]
```

## Principles
- Flexibility > rigidity
- Revision = strength
- Hypothesis-driven
- Context-aware across thoughts
- Clarity at completion
