# Thiết kế hệ thống

## Mục tiêu và phạm vi

TaskFlow là REST API quản lý công việc dạng modular monolith nhỏ. Toàn bộ HTTP, validation và database chạy trong một FastAPI process để kỹ sư theo dõi request end-to-end. Auth, background job, notification và multi-tenant chưa được triển khai.

## Sơ đồ ngữ cảnh

```text
API Client
   │ HTTP + JSON
   ▼
FastAPI (app/main.py)
   ├── Router Todo (app/routes.py)
   ├── Contract Pydantic (app/schemas.py)
   ├── Entity SQLAlchemy (app/models.py)
   └── Session (app/database.py)
              │
              ▼
        SQLite mặc định
```

## Trách nhiệm thành phần

| Thành phần | Trách nhiệm |
|---|---|
| `app/main.py` | Tạo app, CORS, router, health và init table |
| `app/routes.py` | HTTP behavior, filter, CRUD, status code |
| `app/schemas.py` | Validate request và serialize response |
| `app/models.py` | Khai báo cột và default được lưu |
| `app/database.py` | Engine, session factory, vòng đời session |
| `tests/test_todos.py` | Bảo vệ hành vi public bằng DB cô lập |

## Luồng tạo Todo

1. Client gửi `POST /todos`.
2. FastAPI chọn `create_todo`.
3. `TodoCreate` validate title và priority trước khi handler chạy.
4. Handler tạo entity, add, commit và refresh.
5. `TodoRead` serialize kết quả; trả `201`.
6. Input sai trả `422`; không ghi database.

## Luồng đọc danh sách

Handler tạo query Todo, thêm predicate khi có `is_done` hoặc `priority`, sắp xếp `created_at.desc()`, rồi serialize toàn bộ kết quả. Chưa có pagination.

## Luồng PATCH

Handler tải entity hoặc trả `404`. `model_dump(exclude_unset=True)` chỉ lấy field client thật sự gửi. Sau khi gán field, handler commit và refresh. Field bị bỏ qua giữ nguyên.

## Transaction và nhất quán

- Mỗi request dùng session riêng từ `get_db`.
- Mutation commit một lần.
- Response lấy từ entity đã refresh.
- Delete là hard delete.
- Chưa có transaction liên hệ thống.

Nếu feature cần nhiều mutation atomic, giữ chúng trong cùng session và chỉ commit sau khi mọi invariant đạt.

## Startup và schema

Startup gọi `init_db` rồi `Base.metadata.create_all`. Cơ chế này chỉ tạo table thiếu, không thể thay thế migration để rename cột, backfill hoặc rollback. Phải thêm Alembic trước schema change production.

## Concurrency

Route và SQLAlchemy hiện dùng synchronous API; FastAPI chạy sync handler trong thread pool. SQLite phù hợp local và write thấp nhưng có thể khóa khi nhiều writer. Khi có contention thật, đánh giá PostgreSQL thay vì tăng timeout tùy tiện.

## Security boundary

Mọi caller có thể thao tác mọi Todo. Wildcard CORS chỉ phù hợp môi trường tham chiếu local. Trước Internet công cộng cần authn, authz, CORS allowlist, TLS, secret management và request limits.

## Observability

Có health endpoint, access log và exception log của Uvicorn. Chưa có DB readiness, metrics, trace, correlation ID hoặc audit event.

## Failure mode

| Lỗi | Biểu hiện | Kiểm tra đầu tiên |
|---|---|---|
| Input sai | `422` | Đọc validation detail |
| ID không có | `404` | Kiểm tra ID và DB đích |
| DB không mở | startup/500 | `DATABASE_URL`, quyền file, driver |
| SQLite locked | 500 | Process ghi đồng thời, transaction dài |
| Port bận | không start | Dừng process hoặc đổi port |

## Invariant kiến trúc

1. Pydantic định nghĩa contract public.
2. Session theo request, không lưu global.
3. Test không dùng `todo.db` của developer.
4. Schema change cần migration strategy.
5. Tài liệu phân biệt rõ hiện trạng và kế hoạch.

