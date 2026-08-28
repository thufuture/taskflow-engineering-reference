# TaskFlow API Reference

TaskFlow is a small but fully executable task-management API used as a reference project for engineer onboarding, repository scanning, document classification, and source-grounded chatbot answers.

The service exposes CRUD operations for shared tasks. It is intentionally implemented as a single FastAPI process with SQLAlchemy and SQLite so a new engineer can understand the complete request path in one working session.

## What is implemented

- Create, list, read, partially update, and delete tasks.
- Filter task lists by completion status and priority.
- Validate request and response payloads with Pydantic.
- Manage one SQLAlchemy session per HTTP request.
- Create the SQLite schema during application startup.
- Run isolated API tests against an in-memory database.
- Expose `/health`, OpenAPI JSON, and Swagger UI.

## Explicit non-goals

The current reference implementation does **not** provide authentication, per-user ownership, Alembic migrations, pagination, rate limiting, or a production deployment topology. Do not expose it directly to the public Internet. These limitations are documented so engineers and AI assistants do not infer capabilities that are not present in code.

## Technology baseline

| Area | Choice | Source of truth |
|---|---|---|
| Runtime | Python 3.11+ | `requirements.txt`, type syntax |
| HTTP framework | FastAPI | `app/main.py`, `app/routes.py` |
| Persistence | SQLAlchemy 2.x ORM | `app/database.py`, `app/models.py` |
| Local database | SQLite | `DATABASE_URL` |
| Validation | Pydantic | `app/schemas.py` |
| Tests | pytest + FastAPI TestClient | `tests/test_todos.py` |

## Quick start

```bash
python -m venv .venv

# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS/Linux
source .venv/bin/activate

python -m pip install -r requirements.txt
uvicorn app.main:app --reload
```

Verify the service:

```bash
curl http://127.0.0.1:8000/health
# {"status":"ok"}
```

Open `http://127.0.0.1:8000/docs` for Swagger UI. See [local setup](docs/setup/local-setup.md) for Windows, macOS/Linux, verification, and troubleshooting details.

## First API workflow

```bash
curl -X POST http://127.0.0.1:8000/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Review onboarding docs","description":"Check architecture and runbook","priority":"high"}'

curl "http://127.0.0.1:8000/todos?is_done=false&priority=high"

curl -X PATCH http://127.0.0.1:8000/todos/1 \
  -H "Content-Type: application/json" \
  -d '{"is_done":true}'
```

The canonical endpoint behavior, validation rules, status codes, and examples live in [Todo API contract](docs/api/todo-api.md).

## Repository map

```text
taskflow-api-reference/
├── app/
│   ├── main.py                 # FastAPI application and startup hook
│   ├── database.py             # Engine, session factory, Base, get_db
│   ├── models.py               # Todo ORM model
│   ├── schemas.py              # Create, update, and response contracts
│   └── routes.py               # HTTP handlers and database operations
├── tests/test_todos.py         # Executable API behavior examples
├── docs/
│   ├── product/project-overview.md
│   ├── architecture/system-design.md
│   ├── architecture/data-model.md
│   ├── api/todo-api.md
│   ├── setup/local-setup.md
│   ├── access/security-guide.md
│   ├── codebase-guide.md
│   ├── operations/runbook.md
│   ├── testing/testing-strategy.md
│   ├── onboarding/first-week.md
│   └── adr/0001-single-service.md
├── .env.example
├── .gitignore
├── CONTRIBUTING.md
└── requirements.txt
```

## Documentation index by onboarding need

| If you need to… | Read this | Expected outcome |
|---|---|---|
| Understand the product and scope | [Project overview](docs/product/project-overview.md) | Explain users, use cases, non-goals, and success criteria |
| Trace a request end to end | [System design](docs/architecture/system-design.md) | Follow HTTP → validation → ORM → SQLite → response |
| Understand fields and invariants | [Data model](docs/architecture/data-model.md) | Change Todo fields without breaking contracts |
| Run the service locally | [Local setup](docs/setup/local-setup.md) | Start the API and pass smoke checks |
| Integrate with the API | [API contract](docs/api/todo-api.md) | Send valid requests and handle errors |
| Modify the implementation | [Codebase guide](docs/codebase-guide.md) | Identify every file and required test impact |
| Assess access and security | [Security guide](docs/access/security-guide.md) | Understand current trust boundary and production blockers |
| Diagnose an incident | [Operations runbook](docs/operations/runbook.md) | Triage health, port, database, and validation failures |
| Add or review tests | [Testing strategy](docs/testing/testing-strategy.md) | Preserve isolated deterministic test behavior |
| Complete onboarding | [First-week checklist](docs/onboarding/first-week.md) | Produce a verified first change |
| Ground chatbot answers | [Chatbot grounding guide](docs/onboarding/chatbot-grounding.md) | Cite implemented behavior and reject unsupported assumptions |
| Understand architecture decisions | [ADR-0001](docs/adr/0001-single-service.md) | Explain why this is a modular monolith reference |

## Definition of done

A change is complete only when:

1. Runtime behavior and API contract agree.
2. Request/response schema changes are covered by tests.
3. `pytest -q` passes from a clean environment.
4. No `.env`, database, log, virtualenv, cache, or dependency directory is committed.
5. Relevant documentation and examples are updated.
6. The pull request describes risk, validation evidence, and rollback approach.

## Useful commands

```bash
pytest -q
uvicorn app.main:app --reload
curl http://127.0.0.1:8000/health
```

## Status

This repository is an onboarding/reference service. Its value is accuracy, traceability, and safe learning—not feature breadth. For contribution rules and review expectations, see [CONTRIBUTING.md](CONTRIBUTING.md).
