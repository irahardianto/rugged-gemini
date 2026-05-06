---
paths:
  - "**/*.component.ts"
  - "**/angular.json"
---

## Angular Idioms and Patterns

Angular (17+) rewards signals, standalone components, and reactive patterns. Idiomatic Angular = typed, modular, RxJS-aware.

> Scope: Angular-specific patterns. For TypeScript: `@.gemini/skills/typescript-idioms/SKILL.md`.

### Standalone Components (Default)

1. **Standalone components** — no NgModules for new components:
   ```typescript
   @Component({
     selector: 'app-task-list',
     standalone: true,
     imports: [CommonModule, TaskCardComponent],
     template: `<app-task-card *ngFor="let task of tasks()" [task]="task" />`
   })
   export class TaskListComponent {
     tasks = signal<Task[]>([]);
   }
   ```

### Signals (17+)

1. **Signals for synchronous state** — prefer over BehaviorSubject for component state.
2. **`computed`** for derived state. **`effect`** for side effects.
3. **RxJS for async streams** — HTTP, WebSocket, complex event handling.

### Dependency Injection

1. **`inject()` function** over constructor injection in standalone components.
2. **Provide at appropriate level** — component, route, or root.

### Reactive Forms

1. **Reactive forms over template-driven** for complex forms.
2. **Typed forms** (`FormControl<string>`) — always.

### Testing

Jasmine/Karma or Jest. Angular Testing Library for component tests.

### Formatting and Static Analysis

| Tool | Purpose | Command |
|---|---|---|
| Prettier | Formatting | `npx prettier --write .` |
| ESLint + angular-eslint | Linting | `npx ng lint` |

### Related
- TypeScript Idioms @.gemini/skills/typescript-idioms/SKILL.md
- Frontend Design @.gemini/skills/frontend-design/SKILL.md
