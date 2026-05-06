---
name: concurrency-and-threading-principles
description: >-
  Concurrency/parallelism patterns: race prevention, deadlock avoidance,
  message passing, async/channels/mutexes/worker pools.
user-invocable: false
---

## Concurrency and Threading Principles

### 1. Race Conditions

Multiple threads + shared data + write + no sync = race.

Prevention: locks/mutexes, immutability, message passing, thread-local storage.

Detection: Go=`-race`, Rust=Miri, C/C++=TSan, Java=JCStress/FindBugs.

### 2. Deadlocks

Two+ threads waiting for each other. Four conditions (ALL required): mutual exclusion, hold-and-wait, no preemption, circular wait.

Prevention (break any one): lock ordering (same order always), try_lock with timeout, avoid nested locks, lock-free data structures.

### 3. Prefer Immutability

Immutable = thread-safe. No sync needed. Share freely. If must change → message passing.

### 4. Message Passing > Shared Memory

"Share memory by communicating" (Go proverb). Channels/queues instead of shared memory. Reduces lock need. Easier to reason about and test.

### 5. Graceful Degradation

Timeouts, retries, circuit breakers. Don't crash app on one thread failure. Supervisors for fault tolerance. Backpressure for producer-consumer.

### Models by Use Case
- **I/O-bound:** async/await, event loops, coroutines, green threads
- **CPU-bound:** OS threads, thread pools, parallel processing
- **Actor:** Erlang/Akka (message passing, isolated state)
- **CSP:** Go channels, Rust channels

### Testing
- Controlled concurrency (deterministic execution)
- Timeout + resource exhaustion scenarios
- Pool full, queue full scenarios

### Related
- Resources @.gemini/skills/resources-and-memory-management/SKILL.md
- Error Handling GEMINI.md § Error Handling Principles
- Testing Strategy GEMINI.md § Testing Strategy