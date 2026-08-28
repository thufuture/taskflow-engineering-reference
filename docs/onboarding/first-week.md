# Engineer First-Week Guide

## Goal

At completion, an engineer can run TaskFlow, explain its architecture, trace a request, add a tested change, and identify production-readiness gaps.

## Day 1: baseline

1. Follow `docs/setup/local-setup.md`.
2. Run `pytest -q`.
3. Start the API and open `/docs`.
4. Create, list, update, and delete one disposable todo.
5. Explain why `/health` does not prove database readiness.

Evidence: test result, one request/response example, and no secret/generated file staged.

## Day 2: request trace

Trace `POST /todos` through `app/main.py`, `app/schemas.py`, `app/routes.py`, `app/models.py`, and `app/database.py`. Then trace invalid priority or an unknown ID.

## Day 3: testing and data safety

- Identify the dependency override in `tests/test_todos.py`.
- Explain `StaticPool` with in-memory SQLite.
- Add one missing negative or boundary test without changing production behavior.
- Confirm local `todo.db` is untouched.

## Day 4: first reviewed change

Choose a small validation, filter-test, or docs improvement. Follow `CONTRIBUTING.md`: focused branch, one coherent change, tests/docs, descriptive commit, and a PR with risk and verification evidence.

## Day 5: operations and security

- Work through one runbook scenario.
- List current security gaps without claiming they are implemented.
- Explain when SQLite becomes unsuitable.
- Review ADR 0001 and propose a measurable revisit trigger.
- Present the change and respond to review.

## Knowledge checks

- Where is the public payload contract?
- When is a database session created and closed?
- What does PATCH omission mean?
- What happens for an unknown ID?
- Why is `create_all` not a migration strategy?
- Which database do tests use?
- Is public internet exposure safe today?
- Which artifacts change when adding a field?

## Asking for help

Provide exact command, expected and actual result, first relevant traceback, sanitized environment facts, and prior attempts. This enables useful help from engineers and source-grounded chatbots.
