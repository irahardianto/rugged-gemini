---
paths:
  - "**/*.cpp"
  - "**/*.hpp"
  - "**/*.h"
  - "**/CMakeLists.txt"
---

## C++ Idioms and Patterns

Modern C++ (17/20/23) rewards RAII, value semantics, and zero-cost abstractions. Lean into the type system and smart pointers. Idiomatic C++ = safe, deterministic, performant.

> Scope: C++ coding idioms. Test naming: GEMINI.md § Testing Strategy.

### Ownership and Memory

1. **RAII — resources tied to object lifetime. No manual `new`/`delete`.**
2. **`std::unique_ptr` for exclusive ownership, `std::shared_ptr` only when sharing is necessary.**
3. **Move semantics — prefer moving over copying for expensive types.**

### Error Handling

1. **`std::expected` (C++23) or Result types for expected failures.**
2. **Exceptions for exceptional conditions only.** Never for control flow.
3. **`noexcept` on move constructors and destructors.**

### Modern Features

1. **`std::string_view`** for read-only string parameters (no allocation).
2. **`constexpr`** for compile-time evaluation.
3. **Structured bindings**: `auto [id, title] = getTask();`
4. **`std::optional`** for nullable values.

### Naming

1. **PascalCase** for types. **camelCase** or **snake_case** for functions (project-consistent).
2. **`_` suffix** for private members: `storage_`, `logger_`.
3. No Hungarian notation.

### Testing

Google Test or Catch2. Google Mock for interface mocking.

### Formatting and Static Analysis

| Tool | Purpose | Command |
|---|---|---|
| `clang-format` | Formatting | `clang-format -i src/**/*.cpp` |
| `clang-tidy` | Static analysis | `clang-tidy src/*.cpp --` |
| `cppcheck` | Bug detection | `cppcheck --enable=all src/` |
| AddressSanitizer | Memory errors | `-fsanitize=address` |
| ThreadSanitizer | Race conditions | `-fsanitize=thread` |

### Related
- Code Idioms and Conventions GEMINI.md § Code Idioms and Conventions
- Resources and Memory Management @.gemini/skills/resources-and-memory-management/SKILL.md
- Concurrency and Threading Principles @.gemini/skills/concurrency-and-threading-principles/SKILL.md
