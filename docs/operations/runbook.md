# Operations Runbook

## Scope

This runbook covers the current local/reference deployment. It does not imply production readiness.

## Start the service

From the repository root with dependencies installed:

```bash
uvicorn app.main:app --reload
```

Expected endpoint: `http://127.0.0.1:8000`.

## Verify service state

1. Call `GET /health` and expect `200` with `{"status":"ok"}`.
2. Open `/docs` and confirm OpenAPI renders.
3. Call `GET /todos` and confirm a JSON array.
4. For write verification, create a disposable todo and then delete it.

The health endpoint verifies process liveness only. A successful health response does not prove every database operation will succeed.

## Configuration

The service reads `DATABASE_URL` from the process environment. If absent, it uses the SQLite default defined in `app/database.py`. Confirm the effective working directory because a relative SQLite path is resolved from the process context.

Do not print credentials while diagnosing a non-SQLite URL. Record the database type and host separately from secrets.

## Incident triage

### API does not start

1. Read the first traceback rather than later cascading errors.
2. Confirm the virtual environment and dependency installation.
3. Verify port 8000 is available.
4. Check `DATABASE_URL` syntax and driver availability.
5. Confirm the process can create/open the SQLite file directory.

### Requests return `422`

This usually indicates contract validation, not infrastructure failure. Inspect the `detail` array for field location, error type, and received value. Compare it with `app/schemas.py` and the API contract.

### Requests return `404`

Confirm both route path and resource ID. A missing todo is expected domain behavior. Also verify the application is connected to the database you believe it is using.

### Requests return `500`

Capture the traceback, endpoint, sanitized payload shape, and database target type. Check for connection/open errors, invalid schema state, or SQLite locking. Add a regression test once the cause is known.

### SQLite reports “database is locked”

1. Identify other running processes using the same file.
2. Stop duplicate development servers if safe.
3. Confirm handlers are not holding sessions beyond a request.
4. Reassess whether the workload requires PostgreSQL rather than increasing arbitrary timeouts.

## Backup and restore for local SQLite

For valuable local data, stop writers before copying the database file. A filesystem copy during active writes may be inconsistent. Restore by placing the verified copy at the configured path before startup.

This is not a production backup strategy. A production database requires automated backups, retention, restore drills, monitoring, and an agreed recovery point/time objective.

## Rollback guidance

- Application-only change: deploy the previously tested revision.
- Contract change: verify clients remain compatible before rollback.
- Schema change: do not rely on `create_all`; use a tested migration downgrade or forward fix.
- Data mutation defect: preserve evidence and take a backup before repair.

## Logs and evidence

Uvicorn access logs provide method, path, and status. Exception tracebacks appear in server output. The application currently has no correlation ID, structured event log, metric, trace, or audit trail.

When reporting an incident, include:

- timestamp and timezone;
- code revision;
- command used to start the service;
- sanitized environment facts;
- request method/path and response status;
- smallest reproducible input;
- full first traceback without secrets.

## Post-incident checklist

- document root cause rather than only the symptom;
- add regression coverage;
- update this runbook if diagnosis was unclear;
- record architectural follow-up in an ADR when appropriate;
- verify no credential or sensitive todo data entered logs or tickets.
