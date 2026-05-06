---
paths:
  - "**/*.vue"
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.css"
  - "**/*.html"
---

## Vue/React Frontend Layout

Vertical slice — features are self-contained modules, not scattered across global folders.

```
apps/
  frontend/
    src/
      assets/                       # Fonts, images
      features/                     # Vertical slices. Each SELF-CONTAINED.
        task/
          components/               # Feature-specific components (NOT in top-level)
            TaskForm.vue
            TaskListItem.vue
            TaskFilters.vue
            TaskInput.vue
            TaskInput.spec.ts       # Component unit tests
          store/
            task.store.ts           # Pinia store
            task.store.spec.ts
          api/
            task.api.ts             # interface TaskAPI
            task.api.backend.ts     # Production impl
            task.api.mock.ts        # Test impl
          services/
            task.service.ts
            task.service.spec.ts
          types/                    # TS interfaces (DTOs)
          composables/              # Feature-specific hooks
          index.ts                  # Public exports only
        order/
      composables/                  # Global reactive logic (useAuth, useTheme)
      components/
        ui/                         # Shared atoms/molecules. NO domain logic.
          BaseButton.vue
          BaseButton.spec.ts
          types.ts
          index.ts
        layout/                     # Organisms (header, sidebar, error boundary)
          AppHeader.vue
          AppSidebar.vue
          ErrorBoundary.vue
          EmptyState.vue
      layouts/                      # App shells (Sidebar, Navbar wrappers)
        MainLayout.vue
        AuthLayout.vue
      views/                        # Route entry points ("Glue")
        HomeView.vue
        TaskView.vue
      utils/                        # Pure stateless helpers. No domain, no reactivity.
      router/
      plugins/                      # Library configs (Axios, I18n)
      App.vue
      main.ts
```

Key: `features/` = vertical slices, export via `index.ts`. `components/ui/` = shared dumb UI. `views/` compose features. Feature components inside feature, NOT top-level. Applies to React (.tsx), Vue (.vue), Svelte (.svelte) — swap state mgmt as needed.

### Related
- Project Structure GEMINI.md § Project Structure
- Vue Idioms and Patterns @.gemini/skills/vue-idioms/SKILL.md
- TypeScript Idioms and Patterns @.gemini/skills/typescript-idioms/SKILL.md
