# Local development setup

## Objective

At the end of this guide, a new engineer can run TaskFlow locally, execute the test suite, make one API request, and identify the local database file without relying on undocumented machine state.

## Prerequisites

- Python 3.11 or newer.
- Git 2.40 or newer.
- A terminal with permission to create a virtual environment.
- TCP port `8000`, or another available port.

```bash
python --version
git --version
```

## 1. Clone and enter the repository

```bash
git clone <repository-url> taskflow-api-reference
cd taskflow-api-reference
```

## 2. Create an isolated environment

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

If activation is blocked for the current process:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source .venv/bin/activate
```

Verify the interpreter path contains `.venv`:

```bash
python -c "import sys; print(sys.executable)"
```

## 3. Install dependencies

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -c "import fastapi, sqlalchemy, pydantic; print('dependencies ok')"
```

## 4. Configure runtime

The current code reads only `DATABASE_URL`. Do not add unrelated placeholder secrets to `.env.example`; configuration examples should reflect variables the application actually consumes.

Windows PowerShell:

```powershell
Copy-Item .env.example .env
$env:DATABASE_URL = "sqlite:///./todo.db"
```

macOS/Linux:

```bash
cp .env.example .env
export DATABASE_URL="sqlite:///./todo.db"
```

The app does not load `.env` automatically because `python-dotenv` is not installed. Export variables in the shell or use the default URL. Never commit `.env`.

## 5. Run tests first

```bash
pytest -q
```

Tests use an in-memory SQLite database via `StaticPool`; they do not modify local `todo.db`.

## 6. Start the API

```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Startup sequence:

1. Uvicorn imports `app.main:app`.
2. FastAPI runs the startup handler.
3. `init_db()` calls `Base.metadata.create_all()`.
4. SQLite creates `todo.db` and `todos` if absent.

`create_all()` does not migrate existing tables. A model-column change needs a migration strategy; deleting a database is acceptable only for disposable local data.

## 7. Smoke test

```bash
curl http://127.0.0.1:8000/health
curl http://127.0.0.1:8000/openapi.json
curl http://127.0.0.1:8000/todos
```

| Check | Expected |
|---|---|
| `/health` | HTTP 200, `{"status":"ok"}` |
| `/openapi.json` | HTTP 200 OpenAPI JSON |
| `/todos` | HTTP 200 JSON array |

## 8. Exercise write flow

```bash
curl -X POST http://127.0.0.1:8000/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Complete local setup","priority":"high"}'

curl -X PATCH http://127.0.0.1:8000/todos/1 \
  -H "Content-Type: application/json" \
  -d '{"is_done":true}'
```

## Troubleshooting

| Symptom | Diagnosis | Resolution |
|---|---|---|
| `python` not recognized | Python absent from PATH | Install Python 3.11+; reopen terminal |
| `ModuleNotFoundError: app` | Wrong working directory | Run from directory containing `app/` |
| `ModuleNotFoundError: fastapi` | Wrong interpreter/dependencies | Activate `.venv`; reinstall requirements |
| Port 8000 in use | Another process owns port | Start with `--port 8001`; update URLs |
| Database is read-only | Clone directory not writable | Move repository to writable location |
| New field missing in DB | `create_all()` is not migration | Recreate disposable DB or add migration |
| `.env` ignored at runtime | No dotenv loader | Export variables before Uvicorn starts |
| Tests modify local DB | Dependency override broken | Restore in-memory engine and `get_db` override |

## Completion checklist

- [ ] Python executable is inside `.venv`.
- [ ] Dependencies import successfully.
- [ ] `pytest -q` passes.
- [ ] `/health` returns 200.
- [ ] A task can be created and read.
- [ ] Secret, database, log, cache, and dependency files remain ignored.
