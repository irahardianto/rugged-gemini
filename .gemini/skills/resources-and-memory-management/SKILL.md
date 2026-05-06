---
name: resources-and-memory-management
description: >-
  Resource lifecycle patterns: files, DB connections, sockets, locks.
  RAII, defer/finally, pooling, graceful shutdown, leak prevention.
user-invocable: false
---

## Resource and Memory Management

### 1. Always Clean Up

Resources: files, connections, locks, semaphores, memory, OS handles, GPU.

Clean up in ALL paths: success, error, early return.

Language patterns: Go=defer, Rust=Drop(RAII), Python=with, TS=try/finally, Java=try-with-resources.

### 2. Timeout All I/O

- Network: 30s default, 5-10s interactive
- DB queries: 10s default, per-query complexity
- Files: timeout on network FS
- MQ: configurable, avoid indefinite blocking

### 3. Pool Expensive Resources

- DB connections: pool 5-20 per instance
- HTTP: keep-alive reuse
- Threads: CPU count (CPU-bound) or I/O wait (I/O-bound)

Pool config: min 5 (warm), max 20-50, idle timeout 5-10 min, validate before use, monitor utilization/wait/timeout.

### 4. Prevent Leaks

Leak = acquire + never release → exhaust system resources. Detect: monitor fd/connection/memory over time, long-duration tests, leak tools (valgrind, ASan, heap profilers). Prevent: use language cleanup patterns, never rely on manual alone.

### 5. Handle Backpressure

Problem: producer > consumer → unbounded queue → OOM.

Solutions: bounded queues, rate limiting, flow control, circuit breakers, drop/reject (fail fast > crash).

### Memory by Language

**GC'd (Go, Java, Python, JS):** auto-freed, still must release non-memory resources, GC pauses in latency-sensitive, profile for retained refs.

**Manual (C/C++):** malloc/free, RAII in C++, prefer smart pointers.

**Ownership (Rust):** compiler-enforced safety, no GC, no manual mgmt. Arc/Rc for shared ownership.

### Related
- Concurrency Mandate GEMINI.md § Concurrency and Threading Mandate
- Concurrency Principles @.gemini/skills/concurrency-and-threading-principles/SKILL.md
- Error Handling GEMINI.md § Error Handling Principles
