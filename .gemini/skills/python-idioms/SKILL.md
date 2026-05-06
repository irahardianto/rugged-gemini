---
paths:
  - "**/*.py"
---

## Python Idioms and Patterns

Python rewards explicitness and readability. Follow Zen of Python. If it reads like plain English, it's idiomatic.

> Scope: Python coding idioms. Layout: `@.gemini/skills/project-structure-python/SKILL.md`. Tests: GEMINI.md § Testing Strategy. Logging: `@.gemini/skills/logging-and-observability-principles/SKILL.md`.

### Type Hints — Non-Negotiable

Always annotate function signatures and public APIs. Use `from __future__ import annotations`.

```python
from __future__ import annotations
from collections.abc import Sequence

def calculate_discount(items: Sequence[Item], coupon: Coupon) -> float: ...
```

1. **`X | None` over `Optional[X]`** (3.10+):
   ```python
   def find_user(user_id: str) -> User | None: ...
   ```

2. **`TypeAlias` and `TypeVar`:**
   ```python
   from typing import TypeVar, TypeAlias
   T = TypeVar("T")
   UserId: TypeAlias = str
   ```

3. **`Protocol` for structural interfaces** (duck-typing):
   ```python
   from typing import Protocol

   class TaskStorage(Protocol):
       def get_by_id(self, task_id: str) -> Task: ...
       def save(self, task: Task) -> None: ...
   ```

4. **`TypedDict` for structured boundary dicts:**
   ```python
   from typing import TypedDict

   class CreateTaskRequest(TypedDict):
       title: str
       priority: Literal["low", "medium", "high"]
   ```

### Error Handling

> General: GEMINI.md § Error Handling Principles. Python-specific below.

1. **Specific exceptions over `except Exception`:**
   ```python
   try:
       task = storage.get_by_id(task_id)
   except TaskNotFoundError:
       raise HTTPException(status_code=404, detail="Task not found")
   ```

2. **Domain exception hierarchies:**
   ```python
   class FathError(Exception):
       """Base exception for all domain errors."""

   class NotFoundError(FathError):
       def __init__(self, resource: str, resource_id: str) -> None:
           self.resource = resource
           self.resource_id = resource_id
           super().__init__(f"{resource} '{resource_id}' not found")

   class ValidationError(FathError):
       def __init__(self, field: str, message: str) -> None:
           self.field = field
           self.message = message
           super().__init__(f"Validation failed on '{field}': {message}")
   ```

3. **Never silence exceptions** — if caught and not re-raised, log explicitly:
   ```python
   try:
       notify_user(user_id)
   except NotificationError:
       logger.warning("notification_failed", user_id=user_id, exc_info=True)
   ```

4. **`contextlib.suppress`** only for truly expected, inconsequential exceptions:
   ```python
   with suppress(FileNotFoundError):
       cache_path.unlink()  # cleanup, not business logic
   ```

### Dataclasses and Pydantic

1. **`dataclasses` for internal domain models:**
   ```python
   @dataclass(frozen=True)
   class Task:
       id: str
       title: str
       priority: str
       tags: tuple[str, ...] = field(default_factory=tuple)
   ```

2. **Pydantic `BaseModel` for boundary data (API, config):**
   ```python
   from pydantic import BaseModel, Field

   class CreateTaskRequest(BaseModel):
       title: str = Field(min_length=1, max_length=200)
       priority: Literal["low", "medium", "high"] = "medium"
       due_date: datetime | None = None
       model_config = ConfigDict(frozen=True)
   ```

3. **Keep separate:** `models.py` = dataclasses (domain), `schemas.py` = Pydantic (API boundary).

### Interfaces and DI

1. **Define Protocol where used:**
   ```python
   # task/storage.py ← consumer feature
   class TaskStorage(Protocol):
       def get_by_id(self, task_id: str) -> Task: ...
       def save(self, task: Task) -> None: ...
       def delete(self, task_id: str) -> None: ...
   ```

2. **Inject via `__init__`:**
   ```python
   class TaskService:
       def __init__(self, storage: TaskStorage) -> None:
           self._storage = storage
   ```

3. **Wire in entry point:**
   ```python
   storage = PostgresTaskStorage(db=database)
   service = TaskService(storage=storage)
   router.include_router(build_task_router(service))
   ```

### Async / Await

> General: GEMINI.md § Concurrency and Threading Mandate. Python-specific below.

1. **One async paradigm, stay consistent.**
2. **Never blocking I/O in async fn:** use `aiofiles` or executor.
   ```python
   async with aiofiles.open(path) as f:
       return await f.read()
   ```

3. **`asyncio.gather` for concurrent ops:**
   ```python
   user, tasks = await asyncio.gather(get_user(user_id), get_tasks(user_id))
   ```

4. **`asyncio.TaskGroup` (3.11+) for structured concurrency:**
   ```python
   async with asyncio.TaskGroup() as tg:
       user_task = tg.create_task(get_user(user_id))
       tasks_task = tg.create_task(get_tasks(user_id))
   user = user_task.result()
   tasks = tasks_task.result()
   ```

### Naming — PEP 8, No Exceptions

| Construct | Convention | Example |
|---|---|---|
| Module/Package | `snake_case` | `task_service.py` |
| Class | `PascalCase` | `TaskService` |
| Function/Method | `snake_case` | `get_by_id` |
| Private | `_snake_case` | `_validate_title` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_TITLE_LENGTH = 200` |
| Type alias | `PascalCase` | `UserId = str` |
| Protocol | `PascalCase` | `TaskStorage` |

- No single-letter names outside comprehensions/math.
- Avoid `data`, `info`, `obj`, `result` standalone — use domain concepts.
- Booleans read as yes/no: `is_active`, `has_permission`, `can_edit()`.

### Idiomatic Patterns

1. **Context managers** for resource cleanup — `with` over manual `close()`.
2. **Generator expressions** for lazy evaluation: `(task.id for task in tasks if task.is_active)`.
3. **`dataclasses.replace()`** for immutable updates: `replace(task, title="New Title")`.
4. **`functools.cache`/`lru_cache`** for pure fn memoization.
5. **`__slots__`** on hot-path, frequently instantiated classes.
6. **`enum.StrEnum`** (3.11+) for domain constants:
   ```python
   class Priority(StrEnum):
       LOW    = "low"
       MEDIUM = "medium"
       HIGH   = "high"
   ```

### Testing

> Naming/pyramid: GEMINI.md § Testing Strategy. Python-specific below.

1. **`pytest` only** — never `unittest.TestCase`:
   ```python
   def test_calculate_discount_returns_zero_for_no_items() -> None:
       result = calculate_discount(items=[], coupon=Coupon(code="SAVE10"))
       assert result == 0.0
   ```

2. **`@pytest.mark.parametrize`:**
   ```python
   @pytest.mark.parametrize("priority,expected_score", [
       ("low", 1), ("medium", 5), ("high", 10),
   ])
   def test_priority_score(priority: str, expected_score: int) -> None:
       assert priority_score(priority) == expected_score
   ```

3. **`pytest-mock` (`mocker` fixture):**
   ```python
   def test_task_service_creates_task(mocker: MockerFixture) -> None:
       mock_storage = mocker.create_autospec(TaskStorage, instance=True)
       service = TaskService(storage=mock_storage)
       service.create(title="Test", priority="high")
       mock_storage.save.assert_called_once()
   ```

4. **In-memory test adapter:**
   ```python
   class InMemoryTaskStorage:
       def __init__(self) -> None:
           self._store: dict[str, Task] = {}

       def get_by_id(self, task_id: str) -> Task:
           if task_id not in self._store:
               raise NotFoundError("Task", task_id)
           return self._store[task_id]

       def save(self, task: Task) -> None:
           self._store[task.id] = task

       def delete(self, task_id: str) -> None:
           self._store.pop(task_id, None)
   ```

5. **`pytest-asyncio`:**
   ```python
   @pytest.mark.asyncio
   async def test_async_create_task() -> None:
       service = TaskService(storage=InMemoryTaskStorage())
       task = await service.create(title="Async Task", priority="low")
       assert task.title == "Async Task"
   ```

6. **Fixtures for reusable setup** — no repeated Arrange blocks.

### Formatting and Static Analysis

Must pass zero warnings/errors. See GEMINI.md § Code Completion Mandate.

| Tool | Purpose | Command |
|---|---|---|
| `ruff format` | Formatting | `ruff format .` |
| `ruff check` | Lint | `ruff check . --fix` |
| `mypy` | Type checking | `mypy src/ --strict` |
| `bandit` | Security scan | `bandit -r src/ -c pyproject.toml` |
| `pip-audit` | CVE scanning | `pip-audit` |

```toml
[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "S", "B", "ANN"]
ignore = []

[tool.mypy]
strict = true
python_version = "3.11"

[tool.pytest.ini_options]
asyncio_mode = "auto"
```

> Never `print()` in production. Use `logging`/`structlog` for structured JSON. See `@.gemini/skills/logging-and-observability-principles/SKILL.md`.

### Related
- Code Idioms and Conventions GEMINI.md § Code Idioms and Conventions
- Project Structure — Python Backend @.gemini/skills/project-structure-python/SKILL.md
- Testing Strategy GEMINI.md § Testing Strategy
- Error Handling Principles GEMINI.md § Error Handling Principles
- Concurrency and Threading Mandate GEMINI.md § Concurrency and Threading Mandate
- Logging and Observability Principles @.gemini/skills/logging-and-observability-principles/SKILL.md
- Security Principles GEMINI.md § Security Principles
- Dependency Management Principles @.gemini/skills/dependency-management-principles/SKILL.md
