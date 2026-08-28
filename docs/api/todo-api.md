# Todo API Contract

## Conventions

- Base URL in local development: `http://127.0.0.1:8000`
- Content type for request bodies: `application/json`
- Error bodies use FastAPI's standard `detail` structure.
- Datetimes are JSON strings produced by Pydantic from Python datetime values.
- IDs are positive database-generated integers.

## Todo representation

```json
{
  "id": 1,
  "title": "Read the architecture guide",
  "description": "Review request flow and persistence boundaries",
  "is_done": false,
  "priority": "high",
  "created_at": "2026-08-28T10:15:00",
  "updated_at": "2026-08-28T10:15:00"
}
```

## Health

### `GET /health`

Confirms that the FastAPI process is responsive.

Success: `200 OK`

```json
{"status":"ok"}
```

This is a liveness signal, not a database readiness check.

## Create a todo

### `POST /todos`

Request:

```json
{
  "title": "Add contract tests",
  "description": "Cover invalid priority and missing IDs",
  "priority": "high"
}
```

Response: `201 Created` with the complete todo representation.

Expected errors:

- `422` when a required field is missing or priority is invalid.

## List todos

### `GET /todos`

The endpoint supports the optional `is_done` boolean and `priority` enum filters implemented in `app/routes.py`.

Typical request:

```bash
curl "http://127.0.0.1:8000/todos?is_done=false&priority=high"
```

Response: `200 OK` and a JSON array. An empty result is `[]`, not `404`.

Results are ordered by `created_at` descending. Pagination is not implemented, so consumers should avoid assuming this endpoint is suitable for an unbounded production dataset.

## Read one todo

### `GET /todos/{todo_id}`

- `200` with the todo when it exists.
- `404` when no matching row exists.

```bash
curl http://127.0.0.1:8000/todos/1
```

## Partially update a todo

### `PATCH /todos/{todo_id}`

Only supplied fields are changed.

```json
{
  "is_done": true,
  "priority": "low"
}
```

- `200` with the updated representation.
- `404` for an unknown ID.
- `422` for an invalid field value.

Omission means “leave unchanged.” Before allowing explicit `null` for a field, verify the Pydantic type and database nullability; do not assume omission and null are equivalent.

## Delete a todo

### `DELETE /todos/{todo_id}`

Deletion is permanent in the current implementation.

- success returns `204 No Content`;
- an unknown ID returns `404`.

Consumers that need recovery must not use this endpoint until an archive model is designed.

## Validation example

Invalid priority:

```json
{
  "title": "Invalid example",
  "priority": "urgent"
}
```

The service returns `422` before database mutation because the value is outside `low`, `medium`, and `high`.

## Compatibility policy

Treat these as breaking changes:

- removing or renaming a response field;
- changing a field type or enum value;
- making an optional request field mandatory;
- changing status codes or delete semantics;
- changing filter interpretation.

For any breaking change, add versioning or coordinate consumers, update this document, and add regression tests in the same pull request.
