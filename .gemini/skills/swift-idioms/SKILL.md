---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

## Swift Idioms and Patterns

Swift rewards value types, optionals, and protocol-oriented design. Idiomatic Swift = safe, expressive, Swifty.

> Scope: Swift coding idioms. Test naming: GEMINI.md § Testing Strategy.

### Value Types and Optionals

1. **Prefer structs over classes** — value semantics by default. Classes only for identity, inheritance, or reference counting.
2. **Optionals — never force-unwrap (`!`) in production:**
   ```swift
   // ✅ Guard let for early exit
   guard let task = storage.findById(id) else {
       throw TaskError.notFound(id)
   }

   // ✅ Optional chaining
   let title = task?.title ?? "Untitled"

   // ❌ Force unwrap — crash risk
   let task = storage.findById(id)!
   ```

### Error Handling

1. **Typed throws (Swift 6) or `Error` protocol:**
   ```swift
   enum TaskError: Error {
       case notFound(String)
       case validationFailed(field: String, message: String)
   }

   func getTask(id: String) throws(TaskError) -> Task { ... }
   ```

2. **`Result` type for async callbacks:**
   ```swift
   func fetchTask(id: String) async -> Result<Task, TaskError> { ... }
   ```

### Concurrency

1. **Structured concurrency with `async`/`await`:**
   ```swift
   func loadDashboard() async throws -> Dashboard {
       async let user = fetchUser(id)
       async let tasks = fetchTasks(userId: id)
       return Dashboard(user: try await user, tasks: try await tasks)
   }
   ```

2. **`@Sendable`** for closures crossing concurrency domains.
3. **Actors** for thread-safe mutable state.

### Naming (Swift API Design Guidelines)

1. **camelCase** for functions, properties, variables.
2. **PascalCase** for types, protocols, enums.
3. **Omit needless words** — `remove(at:)` not `removeItem(atIndex:)`.
4. **Protocols for capabilities** use `-able`/`-ible`: `Codable`, `Identifiable`.

### Testing

XCTest or Swift Testing (6.0+). Mock via protocols.

### Formatting and Static Analysis

| Tool | Purpose | Command |
|---|---|---|
| `swift-format` | Formatting | `swift-format -i -r Sources/` |
| SwiftLint | Linting | `swiftlint lint --strict` |
| Xcode Analyzer | Static analysis | Built-in |

### Related
- Code Idioms and Conventions GEMINI.md § Code Idioms and Conventions
- Testing Strategy GEMINI.md § Testing Strategy
- Error Handling Principles GEMINI.md § Error Handling Principles
