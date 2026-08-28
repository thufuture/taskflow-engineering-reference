# Testing Strategy

Tests protect observable API behavior, isolate persistence state, and make refactoring safe.

## Current architecture

`tests/test_todos.py` calls the FastAPI app through an HTTP test client. It overrides the production database dependency with in-memory SQLite and `StaticPool`, so requests share one transient test database without touching `todo.db`.

## Required coverage matrix

| Behavior | Success | Invalid input | Missing resource | Edge cases |
|---|---:|---:|---:|---:|
| Create | required | required | n/a | defaults, optional fields |
| List/filter | required | required | empty list | combined filters, newest-first order |
| Read | required | path validation | required | response shape |
| PATCH | required | required | required | omission versus null |
| Delete | required | path validation | required | repeated delete |
| Health | required | n/a | n/a | liveness-only scope |

## Regression-test workflow

1. Reproduce the defect with the smallest request.
2. Use a behavior name such as `test_patch_omitted_title_keeps_existing_value`.
3. Arrange only required data.
4. Assert status code and meaningful response fields.
5. Assert persisted state when mutation matters.
6. Confirm failure before the fix and success after it.

## Isolation rules

- Never mutate through the production session factory.
- Never depend on execution order.
- Create state in the test or an explicit fixture.
- Cleanly override and restore FastAPI dependencies.
- Avoid generated-ID assumptions unless identity is under test.

## Commands

```bash
pytest -q
pytest -q tests/test_todos.py::test_name_here
```

Run the full suite before review.

## Current gaps

- combined filters and explicit newest-first ordering;
- malformed payload and invalid enum coverage;
- PATCH null versus omission;
- database-failure mapping;
- concurrency behavior;
- tests against a future production database;
- authorization tests once access control exists.

## Review checklist

- Can the test fail for the intended regression?
- Is state local and deterministic?
- Does it assert persistence where needed?
- Does it cover a negative path?
- Would it remain valid after internal refactoring?
