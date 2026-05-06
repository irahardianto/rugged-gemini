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

```vue
<!-- ✅ Canonical -->
<script setup lang="ts">
import { ref, computed } from 'vue';

const props = defineProps<{ title: string; count?: number }>();
const emit = defineEmits<{ 'update:count': [value: number] }>();

const doubled = computed(() => (props.count ?? 0) * 2);
</script>

<!-- ❌ Options API — not for new components -->
<script lang="ts">
export default { props: { title: String }, ... }
</script>
```

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

3. **Writable computed for two-way bindings:**
   ```typescript
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
       const increment = () => count.value++;
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
   const props = defineProps<{
       taskId: string;
       variant?: 'compact' | 'full';
   }>();

   const props = withDefaults(defineProps<{ variant?: 'compact' | 'full' }>(), {
       variant: 'full',
   });
   ```

2. **`defineEmits` typed:**
   ```typescript
   const emit = defineEmits<{
       'update:modelValue': [value: string];
       'submit': [task: CreateTaskRequest];
   }>();
   ```

3. **v-model contract:** `modelValue` prop + `update:modelValue` emit.

4. **`defineExpose`** for selective parent access. Everything private by default.

5. **`v-bind="$attrs"` + `inheritAttrs: false`** for attribute forwarding.

6. **One concern per component.** Template over 100 lines -> extract sub-component.

7. **No business logic in template** — computed/composables in `<script setup>`.

### Template Patterns

1. **`:key` with stable unique IDs in `v-for`** — never index when list order changes:
   ```html
   <TaskCard v-for="task in tasks" :key="task.id" :task="task" />
   ```

2. **Never combine `v-if` and `v-for` on same element** — wrap with `<template>`.

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
