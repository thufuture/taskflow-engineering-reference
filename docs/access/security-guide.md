# Access and Security Guide

## Current posture

TaskFlow is a local reference application. It has no authentication or authorization and must not be exposed directly to the public internet. Every caller who can reach the API can read, create, modify, and delete all todos.

## Trust boundary

```text
Trusted local client -> TaskFlow API -> configured database
```

There is no implemented user identity, tenant boundary, role model, or row ownership. Adding a login page alone would not solve this; enforcement must occur at API endpoints and query boundaries.

## Configuration inventory

| Setting | Purpose | Secret? | Safe repository value |
|---|---|---:|---|
| `DATABASE_URL` | SQLAlchemy connection URL | potentially | SQLite example only |

The application reads process environment variables. It does not automatically parse `.env`. Local tooling may load the file before startup, but production platforms should inject configuration through their secret-management mechanism.

## Implemented controls

- Pydantic validates structured request data.
- SQLAlchemy generates parameterized SQL for ORM queries.
- API responses are constrained by response models.
- Dependencies are pinned to explicit package names in `requirements.txt`.

These controls reduce common input and query risks but do not establish access control.

## Known gaps

- no authentication or authorization;
- wildcard CORS;
- no TLS termination in the application;
- no rate limiting, request quota, or abuse protection;
- no audit log or correlation ID;
- no secret rotation workflow;
- no dependency or container vulnerability gate;
- no application-level encryption of todo content;
- no database readiness probe.

## Secret-handling rules

1. Never commit `.env`, database credentials, tokens, private keys, or production URLs.
2. Keep `.env.example` limited to placeholders and safe local defaults.
3. Treat connection strings as secrets because they may contain usernames and passwords.
4. Rotate a credential immediately if it appears in Git history; deleting the visible line is insufficient.
5. Do not include secrets in screenshots, issue descriptions, logs, or chatbot source documents.

## Data classification

The reference todo model is intended for synthetic or non-sensitive work descriptions. Until identity, retention, encryption, and audit requirements exist, do not store credentials, personal data, customer data, incident secrets, or regulated information.

## Production readiness gate

Before a non-local deployment, require all of the following:

- documented identity provider and token verification;
- explicit authorization rules for every operation;
- restrictive CORS allowlist;
- HTTPS at the trusted ingress;
- managed secret injection and rotation;
- PostgreSQL or another supported operational database;
- Alembic migrations and rollback procedures;
- structured logging with request correlation;
- rate and body-size limits;
- dependency scanning and patch policy;
- backup, restore, retention, and incident procedures;
- negative authorization and abuse-case tests.

## Security review prompts

For each new endpoint, reviewers should ask:

- Who is allowed to call it?
- Which rows may that caller access?
- Can supplied identifiers cross a tenant or owner boundary?
- Is request size bounded?
- Could any value reach logs or error messages as sensitive data?
- Are validation failures distinguishable without leaking internal details?
- Does the operation require an auditable event?

## Reporting a suspected issue

Do not publish exploitable details in a public issue. Share the minimal reproduction with the repository owner through a private channel, include affected versions and impact, and avoid using real credentials or production data.
