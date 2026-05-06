---
paths:
  - "**/*.dart"
---

## Flutter Idioms and Patterns (Riverpod 3)

Flutter = UI toolkit first — performance is first-class. `const` widgets + immutable data keep render tree efficient. **Riverpod 3** = canonical state management: compile-safe, testable without BuildContext, no implicit global state, automatic retry + pause/resume.

**Code generation mandatory.** All providers use `@riverpod`/`@Riverpod(keepAlive: true)` with `riverpod_generator` + `build_runner`. Catches type errors, missing overrides, broken provider graphs at compile time.

```yaml
# pubspec.yaml
dependencies:
  flutter_riverpod: 3.2.1
  riverpod_annotation: 4.0.2

dev_dependencies:
  riverpod_generator: 4.0.3
  build_runner: # latest
  riverpod_lint: # latest
```

> Scope: Flutter/Dart coding idioms. Layout: `@.gemini/skills/project-structure-flutter/SKILL.md`. Tests: GEMINI.md § Testing Strategy. Error handling: GEMINI.md § Error Handling Principles.

### `const` Constructors — Everywhere

```dart
// ✅ const constructor — rebuild-safe
class TaskCard extends StatelessWidget {
    const TaskCard({super.key, required this.task});
    final Task task;
}

// Usage — compile-time constant
const TaskCard(task: myTask)
```

Rules: every StatelessWidget with no mutable state = `const` constructor. Pass `const` at call site. Enable `prefer_const_constructors` lint.

### Widget Decomposition

1. **Extract widget when subtree has distinct responsibilities:**
   ```dart
   // ✅ Each subtree = named widget with const constructor
   @override
   Widget build(BuildContext context) {
       return Column(children: [
           const TaskHeader(),
           const TaskList(),
           const TaskFooter(),
       ]);
   }
   ```

2. **Never use builder methods (`_buildHeader()`)** — they skip `const` optimization. Extract proper widgets.
3. **`build` under ~30 lines** — decompose if longer.

### Immutable Data with `freezed`

All domain models immutable via `freezed`: value objects with `copyWith`, union/sealed types, generated `==`/`hashCode`/`toString`.

```dart
@freezed
class Task with _$Task {
    const factory Task({
        required String id,
        required String title,
        @Default(TaskStatus.pending) TaskStatus status,
        DateTime? dueDate,
    }) = _Task;

    factory Task.fromJson(Map<String, dynamic> json) => _$TaskFromJson(json);
}

final updated = task.copyWith(status: TaskStatus.done);
```

Rules: all domain models `@freezed`. Never expose mutable fields. Run `dart run build_runner build` after changes.

### Provider Decision Tree

| Has side-effects? | Async? | Pattern | Generated type |
|---|---|---|---|
| No | No | `@riverpod` function | `Provider` |
| No | Yes | `@riverpod` async function | `FutureProvider` |
| No | Stream | `@riverpod` Stream function | `StreamProvider` |
| Yes | No | `@riverpod` class | `NotifierProvider` |
| Yes | Yes | `@riverpod` class with Future build | `AsyncNotifierProvider` |
| Yes | Stream | `@riverpod` class with Stream build | `StreamNotifierProvider` |

### Riverpod — State Management

Riverpod 3 only. No BLoC, Cubit, Provider (package:provider), or GetX.

#### App Entry Point

```dart
void main() {
    runApp(const ProviderScope(child: MyApp()));
}
```

One `ProviderScope` at root. Never nest. `overrides:` only in tests. `retry:` on root for retry config.

#### Class-Based (Side-Effects)

```dart
// Synchronous notifier
@riverpod
class TaskFilter extends _$TaskFilter {
    @override
    TaskFilterState build() {
        return const TaskFilterState();
    }

    void setStatus(TaskStatus? status) {
        state = state.copyWith(status: status);
    }

    void toggleShowCompleted() {
        state = state.copyWith(showCompleted: !state.showCompleted);
    }
}
```

```dart
// Async notifier — CRUD
@riverpod
class TaskList extends _$TaskList {
    @override
    Future<List<Task>> build() async {
        return ref.watch(taskRepositoryProvider).getTasks();
    }

    Future<void> addTask(CreateTaskRequest request) async {
        state = const AsyncLoading();
        state = await AsyncValue.guard(() async {
            final repo = ref.read(taskRepositoryProvider);
            await repo.createTask(request);
            if (!ref.mounted) return state.requireValue;
            return repo.getTasks();
        });
    }

    Future<void> deleteTask(String id) async {
        state = const AsyncLoading();
        state = await AsyncValue.guard(() async {
            final repo = ref.read(taskRepositoryProvider);
            await repo.deleteTask(id);
            if (!ref.mounted) return state.requireValue;
            return repo.getTasks();
        });
    }
}
```

#### Functional (Read-Only / Computed)

```dart
@riverpod
List<Task> filteredTasks(Ref ref) {
    final tasks = ref.watch(taskListProvider).valueOrNull ?? [];
    final filter = ref.watch(taskFilterProvider);
    return tasks.where((t) => filter.matches(t)).toList();
}

@riverpod
Future<Task> taskDetail(Ref ref, String id) async {
    return ref.watch(taskRepositoryProvider).getById(id);
}

@riverpod
Stream<List<Task>> taskStream(Ref ref) {
    return ref.watch(taskRepositoryProvider).watchAll();
}
```

**Family providers:** extra params beyond `Ref` create family — each unique arg combo gets independent instance. Code generation supports any number of params (named, optional, defaults). All params must implement `==`/`hashCode`.

```dart
@riverpod
Future<Task> taskDetail(Ref ref, String id) async {
    return ref.watch(taskRepositoryProvider).getById(id);
}

@riverpod
Future<List<Task>> projectTasks(Ref ref, String projectId, {TaskStatus? status}) async {
    return ref.watch(taskRepositoryProvider).getByProject(projectId, status: status);
}
```

#### `ref.watch` vs `ref.read`

```dart
// ref.watch — subscribes, use inside build()
final tasks = ref.watch(taskListProvider);

// ref.read — one-time, use in event handlers / notifier actions
Future<void> onSubmit() async {
    await ref.read(taskListProvider.notifier).addTask(request);
}

// ❌ Never ref.watch inside async/event handlers
```

#### `Ref.mounted` — Mandatory After Awaits

```dart
Future<void> updateTask(Task task) async {
    final repo = ref.read(taskRepositoryProvider);
    await repo.update(task);
    if (!ref.mounted) return; // REQUIRED
    state = await AsyncValue.guard(() => repo.getTasks());
}
```

#### Auto-Dispose and `keepAlive`

```dart
// autoDispose = DEFAULT with @riverpod (disposed when no consumers)
@riverpod
Future<Task> taskDetail(Ref ref, String id) async {
    return ref.watch(taskRepositoryProvider).getById(id);
}

// keepAlive for app-wide, long-lived state
@Riverpod(keepAlive: true)
class AuthState extends _$AuthState {
    @override
    Future<User?> build() async {
        return ref.watch(authRepositoryProvider).getCurrentUser();
    }
}

// Repositories = keepAlive (hold connection state)
@Riverpod(keepAlive: true)
TaskRepository taskRepository(Ref ref) {
    return TaskRepositoryImpl(apiClient: ref.watch(apiClientProvider));
}
```

#### Repository Interface Pattern

```dart
// Abstract interface (contract)
abstract class TaskRepository {
    Future<List<Task>> getTasks();
    Future<Task> getById(String id);
    Future<void> createTask(CreateTaskRequest request);
    Future<void> deleteTask(String id);
}
```

```dart
// Production adapter
class TaskRepositoryImpl implements TaskRepository {
    const TaskRepositoryImpl({required this.apiClient});
    final ApiClient apiClient;

    @override
    Future<List<Task>> getTasks() async {
        final response = await apiClient.get('/tasks');
        return (response.data as List)
            .map((e) => Task.fromJson(e as Map<String, dynamic>))
            .toList();
    }

    @override
    Future<Task> getById(String id) async {
        final response = await apiClient.get('/tasks/$id');
        return Task.fromJson(response.data as Map<String, dynamic>);
    }

    @override
    Future<void> createTask(CreateTaskRequest request) async {
        await apiClient.post('/tasks', data: request.toJson());
    }

    @override
    Future<void> deleteTask(String id) async {
        await apiClient.delete('/tasks/$id');
    }
}
```

```dart
// Test adapter
class MockTaskRepository implements TaskRepository {
    final List<Task> _tasks = [];

    @override
    Future<List<Task>> getTasks() async => List.unmodifiable(_tasks);

    @override
    Future<Task> getById(String id) async =>
        _tasks.firstWhere((t) => t.id == id);

    @override
    Future<void> createTask(CreateTaskRequest request) async {
        _tasks.add(Task(id: 'mock-id', title: request.title));
    }

    @override
    Future<void> deleteTask(String id) async {
        _tasks.removeWhere((t) => t.id == id);
    }
}
```

```dart
// Wiring
@Riverpod(keepAlive: true)
TaskRepository taskRepository(Ref ref) {
    return TaskRepositoryImpl(apiClient: ref.watch(apiClientProvider));
}
// Tests: taskRepositoryProvider.overrideWith((_) => MockTaskRepository())
```

#### ConsumerWidget vs ConsumerStatefulWidget

```dart
// Prefer ConsumerWidget — stateless, simpler
class TaskListView extends ConsumerWidget {
    const TaskListView({super.key});

    @override
    Widget build(BuildContext context, WidgetRef ref) {
        final asyncTasks = ref.watch(taskListProvider);
        return asyncTasks.when(
            data: (tasks) => TaskListBody(tasks: tasks),
            loading: () => const LoadingIndicator(),
            error: (e, _) => ErrorView(error: e),
        );
    }
}
// ConsumerStatefulWidget only when local widget state + riverpod needed
```

### Runtime Behaviors

**Auto-retry:** default exponential backoff on provider failure. Disable per-provider with `@Riverpod(retry: null)` or globally via `ProviderScope(retry: (_, __) => null)`. Leave enabled for transient IO. Disable for non-idempotent writes.

**Pause/Resume:** auto-pauses listeners when widget not visible. Automatic, no action needed.

**ProviderException:** failed provider reads throw `ProviderException` wrapping original error.

```dart
try {
    final value = container.read(myProvider);
} on ProviderException catch (e) {
    // e.exception = original, e.provider = which failed
}
```

**State change detection:** uses `==` (not `identical`). `freezed` works out of box. Custom models must implement `==`/`hashCode`.

### Async Patterns

1. **Handle all three `AsyncValue` states:**
   ```dart
   asyncValue.when(
       data: (data) => DataWidget(data: data),
       loading: () => const CircularProgressIndicator(),
       error: (err, stack) => ErrorText(err.toString()),
   );
   ```

2. **`hasError` for inline error + stale data visible:**
   ```dart
   if (asyncTasks.hasError) { /* snackbar/inline error */ }
   final tasks = asyncTasks.valueOrNull ?? const [];
   ```

3. **Safe accessors:** `valueOrNull` (safe). `requireValue` only when state confirmed loaded.
4. **`AsyncValue.guard`** inside notifier actions for exception wrapping.
5. **`StreamProvider`** for real-time — never poll with Timer.
6. **`ref.mounted` after `await`** — always.
7. **`ref.invalidate`** / `ref.invalidateSelf()` for re-fetch. Use `invalidateSelf()` when no optimistic UI needed; manual state assignment for instant visual update.

### Error Handling

```dart
// Typed exception hierarchy
sealed class AppException implements Exception {
    const AppException(this.message);
    final String message;
    @override
    String toString() => message;
}

class NetworkException extends AppException {
    const NetworkException(super.message, {this.statusCode});
    final int? statusCode;
}

class ValidationException extends AppException {
    const ValidationException(super.message, {required this.field});
    final String field;
}

class NotFoundException extends AppException {
    const NotFoundException(super.message);
}
```

```dart
// Map infra -> domain exceptions in notifier
Future<void> addTask(CreateTaskRequest request) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
        try {
            final repo = ref.read(taskRepositoryProvider);
            await repo.createTask(request);
            if (!ref.mounted) return state.requireValue;
            return repo.getTasks();
        } on DioException catch (e) {
            throw NetworkException(
                'Failed to create task: ${e.message}',
                statusCode: e.response?.statusCode,
            );
        }
    });
}
```

```dart
// Typed error handling in UI
asyncTasks.when(
    data: (tasks) => TaskListBody(tasks: tasks),
    loading: () => const LoadingIndicator(),
    error: (error, _) => switch (error) {
        NetworkException() => ErrorView(message: 'Network error: ${error.message}'),
        NotFoundException() => const ErrorView(message: 'Not found'),
        _ => ErrorView(message: 'Unexpected error: $error'),
    },
);
```

Rules: all exceptions extend sealed `AppException`. Infra exceptions caught + re-thrown as domain. Never expose infra types to UI. Exhaustive switch.

### Navigation with `go_router`

```dart
@riverpod
GoRouter appRouter(Ref ref) {
    return GoRouter(
        initialLocation: '/tasks',
        routes: [
            GoRoute(path: '/tasks', builder: (_, __) => const TaskListView()),
            GoRoute(
                path: '/tasks/:id',
                builder: (_, state) => TaskDetailView(
                    id: state.pathParameters['id']!,
                ),
            ),
        ],
    );
}

context.go('/tasks/$taskId');
context.push('/tasks/new');
```

### Dart Language Idioms

1. **Null safety:** `?.`, `??`, `??=` idiomatically. `cache ??= await compute();`
2. **`late`** only for fields initialized before first use that cannot be `final`. Prefer `final`.
3. **Extension methods** for behavior on types you don't own.
4. **`switch` expressions (Dart 3+)** for exhaustive pattern matching — compiler error on missing case.
5. **Avoid `dynamic`** — Dart equivalent of TS `any`.

### Testing

> Naming/pyramid: GEMINI.md § Testing Strategy. Flutter/Riverpod 3 patterns below.

#### Unit Test with `ProviderContainer.test`

```dart
test('addTask updates state', () async {
    final container = ProviderContainer.test(overrides: [
        taskRepositoryProvider.overrideWith((_) => MockTaskRepository()),
    ]);

    await container.read(taskListProvider.notifier).addTask(request);
    expect(container.read(taskListProvider).value, hasLength(1));
});
```

#### `overrideWithBuild` for initial state seeding

```dart
test('deleteTask removes item', () async {
    final container = ProviderContainer.test(overrides: [
        taskRepositoryProvider.overrideWith((_) => MockTaskRepository()),
        taskListProvider.overrideWithBuild((ref, notifier) {
            return Future.value([mockTask1, mockTask2]);
        }),
    ]);

    await container.read(taskListProvider.notifier).deleteTask(mockTask1.id);
    expect(container.read(taskListProvider).value, hasLength(1));
});
```

#### Widget Tests

```dart
testWidgets('shows task list', (tester) async {
    await tester.pumpWidget(ProviderScope(
        overrides: [
            taskRepositoryProvider.overrideWith((_) => MockTaskRepository()),
        ],
        child: const MaterialApp(home: TaskListView()),
    ));
    expect(find.byType(TaskCard), findsWidgets);
});
```

#### `mockito` with `@GenerateNiceMocks`

```dart
@GenerateNiceMocks([MockSpec<TaskRepository>()])
library;

import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'task_notifier_test.mocks.dart';

void main() {
    // tests use MockTaskRepository()
}
```

### Anti-Patterns — NEVER DO THIS

| Anti-Pattern | Correct |
|---|---|
| `StateProvider` | `@riverpod` class with Notifier |
| `StateNotifierProvider` | `@riverpod` class with Notifier |
| `ChangeNotifierProvider` | `@riverpod` class with Notifier |
| `import 'package:riverpod/legacy.dart'` | Never import legacy |
| Typed ref (`TaskDetailRef ref`) | `Ref ref` — single type in Riverpod 3 |
| `ref.watch` in async/handler | `ref.read` for one-shot |
| State/ref access after await without mounted check | Always check `ref.mounted` |
| `ProviderContainer()` + `addTearDown` | `ProviderContainer.test(...)` |
| `Timer.periodic` for polling | `StreamProvider` |
| `keepAlive: false` | Omit — default |
| Manual providers without `@riverpod` | Always codegen |
| Catching raw exceptions | Catch `ProviderException` |
| `overrideWith` for initial state seeding | `overrideWithBuild` |

### Linting and Formatting

| Tool | Purpose |
|---|---|
| `dart format` | Canonical formatting |
| `flutter analyze` | Static analysis + lint |
| `riverpod_lint` | Riverpod-specific rules |
| `dart pub deps` | Dependency audit |
| `dart run build_runner build` | Generate provider code |

```yaml
# analysis_options.yaml (mandatory)
analyzer:
  language:
    strict-casts: true
    strict-raw-types: true
  errors:
    invalid_assignment: error
  plugins:
    - riverpod_lint
linter:
  rules:
    - prefer_const_constructors
    - prefer_const_declarations
    - avoid_dynamic_calls
    - avoid_print
    - use_super_parameters
```

After provider changes:
```bash
dart run build_runner build --delete-conflicting-outputs
flutter analyze
dart format .
```

Dev watch mode:
```bash
dart run build_runner watch --delete-conflicting-outputs
```

### Related
- Code Idioms and Conventions GEMINI.md § Code Idioms and Conventions
- Project Structure — Flutter Mobile @.gemini/skills/project-structure-flutter/SKILL.md
- Architectural Patterns GEMINI.md § Architectural Patterns
- Testing Strategy GEMINI.md § Testing Strategy
- Error Handling Principles GEMINI.md § Error Handling Principles
- Dependency Management Principles @.gemini/skills/dependency-management-principles/SKILL.md
