---
paths:
  - "**/*.jsx"
  - "**/*.tsx"
  - "**/react*"
---

## React Idioms and Patterns

React 18+ rewards composition, hooks, and server components. Idiomatic React = functional, performant, accessible.

> Scope: React-specific patterns. For TypeScript: `@.gemini/skills/typescript-idioms/SKILL.md`. For general frontend: `@.gemini/skills/frontend-design/SKILL.md`.

### Component Patterns

1. **Functional components only** — no class components in new code.
2. **Composition over inheritance:**
   ```tsx
   // ✅ Compound components
   <Card>
     <Card.Header>{title}</Card.Header>
     <Card.Body>{children}</Card.Body>
   </Card>
   ```

3. **Error boundaries for graceful failure:**
   ```tsx
   <ErrorBoundary fallback={<ErrorMessage />}>
     <TaskList />
   </ErrorBoundary>
   ```

### Hooks

1. **Custom hooks for reusable logic:**
   ```tsx
   function useTask(id: string) {
     const { data, error, isLoading } = useSWR(`/api/tasks/${id}`, fetcher);
     return { task: data, error, isLoading };
   }
   ```

2. **`useMemo`/`useCallback` only for measured performance issues** — not by default.

3. **`useEffect` cleanup** — always return cleanup function for subscriptions.

### State Management

1. **Local state first** (`useState`), lift only when needed.
2. **Server state**: TanStack Query / SWR — never in global state.
3. **Client state**: Context for small, Zustand/Jotai for complex.

### Performance

1. **`React.memo`** only when profiling shows unnecessary re-renders.
2. **Code splitting**: `React.lazy` + `Suspense` for route-level splitting.
3. **Virtual scrolling** for long lists (TanStack Virtual).

### Testing

React Testing Library + Vitest/Jest. Test behavior, not implementation.

```tsx
test('displays task title', () => {
  render(<TaskCard task={mockTask} />);
  expect(screen.getByText('Deploy fix')).toBeInTheDocument();
});
```

### Related
- TypeScript Idioms @.gemini/skills/typescript-idioms/SKILL.md
- Frontend Design @.gemini/skills/frontend-design/SKILL.md
- Accessibility Principles GEMINI.md § Accessibility Principles
