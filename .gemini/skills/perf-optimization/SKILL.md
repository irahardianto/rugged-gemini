---
name: perf-optimization
description: >-
  Profile-driven performance optimization: methodology, pattern catalog,
  safety invariants, when-to-stop heuristics. Language-specific in languages/*.md.
---

# Performance Optimization Skill

## When to Use
- Profiling data available (pprof, flamegraph, py-spy, Chrome/Dart DevTools)
- User requests perf analysis
- Benchmark regression detected
- New feature touches hot path

## Methodology

Profile → Analyze → Prioritize → Optimize → Benchmark → Improvement? → Verify & Ship (or re-prioritize)

### 1: Profile
Language-appropriate tool (load `languages/*.md`). Output: raw CPU/heap/trace profile.

### 2: Analyze
1. **Focus `cum`** — total cost of fn + everything it calls. Finds expensive flows.
2. **Contextualize `flat`** — fn's own cost. Trace runtime fns (GC, malloc) UP to user code.
3. **Ignore runtime noise** — scheduler overhead always appears. Note GC pressure but don't "fix" scheduler.
4. **Separate benchmark artifacts** — test harness allocations (httptest, ResponseRecorder) aren't production cost.

Output: `docs/research_logs/{component}-perf-analysis.md`

### 3: Prioritize

| Priority | Criteria |
|---|---|
| Do first | Low risk, high impact (caching, pre-alloc, fast-reject) |
| Do second | Medium risk, high impact (library swap, algorithm change) |
| Do last | High risk, high impact (major refactor, custom impl) |
| Skip | Any risk, low impact (micro-opt below noise) |

Rule: >1 day AND <20% hot path savings → defer.

### 4: Optimize
One fix at a time. TDD (Red→Green→Refactor). Run tests. Benchmark immediately. Never batch multiple opts in one commit.

### 5: Benchmark
Same config before/after (`-benchtime`, `-count`, machine load). Report: `ns/op`, `B/op`, `allocs/op`.

### 6: When to Stop
- Remaining CPU in hardware assembly (AES-NI, SIMD) — can't beat hardware
- Remaining allocs from runtime (GC, goroutine stacks, HTTP internals)
- Fix requires custom impl of audited library — security risk > perf gain
- Improvement <5% and within noise

## Pattern Catalog

### Result Caching
**Symptom:** Repeated expensive computation with identical inputs.
**Fix:** Bounded LRU with TTL.
**Safety:** Security-sensitive results: re-validate expiry, bound cache size (DoS), TTL < credential validity.

### Pre-allocation
**Symptom:** High allocs/op from repeated object construction.
**Fix:** Build once at init, share read-only. Safe if immutable after construction.

### Fast-Reject / Short-Circuit
**Symptom:** Expensive validation on clearly invalid inputs.
**Fix:** Cheap structural pre-check (string length before regex, delimiter count before parse).

### Library Swap
**Symptom:** High alloc/CPU in third-party parsing/serialization.
**Fix:** Lower-allocation library (manual scanners, zero-copy, arena).
**Safety:** Security libs: restrict algorithms, verify well-audited, run full test suite.

### Pooling
**Symptom:** High GC from many short-lived same-type objects.
**Fix:** Object pool (sync.Pool, arena). Only for uniform size with clear acquire/release lifecycle.

### Batching
**Symptom:** Many small I/O ops dominating wall-clock.
**Fix:** Batch into fewer calls (batch INSERT, pipeline Redis, buffer writes).

### Artifact Partitioning by Change Frequency
**Symptom:** Small change invalidates large cached artifact.
**Fix:** Separate stable layer (deps) from volatile (app code). Examples: Vite `manualChunks`, Docker deps-first COPY, monorepo per-package builds.

### Dependency Discovery Parallelization
**Symptom:** Sequential resource discovery creates waterfalls.
**Fix:** Declare deps early for parallel fetch. Examples: `<link rel="preconnect">`, `go mod download`, connection pool warm-up, `dns-prefetch`.

### Concurrent-Fetch Dedup
**Symptom:** Duplicate API calls from co-mounted components.
**Fix:** Loading-state guard (semaphore) at store/service layer.
```
async function fetchData() {
    if (isLoading) return
    isLoading = true
    try { data = await api.getData() }
    finally { isLoading = false }
}
```
For advanced: TanStack Query `staleTime`.

## Anti-Patterns
1. Don't optimize runtime internals — fix user code triggering allocations.
2. Don't replace battle-tested crypto with custom.
3. Don't optimize without profiling first.
4. Don't combine multiple opts in one commit.
5. Don't disable security for performance.
6. Don't profile without stable baseline.

## Load Testing Patterns

### Scenario Design
| Scenario | Purpose | Example |
|---|---|---|
| **Smoke** | Verify system functions under minimal load | 1-5 VUs, 1 minute |
| **Load** | Validate against expected traffic | Target RPS, 15 minutes |
| **Stress** | Find breaking point | Ramp to 2x-3x expected, observe degradation |
| **Soak** | Detect memory leaks, resource exhaustion | Normal load, 4-12 hours |
| **Spike** | Test sudden traffic bursts | 0 → 10x → 0 in seconds |

### Tools
| Tool | Language | Strength |
|---|---|---|
| k6 | JavaScript | Modern, scriptable, CI-friendly |
| Locust | Python | Pythonic, distributed |
| wrk | Lua | Ultra-fast HTTP benchmarking |
| Gatling | Scala | JVM, detailed reports |

### Checklist
- [ ] Baseline established (steady-state metrics)
- [ ] Test data realistic (not synthetic stubs)
- [ ] Warm-up period included
- [ ] Multiple runs for statistical significance
- [ ] Results include p50, p95, p99, and max latency
- [ ] Error rate tracked alongside latency

## Capacity Planning Checklist

1. **Current baseline**: requests/sec, CPU%, memory%, DB connections, queue depth
2. **Growth model**: expected traffic growth rate (monthly/quarterly)
3. **Saturation point**: at what load does p99 latency exceed SLA?
4. **Scaling strategy**: horizontal (add instances) vs vertical (bigger instance)?
5. **Bottleneck identification**: which resource saturates first? (CPU, memory, I/O, network, DB)
6. **Cost projection**: cost per 1000 requests at current and projected scale
7. **Headroom target**: operate at ≤70% capacity (30% buffer for spikes)

## Performance Budget Enforcement

Define per-endpoint or per-page budgets and enforce in CI:

| Metric | Budget | Enforcement |
|---|---|---|
| API p99 latency | < 200ms | Load test in CI |
| Frontend LCP | < 2.5s | Lighthouse CI |
| Bundle size | < 200KB (gzipped) | Build step check |
| Startup time (CLI) | < 100ms | Benchmark in CI |
| Memory per request | < 5MB | Profiling check |

Fail CI if budget exceeded. Track trends over time.

## Benchmark Regression Detection

### Setup
1. Store benchmark results in version control (JSON/CSV)
2. Compare against `main` branch baseline on every PR
3. Flag regressions > 10% as warnings, > 20% as failures

### Workflow
```
PR opened → Run benchmarks → Compare vs baseline → Pass/Warn/Fail → Report in PR comment
```

### Tools
| Language | Tool | Command |
|---|---|---|
| Go | `benchstat` | `benchstat old.txt new.txt` |
| Rust | `criterion` | Built-in comparison |
| Python | `pytest-benchmark` | `--benchmark-compare` |
| Node.js | `benchmark.js` | Custom comparison script |

## Language Modules

| Module | When |
|---|---|
| [Go](languages/go.md) | Go services, APIs, CLI |
| [Rust](languages/rust.md) | Rust binaries, libraries |
| [Python](languages/python.md) | Python services, CLI, data |
| [Frontend](languages/frontend.md) | Web (JS/TS bundle, rendering, network) |

## Profiling Scripts

| Script | Purpose |
|---|---|
| [go-pprof.sh](scripts/go-pprof.sh) | Go pprof CPU/heap → markdown |
| [frontend-lighthouse.sh](scripts/frontend-lighthouse.sh) | Lighthouse (Core Web Vitals) or bundle analysis |

## Related
- Performance Optimization Principles @.gemini/skills/performance-optimization-principles/SKILL.md
- Chaos Testing @.gemini/skills/chaos-testing/SKILL.md (for resilience under load)

