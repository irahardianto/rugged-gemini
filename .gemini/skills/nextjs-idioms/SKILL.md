---
paths:
  - "**/next.config.*"
  - "**/app/**/page.tsx"
  - "**/app/**/layout.tsx"
---

## Next.js Idioms and Patterns

Next.js (14+) rewards App Router, Server Components, and Server Actions. Idiomatic Next.js = server-first, streaming, edge-ready.

> Scope: Next.js-specific patterns. For React: `@.gemini/skills/react-idioms/SKILL.md`.

### App Router (Default)

1. **Server Components by default** — add `'use client'` only when needed (hooks, events, browser APIs).
2. **Layouts for shared UI** — never duplicate headers/sidebars.
3. **Loading/Error boundaries** per route segment:
   ```
   app/tasks/
   ├── page.tsx        # Server Component
   ├── loading.tsx     # Suspense fallback
   ├── error.tsx       # Error boundary
   └── layout.tsx      # Shared layout
   ```

### Data Fetching

1. **Server Components fetch data directly** — no useEffect:
   ```tsx
   // app/tasks/page.tsx (Server Component)
   export default async function TasksPage() {
     const tasks = await db.tasks.findMany();
     return <TaskList tasks={tasks} />;
   }
   ```

2. **Server Actions for mutations:**
   ```tsx
   'use server';
   export async function createTask(formData: FormData) {
     await db.tasks.create({ title: formData.get('title') });
     revalidatePath('/tasks');
   }
   ```

3. **`revalidatePath`/`revalidateTag`** for cache invalidation.

### Performance

1. **Static generation by default** — dynamic only when needed.
2. **Image optimization**: `next/image` — always.
3. **Route prefetching** via `next/link`.

### SEO

1. **Metadata API** for per-page SEO:
   ```tsx
   export const metadata: Metadata = {
     title: 'Tasks',
     description: 'Manage your tasks',
   };
   ```

### Related
- React Idioms @.gemini/skills/react-idioms/SKILL.md
- TypeScript Idioms @.gemini/skills/typescript-idioms/SKILL.md
