---
name: logging-and-observability-principles
description: >-
  Structured logging and observability patterns: log levels, structured fields,
  correlation IDs, language-specific logger setup.
user-invocable: false
---

## Logging and Observability Principles

> ⚠️ All operations MUST be logged per Logging and Observability Mandate GEMINI.md § Logging and Observability Mandate.

### Log Levels

| Level | When | Examples |
|---|---|---|
| TRACE | Extremely detailed diagnostics | Fn entry/exit, var states (dev only) |
| DEBUG | Detailed flow for debugging | Query exec, cache hits, state transitions |
| INFO | General informational | Request started, task created, login |
| WARN | Potentially harmful | Deprecated API, fallback, retry |
| ERROR | Error, app continues | Request failed, API timeout, validation |
| FATAL | Severe, causes shutdown | DB unreachable, critical config missing |

### Rules

**1. Every operation logs start + success/error:**
```
log.Info("creating task",
"correlationId", correlationID,
"userId", userID,
"title", task.Title,
)

log.Info("task created successfully",
"correlationId", correlationID,
"taskId", task.ID,
"duration", time.Since(start),
)

log.Error("failed to create task",
"correlationId", correlationID,
"error", err,
"userId", userID,
)
```

**2. Context fields:** `correlationId` (UUID), `userId`, `duration` (ms), `error`.

**3. Structured only:**
```
// ✅ Structured
log.Info("user login", "userId", userID, "ip", clientIP)

// ❌ String formatting
log.Info(fmt.Sprintf("User %s logged in from %s", userID, clientIP))
```

**4. Never log:** passwords, API keys, tokens, credit cards, PII (sanitize if needed), full req/res bodies.

**5. Never log in hot paths:** tight loops, per-item batch processing, latency-critical sync paths. Use logger middleware redaction (pino-redact, zap masking).

### Language Implementations

**Go (slog):**
```
import "log/slog"

logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
  Level: slog.LevelInfo,
}))

logger.Info("operation started",
  "correlationId", correlationID,
  "userId", userID,
)
```

**TypeScript (pino):**
```
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
});

logger.info({
  correlationId,
  userId,
  duration: Date.now() - startTime,
}, 'task created successfully');
```

**Python (structlog):**
```python
import structlog

logger = structlog.get_logger()

logger.info("task_created",
correlation_id=correlation_id,
user_id=user_id,
task_id=task.id,
)
```

### Log Patterns

**API:** log request received (method, path, correlationId) + request completed (status, duration).

**DB:** DEBUG level — query start (query text) + complete (rows, duration) + error (query, error).

**External API:** INFO start (service, endpoint). WARN retry (attempt, error). WARN circuit breaker (failureCount).

**Background Jobs:** INFO start (jobId, type) + progress (periodic, not per-item) + complete (duration, items).

**Errors:** ERROR recoverable (sanitized input). FATAL critical dependency (action: shutting down).

### Environment Config

| Env | Level | Format | Destination |
|---|---|---|---|
| Development | DEBUG | Pretty (colored) | Console |
| Staging | INFO | JSON | Stdout → cloud logs |
| Production | INFO | JSON | Stdout → cloud logs |

```
func configureLogger() *slog.Logger {
var handler slog.Handler

    level := slog.LevelInfo
    if os.Getenv("ENV") == "development" {
        level = slog.LevelDebug
        handler = slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
            Level: level,
        })
    } else {
        handler = slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
            Level: level,
        })
    }
    
    return slog.New(handler)
    }
```

### Testing Logs

Capture and assert on log output:
```
func TestUserLogin(t *testing.T) {
var buf bytes.Buffer
logger := slog.New(slog.NewJSONHandler(&buf, nil))

    service := NewUserService(logger, mockStore)
    err := service.Login(ctx, email, password)
    
    require.NoError(t, err)
    logs := buf.String()
    assert.Contains(t, logs, "user login successful")
    assert.Contains(t, logs, email)
    }
```

### Monitoring

**Correlation IDs:** generate at ingress, propagate through all services, include in all logs/errors/traces. UUID v4.

**Aggregation:** centralized (CloudWatch/GCP/Datadog). Index: correlationId, userId, level, timestamp. Alert on ERROR/FATAL. Dashboard: request rates, error rates, latency.

### Checklist
- [ ] All public ops log INFO on start
- [ ] All ops log INFO/ERROR on complete/failure
- [ ] All logs include correlationId
- [ ] No sensitive data
- [ ] Structured (key-value pairs)
- [ ] Appropriate level
- [ ] Error logs include error details
- [ ] Perf-critical paths use DEBUG

### Related
- Logging and Observability Mandate GEMINI.md § Logging and Observability Mandate
- Monitoring and Alerting Principles @.gemini/skills/monitoring-and-alerting-principles/SKILL.md
- Error Handling Principles GEMINI.md § Error Handling Principles
- Security Mandate GEMINI.md § Security Mandate
- Security Principles GEMINI.md § Security Principles
- API Design Principles @.gemini/skills/api-design-principles/SKILL.md
