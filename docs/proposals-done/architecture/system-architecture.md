# Kiến trúc hệ thống TaskFlow

## Trạng thái

Đã triển khai và đang sử dụng.

## Kiến trúc

TaskFlow là modular monolith FastAPI. Request đi từ router qua Pydantic schema, route handler, SQLAlchemy session tới SQLite. Không có network hop nội bộ, queue hoặc background worker.

## Ranh giới module

- `main.py`: composition và lifecycle.
- `routes.py`: HTTP orchestration.
- `schemas.py`: contract.
- `models.py`: persistence.
- `database.py`: engine/session.
- `tests`: behavior evidence.

## Request flow

Create validate trước mutation, commit một lần và refresh entity. List áp filter tùy chọn và newest-first ordering. PATCH chỉ cập nhật field được gửi. Delete xóa vĩnh viễn.

## Quan sát vận hành

Mỗi request phải có `request_id` xuyên suốt access log và error log. Theo dõi tối thiểu latency p50/p95, tỷ lệ lỗi theo endpoint và số kết nối database đang sử dụng. Health check chỉ xác nhận tiến trình còn hoạt động; readiness check phải xác nhận ứng dụng có thể truy cập dependency bắt buộc trước khi nhận traffic.

## Giới hạn

SQLite phù hợp local nhưng giới hạn concurrent write. `create_all` không phải migration. Chưa có auth, readiness, metrics hoặc trace.

## Khi mở rộng

Chỉ thêm service layer khi rule dùng chung hoặc transaction phức tạp. Chuyển PostgreSQL khi có nhu cầu concurrency/durability đo được. Không tách microservice nếu chưa có ownership hoặc scaling boundary.
