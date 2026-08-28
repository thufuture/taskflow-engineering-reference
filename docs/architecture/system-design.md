# System Design

## Purpose and scope

TaskFlow is a reference REST API for managing personal or team work items. The current implementation intentionally uses one FastAPI process and one relational database so engineers can understand the complete request path without distributed-system noise.

The service currently supports create, list, read, update, and delete operations for `Todo` records. Authentication, authorization, background jobs, notifications, and multi-tenant isolation are outside the implemented scope.

## System context

```text
API client
   |
   | HTTP + JSON
   v
FastAPI application (app/main.py)
   |
   +-- Todo router (app/routes.py)
   |      |
   |      +-- Pydantic contracts (app/schemas.py)
   |      +-- SQLAlchemy entity (app/models.py)
   |
   +-- Database session (app/database.py)
          |
          v
      SQLite by default
      or another SQLAlchemy URL via DATABASE_URL
```

There are no internal network hops. Route handlers execute database work in the request process.

## Components and responsibilities

| Component | Responsibility | Must not own |
|---|---|---|
| `app/main.py` | Construct FastAPI, configure CORS, include routes, expose health check, initialize tables | Todo business rules |
| `app/routes.py` | HTTP behavior, filtering, CRUD orchestration, status codes | Engine construction or table declaration |
| `app/schemas.py` | Validate request data and serialize responses | Database queries |
| `app/models.py` | Define persisted columns and defaults | HTTP concerns |
| `app/database.py` | Build the engine/session and provide request-scoped sessions | Endpoint-specific queries |
| `tests/test_todos.py` | Verify public behavior using an isolated database | Production data |

This separation is the primary architectural invariant. New code should preserve it unless an ADR records a deliberate change.

## Request lifecycle

### Create a work item

1. The client sends `POST /todos` with JSON.
2. FastAPI selects `create_todo`.
3. `TodoCreate` validates required fields and enum values before the handler runs.
4. The handler creates a SQLAlchemy `Todo`, adds it to the session, commits, and refreshes it.
5. FastAPI serializes the entity through `TodoRead` and returns `201 Created`.
6. Validation failures return `422`; unexpected database failures roll back the active transaction through session cleanup behavior.

### Read a collection

1. The client sends `GET /todos` with optional filters.
2. The handler starts from a `Todo` query and conditionally adds predicates.
3. Results are ordered by `created_at` descending.
4. The complete matching collection is serialized as a list of `TodoRead` values; pagination is not implemented.

### Partially update a work item

1. The client sends `PATCH /todos/{todo_id}`.
2. The handler loads the row or returns `404`.
3. `TodoUpdate` retains only explicitly supplied fields.
4. Those fields are assigned to the entity, then committed and refreshed.
5. Omitted fields remain unchanged. See the API contract for null semantics.

## Data consistency and transactions

- Each request receives a SQLAlchemy session from `get_db`.
- Mutating handlers commit exactly once after applying their changes.
- The refreshed entity is the response source, so generated identifiers and database-side values are visible.
- The current service does not coordinate transactions across external systems.
- Delete is a hard delete; there is no archive or recovery state.

If a future feature needs multiple writes to be atomic, keep them in the same session and commit only after all invariants are satisfied.

## Startup and schema lifecycle

Application startup calls `init_db`, which invokes SQLAlchemy `create_all`. This creates missing tables but is not a migration system: it cannot safely rename columns, backfill data, or perform controlled rollbacks. Any production evolution should introduce Alembic before the first incompatible schema change.

## Concurrency model

Route functions are synchronous and use a synchronous SQLAlchemy session. FastAPI runs synchronous handlers in its worker thread pool. SQLite is suitable for local development and low-write reference use, but concurrent writes may contend on a database-level lock.

Do not solve SQLite contention by adding retries blindly. First decide whether the deployment requires PostgreSQL, multiple workers, or a database-specific concurrency policy.

## Security boundary

The API currently trusts every caller. Wildcard CORS allows browser clients from any origin. This is acceptable only for a local reference environment. Before exposure to an untrusted network, add identity verification, authorization, restrictive CORS, secret management, request limits, and transport security. The complete gate is in [Security Guide](../access/security-guide.md).

## Observability

Implemented signals are intentionally minimal:

- `/health` confirms that the web application can answer a request;
- Uvicorn provides access and exception logs;
- HTTP status codes expose validation and not-found failures.

The health endpoint does not verify database connectivity. There are no metrics, traces, correlation IDs, audit events, or structured domain logs.

## Failure modes

| Failure | Visible behavior | First investigation |
|---|---|---|
| Invalid JSON or field | `422` | Inspect FastAPI validation detail |
| Unknown todo ID | `404` | Confirm ID and database target |
| Database cannot open | startup/request error | Check `DATABASE_URL`, path permissions, and driver |
| SQLite write contention | lock-related server error | Check concurrent writers and transaction duration |
| Port already bound | server fails to start | Stop the conflicting process or select another port |

## Scaling decision points

Move beyond this architecture only when evidence requires it:

- choose PostgreSQL when concurrent writes or operational durability exceed SQLite's target;
- add a service layer when rules are shared by multiple handlers or transactions become non-trivial;
- add background workers only for work that should not block an HTTP response;
- split services only when ownership, scaling, or failure isolation justify the operational cost.

## Architecture invariants

1. Public request and response behavior is defined by Pydantic schemas.
2. Database sessions are request-scoped and never stored globally.
3. Tests do not use the developer's `todo.db`.
4. Schema changes require an explicit migration strategy.
5. Documentation distinguishes implemented behavior from planned behavior.
