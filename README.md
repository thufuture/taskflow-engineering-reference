# TaskFlow API Reference

TaskFlow là dự án API quản lý công việc mẫu, giúp kỹ sư học toàn bộ luồng backend từ HTTP, validation, xử lý nghiệp vụ, lưu database đến kiểm thử. Repo cũng là nguồn chuẩn để kiểm thử đồng bộ GitHub và chatbot có trích dẫn của Ralion.

## Phạm vi hiện tại

Hệ thống đã có:

- CRUD công việc và cập nhật một phần bằng PATCH;
- lọc theo trạng thái hoàn thành và độ ưu tiên;
- FastAPI, Pydantic, SQLAlchemy và SQLite;
- OpenAPI tại `/docs`, health check tại `/health`;
- test API trên database SQLite in-memory độc lập.

Hệ thống **chưa có** đăng nhập, phân quyền, owner, phân trang, migration Alembic, soft delete, rate limit, metrics hay kiến trúc production. Không triển khai trực tiếp repo này ra Internet công cộng.

## Công nghệ

| Thành phần | Công nghệ | Vai trò |
|---|---|---|
| Web API | FastAPI | Routing, dependency injection, OpenAPI |
| Contract | Pydantic | Kiểm tra request và serialize response |
| Persistence | SQLAlchemy | Model, query và transaction |
| Local database | SQLite | Lưu dữ liệu phát triển |
| Test | Pytest + TestClient | Kiểm thử HTTP và database |
| Server | Uvicorn | Chạy ứng dụng ASGI |

## Chạy nhanh

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
pytest -q
uvicorn app.main:app --reload
```

Kiểm tra:

```bash
curl http://127.0.0.1:8000/health
curl http://127.0.0.1:8000/todos
```

Tạo công việc:

```bash
curl -X POST http://127.0.0.1:8000/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Đọc tài liệu kiến trúc","priority":"high"}'
```

## Cấu trúc repo

```text
app/
├── main.py       # FastAPI, CORS, router, health và startup
├── routes.py     # CRUD và filter Todo
├── schemas.py    # Contract request/response
├── models.py     # Entity SQLAlchemy
└── database.py   # Engine, session factory và dependency
tests/            # Test tích hợp bằng database in-memory
docs/             # Tài liệu sản phẩm, kỹ thuật và vận hành
```

## Bản đồ tài liệu

| Nhu cầu | Tài liệu |
|---|---|
| Hiểu mục tiêu và giới hạn | [Tổng quan dự án](docs/product/project-overview.md) |
| Hiểu kiến trúc và request flow | [Thiết kế hệ thống](docs/architecture/system-design.md) |
| Hiểu entity và quy tắc dữ liệu | [Mô hình dữ liệu](docs/architecture/data-model.md) |
| Tích hợp API | [API Todo](docs/api/todo-api.md) |
| Cài môi trường local | [Thiết lập local](docs/setup/local-setup.md) |
| Kiểm tra an toàn | [Hướng dẫn bảo mật](docs/access/security-guide.md) |
| Tìm vị trí sửa code | [Hướng dẫn codebase](docs/codebase-guide.md) |
| Xử lý lỗi vận hành | [Runbook](docs/operations/runbook.md) |
| Viết và review test | [Chiến lược kiểm thử](docs/testing/testing-strategy.md) |
| Onboarding kỹ sư | [Kế hoạch tuần đầu](docs/onboarding/first-week.md) |
| Grounding chatbot | [Hướng dẫn chatbot](docs/onboarding/chatbot-grounding.md) |
| Quyết định kiến trúc | [ADR-0001](docs/adr/0001-single-service.md) |
| Quy trình đóng góp | [CONTRIBUTING](CONTRIBUTING.md) |

## Luồng request

```text
Client -> FastAPI route -> Pydantic schema -> SQLAlchemy session
       -> SQLite/database -> response_model -> JSON response
```

Mỗi request nhận một session riêng qua `get_db`. Handler ghi dữ liệu commit sau khi áp dụng thay đổi và refresh entity trước khi trả response.

## Definition of Done

Một thay đổi chỉ hoàn thành khi hành vi rõ ràng, schema và model nhất quán, có test success/failure, tài liệu cập nhật, đã đánh giá compatibility/security, toàn bộ test pass và không commit secret hay file sinh local.

