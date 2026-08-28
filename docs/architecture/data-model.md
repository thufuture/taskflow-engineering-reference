# Data Model

## Entity overview

The current domain has one persisted entity: `Todo`. It represents a work item and its completion state.

| Field | Type | Required on create | Persisted behavior |
|---|---|---:|---|
| `id` | integer | generated | Primary key |
| `title` | string | yes | Human-readable summary |
| `description` | string or null | no | Additional context |
| `is_done` | boolean | no | Defaults to `false` |
| `priority` | `low`, `medium`, `high` | no | Defaults to `medium` |
| `created_at` | datetime | generated | Creation timestamp |
| `updated_at` | datetime | generated | Last database-managed update timestamp |

The canonical declaration is `app/models.py`; API-level validation lives in `app/schemas.py`. Both must be reviewed when a field changes.

## API schema roles

- `TodoCreate` defines what clients may provide when creating a record.
- `TodoUpdate` makes mutable fields optional so PATCH can distinguish omitted values.
- `TodoRead` defines the stable response representation and enables ORM attribute serialization.

Never return an unrestricted ORM object from a new endpoint without a response model. An explicit response schema prevents accidental exposure when columns are added later.

## Invariants

### Enforced by code

- priority must be one of the enum values accepted by Pydantic;
- `title` is required by the create contract;
- `id` is database-generated and unique;
- `is_done` and `priority` receive defaults when omitted;
- unknown IDs are rejected by endpoint behavior.

### Assumed but not fully enforced

- titles should be meaningful rather than whitespace-only;
- timestamps should be interpreted consistently as UTC;
- descriptions should remain small enough for normal JSON responses;
- clients should recognize that list results are explicitly ordered by `created_at` descending, with no secondary tie-breaker.

If any assumption becomes a product requirement, implement validation and add contract tests rather than relying on documentation alone.

## Relationship model

There are currently no foreign keys or relationships. A todo does not have an owner, project, label, comment, or dependency. Adding one of those concepts changes authorization and deletion behavior and therefore requires an ADR plus migration planning.

## Field-change checklist

For a new or modified field:

1. define the product meaning and whether null differs from omission;
2. update the SQLAlchemy model;
3. update create, update, and read schemas as appropriate;
4. decide default and backfill behavior for existing rows;
5. add a migration before production deployment;
6. update endpoint examples and compatibility notes;
7. test create, read, update, filtering, invalid input, and legacy rows.

## Example: evaluating a future `due_at` field

Before implementation, answer:

- Is it optional?
- Must it include an offset?
- Is a past deadline valid?
- Can PATCH clear it with `null`?
- Does listing need a range filter or sort order?
- How will old rows be backfilled?

These answers belong in the contract and tests, not only in the database column definition.
