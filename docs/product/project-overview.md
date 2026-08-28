# Project Overview

## Product statement

TaskFlow is a compact work-item API used to demonstrate how an engineering team can build, test, document, and evolve a source-grounded backend service. It provides one consistent place to create todos, assign priority, mark completion, query work, and remove obsolete items.

The repository is deliberately small enough to understand end to end, but its documentation and engineering practices are structured like a production project. This makes it suitable for onboarding exercises, API integration practice, and Ralion knowledge retrieval tests.

## Problem being solved

Teams often teach backend conventions through disconnected snippets. New engineers can see a route or model but cannot answer how a request flows, which contract is authoritative, how changes are tested, or what is unsafe in production. TaskFlow provides a coherent reference system where those relationships are explicit.

## Target users

| Persona | Goal | Primary material |
|---|---|---|
| Backend engineer | Implement a safe API change | Architecture, codebase guide, API contract |
| QA engineer | Design contract and regression tests | API contract, testing strategy |
| DevOps/SRE learner | Run and diagnose the service | Setup guide, operations runbook |
| Security reviewer | Identify current controls and gaps | Security guide |
| Technical lead/reviewer | Evaluate change scope and trade-offs | ADR, contribution guide |
| Chatbot user | Get answers grounded in repository facts | Entire documentation set plus source code |

## Core user journeys

1. Create a work item with a title, optional description, and priority.
2. List work items and narrow the result using supported filters.
3. Read a specific item by its identifier.
4. Partially update mutable fields without replacing the whole entity.
5. Mark an item complete.
6. Permanently delete an item when it is no longer needed.

## Implemented scope

- REST endpoints implemented with FastAPI;
- request and response validation with Pydantic;
- SQLAlchemy persistence;
- SQLite as the default local database;
- generated OpenAPI documentation;
- isolated automated API tests;
- health endpoint for process liveness.

## Explicit non-goals

The current code does not implement:

- login, identity, roles, ownership, or tenants;
- due dates, tags, projects, comments, or todo dependencies;
- soft deletion or historical revisions;
- event delivery, notifications, scheduled jobs, or queues;
- production database migrations;
- metrics, traces, audit logs, or database readiness checks.

Documentation must never describe these as available features. Proposed additions should be labeled as proposals until merged and tested.

## Product rules

- A todo must have a title at creation time.
- Priority is limited to `low`, `medium`, or `high`.
- New todos are incomplete unless the caller explicitly supplies another accepted value.
- PATCH changes only fields included by the caller.
- Reading, updating, or deleting an unknown ID returns not found.
- Deletion is permanent.

## Success criteria for the reference project

The project is successful when a new engineer can:

- start it locally using only repository instructions;
- explain the request path from HTTP input to database commit;
- locate the correct file for a schema, route, or persistence change;
- add a tested field or endpoint without silently changing existing contracts;
- state the service's security limitations accurately;
- diagnose common startup, validation, and database failures.

## Definition of done for a feature

A change is complete only when:

1. behavior and edge cases are agreed;
2. schemas and persistence rules are consistent;
3. success and failure paths have tests;
4. API and engineer documentation are updated;
5. security and compatibility impact is reviewed;
6. the full test suite passes from a clean environment.

## Glossary

| Term | Meaning in this repository |
|---|---|
| Todo | The persisted work-item entity |
| Contract | Observable HTTP inputs, outputs, and error behavior |
| Schema | A Pydantic request or response model, unless “database schema” is stated |
| Session | A request-scoped SQLAlchemy unit of work |
| Reference environment | Local or controlled learning use, not an internet-facing production deployment |
| Source-grounded answer | An answer traceable to code or documentation in this repository |

## Ownership and decision records

Cross-cutting architectural choices belong in `docs/adr`. Public behavior belongs in the API contract. Operational facts belong in the runbook. A pull request that changes one of these areas must update the corresponding source of truth.
