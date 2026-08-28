# Sổ tay vận hành

## Phạm vi

Áp dụng cho local/tham chiếu, không khẳng định production readiness.

## Khởi động và xác minh

```bash
uvicorn app.main:app --reload
```

1. `GET /health` trả `200`.
2. `/docs` hiển thị OpenAPI.
3. `GET /todos` trả mảng JSON.
4. Khi cần kiểm tra write, tạo rồi xóa một Todo dùng thử.

Health chỉ kiểm tra process, không bảo đảm database.

## Cấu hình

Service đọc `DATABASE_URL`. Với SQLite path tương đối, vị trí file phụ thuộc working directory. Không in credential khi debug.

## Phân loại sự cố

- **Không start:** đọc traceback đầu; kiểm tra venv, dependency, port, DATABASE_URL, driver và quyền file.
- **422:** đọc `detail`, vị trí field và giá trị; đối chiếu `app/schemas.py`.
- **404:** kiểm tra path, ID và database đang kết nối.
- **500:** ghi commit SHA, endpoint, payload đã che dữ liệu, loại DB và traceback; kiểm tra connection/schema/lock; thêm regression test.
- **Database locked:** tìm process dùng file, dừng dev server trùng, kiểm tra session; đánh giá PostgreSQL nếu cần concurrent writes.

## Sao lưu và quay lui

Dừng writer trước khi copy SQLite. Đây không phải backup production. App-only rollback về revision đã test; schema rollback bằng migration chứ không dùng `create_all`; data defect phải backup trước repair.

## Bằng chứng sự cố

Timestamp/timezone, commit SHA, lệnh start, môi trường đã che secret, method/path/status, input nhỏ nhất và traceback đầu tiên.

Sau sự cố phải ghi root cause, thêm test, cập nhật runbook/ADR và kiểm tra secret không lọt vào log.
