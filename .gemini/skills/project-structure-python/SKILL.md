---
paths:
  - "**/*.py"
---

## Python Backend Layout

Vertical slice — features are packages, not technical layers.

```
apps/
  backend/
    pyproject.toml                  # Single config: ruff, mypy, pytest, bandit, packaging
    src/
      yourapp/                      # src-layout (PEP 517)
        main.py                     # Entry point: wire deps, start server
        platform/                   # Foundation
          database.py               # DB engine/sessions
          server.py                 # ASGI setup (FastAPI)
          logger.py                 # structlog config
          config.py                 # pydantic-settings BaseSettings
        features/
          task/
            __init__.py
            service.py              # Public API
            router.py               # FastAPI APIRouter
            schemas.py              # Pydantic req/res (boundary only)
            test_router.py          # Component tests
            logic.py                # Pure domain fns
            models.py               # Domain dataclasses
            errors.py               # Feature exceptions
            test_logic.py           # Unit tests
            storage.py              # TaskStorage Protocol
            storage_pg.py           # PostgreSQL impl
            storage_mock.py         # InMemory impl (test double)
            test_storage_pg.py      # Integration (testcontainers)
          order/
            service.py
            router.py
            schemas.py
            logic.py
            models.py
            storage.py
            storage_pg.py
            storage_mock.py
    tests/
      e2e/
        test_create_task_api.e2e.py
```

Key: `src/` layout preferred. `pyproject.toml` single config (no `setup.cfg`/`.flake8`/`mypy.ini`). Feature `__init__.py` empty or public API only. `platform/` = shared infra. Tests co-located except E2E. `storage_mock.py` = production-quality in-memory impl, not `unittest.Mock`.

### Dependency Wiring

```python
# src/yourapp/main.py
from yourapp.platform.database import create_engine
from yourapp.platform.server import create_app
from yourapp.features.task.storage_pg import PostgresTaskStorage
from yourapp.features.task.service import TaskService
from yourapp.features.task.router import build_router as build_task_router

def create_application() -> FastAPI:
    engine = create_engine()
    task_storage = PostgresTaskStorage(engine=engine)
    task_service = TaskService(storage=task_storage)

    app = create_app()
    app.include_router(build_task_router(service=task_service))
    return app

app = create_application()
```

### Configuration

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    debug: bool = False
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

def get_settings() -> Settings:
    return Settings()
```

### pyproject.toml Baseline

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "yourapp"
requires-python = ">=3.11"

[tool.hatch.build.targets.wheel]
packages = ["src/yourapp"]

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "S", "B", "ANN"]

[tool.mypy]
strict = true
python_version = "3.11"

[tool.pytest.ini_options]
testpaths = ["src"]
asyncio_mode = "auto"

[tool.bandit]
skips = ["B101"]
```

### Related
- Project Structure GEMINI.md § Project Structure
- Python Idioms and Patterns @.gemini/skills/python-idioms/SKILL.md
