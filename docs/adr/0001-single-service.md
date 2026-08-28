# ADR 0001: Dùng một dịch vụ FastAPI với SQLAlchemy

- Trạng thái: Đã chấp nhận
- Ngày: 2026-08-28
- Chủ sở hữu: Nhóm TaskFlow

## Bối cảnh

Dự án cần dạy request flow hoàn chỉnh với ít prerequisite. Một domain CRUD chưa đủ lý do dùng microservice hoặc broker. Persistence cần thay được để SQLite local có thể chuyển sang managed relational database.

## Quyết định

Dùng một FastAPI application gồm route, schema, model và database. Dùng SQLAlchemy, SQLite mặc định, synchronous handler và request-scoped session.

## Phương án

- **Microservices:** từ chối vì chưa có domain/ownership/scaling boundary.
- **In-memory only:** từ chối vì persistence quan trọng.
- **Bắt buộc PostgreSQL:** hoãn vì tăng chi phí setup.
- **Async ORM:** hoãn vì workload chưa chứng minh nhu cầu.

## Hệ quả

Ưu: trace end-to-end, startup đơn giản, test nhanh. Nhược: SQLite giới hạn write, chưa migration production, DB sync và một failure unit chung.

## Rào chắn và điều kiện xem xét lại

Session theo request; tránh cycle; thêm Alembic trước incompatible schema; không dùng SQLite làm production default. Xem xét lại khi lock kéo dài, cần replica, transaction nhiều domain, background work, ownership độc lập hoặc không đạt SLO.
