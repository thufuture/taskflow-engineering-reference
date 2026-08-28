# Codebase Guide

## Start here

Read files in this order to understand the implementation quickly:

1. `README.md` for scope and commands.
2. `app/main.py` for application composition.
3. `app/schemas.py` for the public data contract.
4. `app/routes.py` for endpoint behavior.
5. `app/models.py` for persistence.
6. `app/database.py` for engine and session lifecycle.
7. `tests/test_todos.py` for executable behavior examples.

## Source map

```text
app/
├── __init__.py
├── main.py       # FastAPI app, middleware, router, health, startup
├── routes.py     # Todo CRUD and filters
├── schemas.py    # Request/response models and priority enum
├── models.py     # SQLAlchemy Todo table
└── database.py   # DATABASE_URL, engine, session factory, dependency
tests/
└── test_todos.py # In-memory API tests
```

## Application composition

`app/main.py` owns framework wiring. The exported `app` object is what Uvicorn imports with `app.main:app`. The module includes the todo router and exposes `/health`. Startup initializes missing tables through `init_db`.

If a new domain router is added, construct it in its own module and include it here. Do not place domain queries directly in `main.py`.

## Contract layer

`app/schemas.py` is the first file to inspect when an API payload fails validation. It separates create, partial update, and response shapes. That distinction prevents clients from supplying generated fields such as IDs or timestamps.

When adding a field, decide independently whether it is:

- accepted during create;
- mutable during PATCH;
- returned to clients;
- nullable, optional, or defaulted.

Those are separate product decisions, not one mechanical model change.

## Route layer

`app/routes.py` owns HTTP orchestration:

- extract path, query, and body values;
- obtain a request-scoped database session;
- load or query entities;
- map absent rows to `404`;
- commit mutations and refresh response entities;
- declare response models and status codes.

Keep route behavior readable. Introduce a service function only when rules are reused, the transaction spans several mutations, or the handler becomes difficult to test as a unit.

## Persistence layer

`app/models.py` declares the table. Python defaults, database nullability, indexes, and timestamps are persisted concerns. `app/database.py` constructs the SQLAlchemy engine from `DATABASE_URL`, creates the session factory, and closes sessions after each request.

Never import and retain a live session at module scope. Never point automated tests at the developer's file database.

## Common change recipes

### Add a field

1. Define semantics in the product/API docs.
2. Update the SQLAlchemy model.
3. Update relevant Pydantic schemas.
4. Add migration planning; `create_all` is insufficient for existing production data.
5. Update create/read/update tests.
6. Update API examples and data-model documentation.

### Add a filter

1. Add a typed query parameter to the list handler.
2. Apply the predicate only when the parameter is present.
3. Test matching, non-matching, combined, and invalid values.
4. Document ordering; if pagination is introduced, specify its interaction with that order.

### Add an endpoint

1. Define method, path, authorization expectation, status codes, and schemas.
2. Implement the smallest route behavior.
3. Cover success, invalid input, unknown resources, and state conflicts.
4. Update the API contract and OpenAPI-facing metadata.

## Review checklist

- Does validation happen before mutation?
- Are omitted and null values handled intentionally?
- Can the operation leave a partially committed state?
- Is the status code part of the documented contract?
- Does an unknown resource produce a consistent `404`?
- Are tests isolated from local files and environment?
- Does the change introduce a security or compatibility requirement?
- Are source comments explaining “why” rather than restating code?

## Dependency direction

Preferred direction:

```text
main -> routes -> schemas/models/database
tests -> public app behavior and controlled database fixtures
```

Avoid importing route modules into models or database configuration. Cyclic imports make startup behavior fragile and blur ownership.

## Debugging sequence

For an unexpected API result:

1. reproduce it with a minimal curl request;
2. inspect the response status and validation detail;
3. confirm the app is using the expected `DATABASE_URL`;
4. trace the matching handler in `app/routes.py`;
5. compare the request with `app/schemas.py`;
6. inspect the persisted model and transaction boundary;
7. add a failing regression test before fixing the issue.
