---
name: performance-optimization-principles
description: >-
  Performance principles: measure-first methodology, data structures, caching,
  batching, when to stop. For hands-on profiling, use perf-optimization skill.
user-invocable: false
---

## Performance Optimization Principles

### Measure First

1. Profile to find actual bottleneck (don't guess)
2. Find 20% of code consuming 80% of resources
3. Optimize that specific bottleneck
4. Measure again — verify with benchmarks
5. Repeat only if still not meeting goals

Don't optimize: "fast enough," rarely executed, no measurable problem.

### Data Structures

- Hash map: O(1) lookup, unordered
- Array: O(1) index, O(n) search, ordered
- Tree: O(log n) ops, sorted
- Set: O(1) membership, unique

Wrong choice = degradation: array for lookups (O(n) vs O(1)), list for sorted data (O(n log n) vs O(log n)).

### Avoid Premature Abstraction

Costs: runtime (indirection, virtual dispatch), cognitive (layers), maintenance (ripple). Start concrete, abstract when pattern emerges. No "future flexibility" without evidence.

### Techniques

- **Caching:** store expensive results, TTL, proper invalidation
- **Lazy loading:** compute/load on-demand
- **Batching:** N queries → 1 query, batch INSERTs, pipeline Redis
- **Async I/O:** don't block, concurrent I/O ops
- **Connection pooling:** see @.gemini/skills/resources-and-memory-management/SKILL.md

### Checklist
- [ ] Measured performance problem (not guessed)?
- [ ] Profiled for actual bottleneck?
- [ ] Appropriate data structures for access pattern?
- [ ] Expensive ops cached with invalidation?
- [ ] Batch ops instead of N+1?
- [ ] Non-blocking I/O where appropriate?
- [ ] Measured improvement after optimization?

### Related
- Resources @.gemini/skills/resources-and-memory-management/SKILL.md
- Concurrency Mandate GEMINI.md § Concurrency and Threading Mandate
- Concurrency Principles @.gemini/skills/concurrency-and-threading-principles/SKILL.md
