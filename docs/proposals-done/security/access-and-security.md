# Quyền truy cập và bảo mật

## Trạng thái

TaskFlow chưa có authentication, authorization, owner hoặc tenant. Mọi caller truy cập được API đều thao tác được mọi Todo. Chỉ dùng trong môi trường local tin cậy.

## Thông tin bí mật

Không commit `.env`, PAT, API key, private key hoặc connection string production. `.env.example` chỉ chứa cấu hình an toàn. Secret từng vào Git history phải rotate.

## Biện pháp kiểm soát hiện có

Pydantic validation, SQLAlchemy parameterized query, response model và test database cô lập.

## Biện pháp kiểm soát còn thiếu

Authn/authz, CORS allowlist, TLS, rate/body limit, audit log, correlation ID, secret rotation, scanning gate và DB readiness.

## Danh sách xử lý khi phát hiện sự cố

1. Thu hồi hoặc xoay vòng credential nghi bị lộ.
2. Ghi nhận thời điểm, phạm vi ảnh hưởng và `request_id` liên quan.
3. Bảo toàn log phục vụ điều tra; không đưa dữ liệu nhạy cảm vào issue công khai.
4. Vá nguyên nhân gốc, bổ sung regression test và chỉ khôi phục dịch vụ sau khi người phụ trách bảo mật xác nhận.

## Điều kiện đưa lên môi trường thật

Trước deployment thật cần identity, row-level authorization, managed secrets, PostgreSQL, Alembic, structured logs, limits, backup/restore và negative security tests.
