# ADR 0001: Use a Single FastAPI Service with SQLAlchemy

- Status: Accepted
- Date: 2026-08-28
- Owners: TaskFlow maintainers

## Context

The project must teach complete backend request flow with minimal prerequisites. One CRUD domain does not justify distributed services or messaging. Persistence should remain replaceable so local SQLite can later give way to a managed relational database.

## Decision

Use one FastAPI application with route, schema, model, and database modules. Use SQLAlchemy and default local SQLite. Keep synchronous handlers and request-scoped sessions.

## Alternatives

- **Microservices:** rejected; no ownership or scaling boundary justifies the overhead.
- **Memory-only storage:** rejected; persistence is an important learning concern.
- **PostgreSQL everywhere:** deferred; better for production, but raises local setup cost.
- **Async ORM:** deferred; current workload does not justify lifecycle complexity.

## Consequences

Benefits include end-to-end traceability, simple local startup, fast tests, and low operational cost. Costs include SQLite write limits, lack of production migrations, synchronous database access, and a shared process failure unit.

## Guardrails

- Keep sessions request-scoped.
- Avoid module cycles.
- Add Alembic before incompatible schema changes.
- Do not recommend SQLite as the production default.
- Record cross-cutting decisions as ADRs.

## Revisit triggers

- sustained database lock contention;
- need for multiple replicas;
- transactions across several domains;
- background work exceeding HTTP budgets;
- separate deployment ownership;
- measured reliability or throughput targets the design cannot meet.
