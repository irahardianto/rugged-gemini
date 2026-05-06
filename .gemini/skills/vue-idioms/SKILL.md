---
paths:
  - "**/*.vue"
  - "**/*.ts"
  - "**/*.tsx"
---

## Vue Idioms and Patterns

Vue 3 Composition API default. `<script setup>` canonical syntax. Think reactive data flows, not lifecycle hooks. Composables (`use*`) = primary logic reuse unit.

> Scope: Vue 3 coding idioms. TS type system: `@.gemini/skills/typescript-idioms/SKILL.md`. Layout: `@.gemini/skills/project-structure-vue/SKILL.md`. Tests: GEMINI.md § Testing Strategy. Logging: `@.gemini/skills/logging-and-observability-principles/SKILL.md`.

### `<script setup>` — Only Style

Always `<script setup lang="ts">`. Never Options API or class-style for new code.

**Canonical ordering inside `<script setup>`:**

```vue
<script setup lang="ts">
// 1. Framework imports
import { ref, computed, onMounted } from 'vue';

// 2. Third-party imports
import { useIntersectionObserver } from '@vueuse/core';

// 3. Internal imports — feature-relative paths
import type { Task } from './types';
import { useAuth } from '@/composables/useAuth';

// 4. Props & Emits — typed interfaces
interface Props {
    title: string;
    variant?: 'compact' | 'full';
}
const props = withDefaults(defineProps<Props>(), { variant: 'full' });
const emit = defineEmits<{ select: [item: Task]; close: [] }>();

// 5. Composables
const { user } = useAuth();

// 6. Reactive state
const isVisible = ref(false);

// 7. Computed
const doubled = computed(() => (props.count ?? 0) * 2);

// 8. Methods — always named functions (not arrow)
function handleSelect(item: Task) {
    emit('select', item);
}

// 9. Lifecycle (always last before template)
onMounted(() => { /* setup */ });
</script>

<!-- ❌ Options API — not for new components -->
<script lang="ts">
export default { props: { title: String }, ... }
</script>
```

**Named functions for methods:** Use `function handleClick()` — not `const handleClick = () =>`. Named functions produce readable stack traces, hoist predictably, and clearly signal "this is an action". Reserve arrow functions for callbacks and inline expressions.

### Reactivity: `ref` vs `reactive`

| Use | When |
|---|---|
| `ref<T>()` | Primitives, single values, values that may be reassigned |
| `reactive()` | Plain objects where you always access properties (never reassign whole object) |
| `readonly()` | State that must not be mutated outside owner |

```typescript
// ✅ ref for primitives and replaceable objects
const count = ref(0);
const user = ref<User | null>(null);
user.value = fetchedUser;

// ✅ reactive for objects
const form = reactive({ title: '', priority: 'medium' });

// ❌ Never destructure reactive — reactivity lost
const { title } = form;
// ✅ Use toRefs
const { title } = toRefs(form);
```

### Computed

1. **All derived state via `computed`** — never recompute in template:
   ```typescript
   const filteredTasks = computed(() =>
       tasks.value.filter(t => t.status === activeFilter.value)
   );
   ```

2. **No side effects in computed** — must be pure.

3. **Writable computed** — escape-hatch for complex two-way bindings only. For standard v-model, use `defineModel()` (see Component Design below):
   ```typescript
   // Only when defineModel() is insufficient (e.g., cross-store sync)
   const modelValue = computed({
       get: () => props.modelValue,
       set: (val) => emit('update:modelValue', val),
   });
   ```

### Watch Strategy

| Watcher | When |
|---|---|
| `watchEffect` | Re-run on any dependency change; auto-tracks |
| `watch` | Need old value, lazy execution, or explicit source |
| `computed` | Synchronous derived value (prefer over watch for transformation) |

```typescript
// watchEffect — auto-tracks
watchEffect(() => {
    document.title = `Tasks (${count.value})`;
});

// watch — explicit source
watch(userId, async (newId, oldId) => {
    if (newId !== oldId) await loadUser(newId);
}, { immediate: true });
```

### Pinia Stores

> Directory structure: `@.gemini/skills/project-structure-vue/SKILL.md`. Coding idioms below.

1. **Setup Store API** (not Options):
   ```typescript
   export const useTaskStore = defineStore('task', () => {
       const tasks = ref<Task[]>([]);
       const isLoading = ref(false);

       const completedTasks = computed(() =>
           tasks.value.filter(t => t.status === 'done')
       );

       async function loadTasks() {
           isLoading.value = true;
           try {
               tasks.value = await taskAPI.getTasks();
           } finally {
               isLoading.value = false;
           }
       }

       return { tasks, isLoading, completedTasks, loadTasks };
   });
   ```

2. **Never mutate store state from outside** — call actions.

3. **Inject API dependency** — never import directly inside store:
   ```typescript
   export const useTaskStore = defineStore('task', () => {
       const api = inject<TaskAPI>(TASK_API_KEY);
       if (!api) throw new Error('[TaskStore] TASK_API_KEY not provided — ensure app.provide() is called before store access');
       // ...
   });
   ```

4. **`storeToRefs` for destructuring:**
   ```typescript
   const { tasks, isLoading } = storeToRefs(useTaskStore());
   const { loadTasks } = useTaskStore(); // actions don't need storeToRefs
   ```

### Composables (`use*`)

1. **Naming:** always prefix `use` — `useTaskFilters`, `useAuth`, `usePagination`.

2. **Return reactive refs:**
   ```typescript
   function useCounter(initial = 0) {
       const count = ref(initial);
       function increment() { count.value++; }
       return { count, increment };
   }
   ```

3. **Clean up side effects in `onUnmounted`:**
   ```typescript
   function useWindowResize() {
       const width = ref(window.innerWidth);
       const handler = () => (width.value = window.innerWidth);
       onMounted(() => window.addEventListener('resize', handler));
       onUnmounted(() => window.removeEventListener('resize', handler));
       return { width };
   }
   ```

4. **Template refs with `useTemplateRef` (Vue 3.5+):**
   ```typescript
   const inputEl = useTemplateRef<HTMLInputElement>('myInput');
   // <input ref="myInput" />
   ```

5. **Feature-specific composables inside feature dir.** Global composables in `src/composables/`.

### Component Design

1. **`defineProps` with TS generics:**
   ```typescript
   // Omit `const props =` if props are only used in template
   defineProps<{ title: string; count?: number }>();

   // Use `const props =` only if props are accessed in <script setup>
   const props = defineProps<{ taskId: string; variant?: 'compact' | 'full' }>();
   ```

2. **Prop defaults — destructuring (Vue 3.5+) or `withDefaults`:**
   ```typescript
   // ✅ Preferred: destructure with defaults (Vue 3.5+)
   const { title = 'Untitled', variant = 'full' } = defineProps<{
       title?: string;
       variant?: 'compact' | 'full';
   }>();

   // ✅ Also valid: withDefaults
   const props = withDefaults(defineProps<{ variant?: 'compact' | 'full' }>(), {
       variant: 'full',
   });
   ```

3. **`defineEmits` typed:**
   ```typescript
   const emit = defineEmits<{
       submit: [task: CreateTaskRequest];
       close: [];
   }>();
   ```

4. **`defineModel()` for v-model (Vue 3.4+):**
   ```typescript
   // ✅ Simple two-way binding — replaces manual modelValue prop + emit
   const title = defineModel<string>();

   // ✅ With options and modifiers
   const [title, modifiers] = defineModel<string>({
       default: 'default value',
       required: true,
       get: (value) => value.trim(),
       set: (value) => modifiers.capitalize
           ? value.charAt(0).toUpperCase() + value.slice(1) : value,
   });

   // ✅ Multiple v-model bindings
   const firstName = defineModel<string>('firstName');
   const age = defineModel<number>('age');
   // Usage: <UserForm v-model:first-name="user.firstName" v-model:age="user.age" />
   ```

5. **`defineExpose`** for selective parent access. Everything private by default.

6. **`v-bind="$attrs"` + `inheritAttrs: false`** for attribute forwarding.

7. **One concern per component.** Template over 100 lines -> extract sub-component.

8. **No business logic in template** — computed/composables in `<script setup>`.

### Template Conventions

1. **`:key` with stable unique IDs in `v-for`** — never index when list order changes:
   ```html
   <TaskCard v-for="task in tasks" :key="task.id" :task="task" />
   ```

2. **Never combine `v-if` and `v-for` on same element** — wrap with `<template>`.

3. **Prop shorthand** — when value matches prop name:
   ```html
   <!-- ✅ Shorthand -->
   <MyComponent :count />
   <!-- ❌ Redundant -->
   <MyComponent :count="count" />
   ```

4. **Slot shorthand** — `#` over `v-slot:`:
   ```html
   <!-- ✅ -->
   <template #header>...</template>
   <template #default>...</template>
   <!-- ❌ -->
   <template v-slot:header>...</template>
   ```

5. **Explicit `<template>` tags for ALL used slots** — never rely on implicit default.

6. **Case conventions:** camelCase in JS (props, emits), kebab-case in templates:
   ```html
   <!-- Template: kebab-case -->
   <UserCard :first-name="name" @update-profile="handleUpdate" />
   ```
   ```typescript
   // Script: camelCase
   defineProps<{ firstName: string }>();
   defineEmits<{ updateProfile: [] }>();
   ```

7. **Component naming direction:** General → Specific — `SearchButtonClear.vue` not `ClearSearchButton.vue`. Mirrors natural language hierarchy.

### Route Transitions

CSS frameworks with `@layer` (Tailwind v4, Open Props, UnoCSS) can break SPA navigation by overriding transition properties. `transitionend` never fires -> entering component permanently blocked.

1. **Avoid `mode="out-in"` with `@layer` frameworks.** Use simultaneous:
   ```html
   <!-- ✅ Safe -->
   <Transition name="fade">
     <component :is="Component" :key="$route.path" />
   </Transition>
   ```

2. **Always `:key="$route.path"`** on dynamic component inside Transition.

3. **`!important` on route transition CSS:**
   ```css
   .fade-enter-active {
     transition: opacity 0.15s ease-in !important;
   }
   .fade-leave-active {
     transition: opacity 0.15s ease-out !important;
     position: absolute !important;
     width: 100% !important;
     top: 0 !important;
     left: 0 !important;
   }
   .fade-enter-from,
   .fade-leave-to {
     opacity: 0 !important;
   }
   ```

4. **Transition parent needs `position: relative`.**

> Diagnosis: Debugging Protocol [Frontend module](file://.gemini/skills/debugging-protocol/languages/frontend.md) § CSS × Animation.

### File-Based Routing

Modern Vue projects use file-based routing where the file/folder structure defines routes. These conventions apply regardless of the specific tool (Unplugin Vue Router, Nuxt, etc.):

1. **Avoid `index.vue`** — use route groups for descriptive names:
   ```
   src/pages/
   ├── (home).vue          # Renders at / — descriptive name
   ├── about.vue            # Renders at /about
   ├── [...path].vue        # Catch-all (404)
   ├── users.vue            # Layout for nested user routes
   └── users/
       ├── (user-list).vue  # Renders at /users
       └── [userId].vue     # Renders at /users/:userId
   ```

2. **Named params over generic** — `[userId]` not `[id]`, `[postSlug]` not `[slug]`.

3. **Dot notation for flat routes** — `users.edit.vue` → `/users/edit` without nesting.

4. **Route groups for shared layouts** without affecting URL:
   ```
   src/pages/
   ├── (admin).vue          # Layout for admin routes
   ├── (admin)/
   │   ├── dashboard.vue    # /dashboard
   │   └── settings.vue     # /settings
   ```

5. **Typed route navigation** — prefer named route locations:
   ```typescript
   // ✅ Type-safe, refactor-safe
   router.push({ name: '/users/[userId]', params: { userId } });
   // ❌ String concatenation — fragile
   router.push('/users/' + userId);
   ```

6. **`definePage()`** to customize route properties (meta, name, alias) inline.

7. **Check `typed-router.d.ts`** for available route names and param types.

### Testing

> Naming/pyramid: GEMINI.md § Testing Strategy. Vue-specific below.

1. **`createTestingPinia`** for component tests:
   ```typescript
   import { vi } from 'vitest';
   const wrapper = mount(TaskView, {
       global: {
           plugins: [createTestingPinia({ createSpy: vi.fn })],
       },
   });
   ```

2. **Test behavior, not implementation** — query by accessible role, not CSS class.
3. **Test stores independently** — `setActivePinia(createPinia())`.

### Linting and Type Checking

| Tool | Purpose |
|---|---|
| `vue-tsc --noEmit` | Full-template type checking |
| `eslint-plugin-vue` | Vue-specific lint rules |
| `prettier` | Canonical formatting |

See GEMINI.md § Code Completion Mandate for exact commands.

### Related
- Code Idioms and Conventions GEMINI.md § Code Idioms and Conventions
- TypeScript Idioms and Patterns @.gemini/skills/typescript-idioms/SKILL.md
- Project Structure — Vue Frontend @.gemini/skills/project-structure-vue/SKILL.md
- Architectural Patterns GEMINI.md § Architectural Patterns
- Testing Strategy GEMINI.md § Testing Strategy
- Logging and Observability Principles @.gemini/skills/logging-and-observability-principles/SKILL.md
