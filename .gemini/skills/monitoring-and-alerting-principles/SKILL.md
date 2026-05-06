---
name: monitoring-and-alerting-principles
description: >-
  Health checks, metrics instrumentation (RED/USE), error tracking,
  graceful degradation. Prometheus, probes, alert thresholds.
user-invocable: false
---

## Monitoring and Alerting Principles

> Covers what agent implements in code. Org concerns (SLO defs, on-call, escalation) out of scope.

### Health Checks

- **`/health` (Liveness):** 200 if alive. No dependency checks. Orchestrator uses to restart.
- **`/ready` (Readiness):** checks all deps (DB, cache, MQ). 503 if unavailable. LB uses to route.

Rules: fast (<1s), no side effects, separate liveness from readiness.

### Metrics — RED/USE

**RED (services):** Rate (req/s), Errors (count/rate), Duration (histogram, not avg).

**USE (resources):** Utilization, Saturation (queued), Errors.

Rules: counters (monotonic), gauges (up/down), histograms (distributions). Consistent labels (service, method, status_code). No high-cardinality labels (no user IDs).

### Error Tracking
- Capture unhandled exceptions + stack traces
- Include context (userId, requestId, correlationId)
- Group by root cause, not instance
- Severity by user impact

### Graceful Degradation
- Circuit breakers for external deps
- Fallbacks for non-critical (cached data, reduced UI)
- Timeouts on all external calls
- Retry with exponential backoff + jitter

Tool-agnostic: Datadog, LGTM, Sentry, New Relic, CloudWatch — same code patterns.

### Checklist
- [ ] /health and /ready endpoints
- [ ] Liveness: no dep checks
- [ ] Readiness: checks all critical deps
- [ ] RED/USE metrics on key ops
- [ ] No high-cardinality labels
- [ ] Unhandled exceptions captured + context
- [ ] Circuit breakers on external deps
- [ ] Timeouts on all external calls

### Related
- Logging Mandate GEMINI.md § Logging and Observability Mandate
- Logging Principles @.gemini/skills/logging-and-observability-principles/SKILL.md
- Error Handling GEMINI.md § Error Handling Principles
- Resources @.gemini/skills/resources-and-memory-management/SKILL.md
- Concurrency @.gemini/skills/concurrency-and-threading-principles/SKILL.md
