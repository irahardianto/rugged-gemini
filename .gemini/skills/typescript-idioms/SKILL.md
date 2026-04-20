---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---

## TypeScript Idioms and Patterns

TS type system = documentation + test + specification. Encode domain invariants so invalid states are unrepresentable. Lean into the compiler.

> Scope: TS-specific type system and language idioms. Vue patterns: `vue-idioms-and-patterns.md`. File layout: `project-structure-vue-frontend.md`. Quality commands: `code-completion-mandate.md`. Logging: `logging-and-observability-principles.md`.

### Strict Mode — Non-Negotiable

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

Never disable per-file without `// STRICT-DISABLE:` rationale comment.

### Type System

1. **`unknown` over `any` — always:**
   ```typescript
   // ✅ Forces narrowing
   function parse(data: unknown): User {
       if (!isUser(data)) throw new Error('Invalid user shape');
       return data;
   }

   // ❌ Disables type checker
   function parse(data: any): User { return data; }
   ```

2. **`readonly` for immutability:**
   ```typescript
   interface TaskState {
       readonly id: string;
       readonly items: readonly Task[];
   }

   function process(tasks: readonly Task[]): Summary { ... }
   ```

3. **Discriminated unions for state machines:**
   ```typescript
   type AsyncState<T> =
       | { status: 'idle' }
       | { status: 'loading' }
       | { status: 'success'; data: T }
       | { status: 'error'; error: Error };

   function render(state: AsyncState<User>): string {
       switch (state.status) {
           case 'idle':    return 'Waiting...';
           case 'loading': return 'Loading...';
           case 'success': return state.data.name;
           case 'error':   return state.error.message;
       }
   }
   ```

4. **Const assertions for literal types:**
   ```typescript
   const ROLES = ['admin', 'editor', 'viewer'] as const;
   type Role = typeof ROLES[number]; // 'admin' | 'editor' | 'viewer'
   ```

5. **Type guards over `as` casts:**
   ```typescript
   // ✅ Safe narrowing
   function isError(value: unknown): value is Error {
       return value instanceof Error;
   }

   // ❌ Bypasses type checker
   const err = value as Error;
   ```

6. **Never non-null assertion `!` in production:**
   ```typescript
   // ❌ Hides null/undefined bug
   const name = user!.profile!.name;

   // ✅ Explicit handling
   const name = user?.profile?.name ?? 'Anonymous';
   ```

7. **`satisfies` for type-checked literals (TS 4.9+):**
   ```typescript
   const config = {
       endpoint: '/api/tasks',
       retries: 3,
   } satisfies ApiConfig;
   // config.retries typed as `3` (literal), not `number`
   ```

### Null Safety

1. **`??` over `||` for defaults** — `??` only falls back for null/undefined, `||` also for 0, '', false.
   ```typescript
   const count = input.count ?? 0;  // ✅
   const count = input.count || 0;  // ❌
   ```

2. **Optional chaining `?.`** for safe navigation: `user?.address?.city`

3. **`undefined` = absence, `null` = intentionally empty (JSON APIs).**

### Async/Await

> General async: `concurrency-and-threading-mandate.md`. TS-specific below.

1. **Always `await` or handle Promises — no floating promises:**
   ```typescript
   // ❌ Fire-and-forget — errors swallowed
   sendEmail(user);

   // ✅ Awaited
   await sendEmail(user);

   // ✅ Intentional fire-and-forget
   void sendEmail(user); // logs errors internally
   ```

2. **`Promise.all` for concurrent independent ops:**
   ```typescript
   const [user, tasks] = await Promise.all([getUser(id), getTasks(id)]);
   ```

3. **`Promise.allSettled` for partial failure tolerance:**
   ```typescript
   const results = await Promise.allSettled(notifications.map(send));
   const failed = results.filter(r => r.status === 'rejected');
   ```

4. **Never mix async/await with raw `.then()/.catch()` in same fn.**

### Runtime Validation at Boundaries

All data crossing system boundary validated at runtime, not just typed.

```typescript
import { z } from 'zod';

const CreateTaskSchema = z.object({
    title: z.string().min(1).max(200),
    priority: z.enum(['low', 'medium', 'high']),
    dueDate: z.string().datetime().optional(),
});

type CreateTaskRequest = z.infer<typeof CreateTaskSchema>;

function parseCreateTask(body: unknown): CreateTaskRequest {
    return CreateTaskSchema.parse(body);
}
```

- Use `zod` for runtime validation at API ingress/egress.
- Never use `as` as substitute for runtime validation.
- Validate on ingress; trust validated types thereafter.

### Centralized HTTP Client

All outbound HTTP MUST go through shared API client utility. No direct `fetch()`/`axios()` in feature code.

```typescript
// ❌ Bypass: no auth, no correlation-ID, no logging
const res = await fetch('/api/tasks');

// ✅ Shared client
import { apiFetch } from '@/infrastructure/apiFetch';
const res = await apiFetch('/api/tasks');
```

Why: consistent auth injection, correlation-ID propagation, centralized error normalization, single retry/timeout/logging point.

Exception: centralized client itself may use raw fetch internally.

> Direct `fetch`/`axios` outside shared client = `[INT]` audit finding.

### Module and Export Patterns

1. **Named exports over default:**
   ```typescript
   // ✅ Explicit, refactor-safe, IDE-friendly
   export function createTask() { ... }
   export type { Task };

   // ❌ Ambiguous import names
   export default function createTask() { ... }
   ```

2. **Avoid barrel re-exports creating circular deps.** Feature `index.ts` only for public API.

3. **Import type separately:**
   ```typescript
   import type { Task } from './types';
   ```

### Testing

> Test naming, file conventions, pyramid: `testing-strategy.md`. TS-specific tooling below.

1. **Type mocks with Vitest types — never `as any`:**
   ```typescript
   import { vi } from 'vitest';
   import type { MockedObject } from 'vitest';

   const mockStore: MockedObject<TaskStore> = {
       create: vi.fn(),
       getById: vi.fn(),
   };
   ```

2. **Assert error types, not just messages:**
   ```typescript
   await expect(service.create(invalid)).rejects.toThrow(ZodError);
   ```

3. **`satisfies` for type-checked fixtures:**
   ```typescript
   const fixture = {
       id: 'abc', title: 'Test task'
   } satisfies Task;
   ```

### Formatting and Static Analysis

| Tool | Purpose | Notes |
|---|---|---|
| `vue-tsc --noEmit` | Full type checking (incl `.vue`) | `tsc --noEmit` for non-Vue |
| `eslint` | Lint + style | Use `@typescript-eslint/recommended-type-checked` |
| `prettier` | Canonical formatting | Non-negotiable |
| `npm audit` / `pnpm audit` | Dependency CVE scanning | Fail on high severity |

See `code-completion-mandate.md` for exact commands.

### Related
- Code Idioms and Conventions @.gemini/rules/code-idioms-and-conventions.md
- Vue Idioms and Patterns @.gemini/rules/vue-idioms-and-patterns.md
- Testing Strategy @.gemini/rules/testing-strategy.md
- Error Handling Principles @.gemini/rules/error-handling-principles.md
- Concurrency and Threading Mandate @.gemini/rules/concurrency-and-threading-mandate.md
- Security Principles @.gemini/rules/security-principles.md
- Dependency Management Principles @.gemini/skills/dependency-management-principles/SKILL.md
