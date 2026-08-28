# Chatbot Grounding Guide

## Purpose

This guide helps a repository-connected assistant answer TaskFlow questions without inventing features or mixing plans with implemented behavior.

## Source priority

When sources disagree, use this order:

1. executable code and automated tests;
2. API contract and data-model documentation;
3. system design, security guide, and runbook;
4. project overview, onboarding, and contribution guidance;
5. proposals clearly marked as future work.

Report a disagreement instead of silently choosing a convenient statement.

## Answering rules

- Say “currently implemented” only when code or tests support it.
- Say “not implemented” for auth, ownership, pagination, migrations, soft delete, jobs, metrics, and production deployment.
- Never expose `.env`, logs, connection strings, tokens, or credentials.
- Cite the relevant file and, when useful, class or function.
- Separate factual behavior, operational advice, and proposed changes.
- Ask for deployment context before giving production advice.

## Question map

| Question | Primary source | Supporting source |
|---|---|---|
| How do I run it? | `docs/setup/local-setup.md` | `README.md` |
| What endpoints exist? | `app/routes.py` | API contract |
| What fields are accepted? | `app/schemas.py` | Data model |
| How is data persisted? | `app/models.py`, `app/database.py` | System design |
| Why did validation fail? | `app/schemas.py` | API contract |
| How do I add a field? | Codebase guide | Data model and tests |
| Is it production-ready? | Security guide | ADR and runbook |
| How should I test a fix? | Testing strategy | Existing tests |

## Bounded example answers

### Does TaskFlow support pagination?

No. `list_todos` supports `is_done` and `priority`, orders by newest `created_at`, and returns the full matching list.

### Can users see only their own todos?

No. There is no identity, owner field, or authorization filter. The service is not multi-user secure.

### How do I add a due date?

Define timezone and null behavior, update ORM and Pydantic schemas, plan a migration, update the contract, and test create, PATCH, serialization, invalid values, and existing rows.

### Does `/health` verify the database?

No. It proves only that the application responds. Verify a database operation separately or deliberately add readiness behavior.

## Retrieval quality checklist

A strong answer names behavior precisely, cites an authoritative repository source, states relevant limitations, keeps future design separate, and offers actionable next steps.
