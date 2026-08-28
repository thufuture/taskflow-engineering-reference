# Quyền truy cập và bảo mật

## Trạng thái

TaskFlow chưa có authentication, authorization, owner hoặc tenant. Mọi caller truy cập được API đều thao tác được mọi Todo. Chỉ dùng trong môi trường local tin cậy.

## Secret

Không commit `.env`, PAT, API key, private key hoặc connection string production. `.env.example` chỉ chứa cấu hình an toàn. Secret từng vào Git history phải rotate.

## Control hiện có

Pydantic validation, SQLAlchemy parameterized query, response model và test database cô lập.

## Control còn thiếu

Authn/authz, CORS allowlist, TLS, rate/body limit, audit log, correlation ID, secret rotation, scanning gate và DB readiness.

## Production gate

Trước deployment thật cần identity, row-level authorization, managed secrets, PostgreSQL, Alembic, structured logs, limits, backup/restore và negative security tests.

