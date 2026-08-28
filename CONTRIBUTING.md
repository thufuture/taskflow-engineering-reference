# Contributing to TaskFlow

Keep changes small, reviewable, tested, and truthful. Documentation and tests are part of each feature.

## Branches

Use focused names such as `feature/add-due-date`, `fix/patch-null-handling`, `docs/improve-runbook`, or `test/filter-boundaries`.

## Verify

```bash
python -m pip install -r requirements.txt
pytest -q
```

Use `uvicorn app.main:app --reload` for manual verification.

## Commits

Write one imperative result per commit, for example `docs: add source-grounded API contract` or `fix: preserve omitted fields during patch`. Avoid vague or manufactured history.

## Pull request

State the problem, decisions, API/data/security impact, verification results, and deliberate out-of-scope work.

## Definition of done

- tests pass and new behavior has regression coverage;
- docs and examples are current;
- no `.env`, local database, cache, virtualenv, vendored dependencies, or logs are staged;
- compatibility and breaking changes are explicit;
- review feedback is resolved or documented.
