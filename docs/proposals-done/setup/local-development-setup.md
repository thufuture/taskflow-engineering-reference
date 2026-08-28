# Thiết lập môi trường phát triển local

## Điều kiện cần

Python, Git và port 8000. Không cần PostgreSQL hoặc Redis.

## Cài đặt

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
pytest -q
uvicorn app.main:app --reload
```

## Cấu hình

Ứng dụng chỉ đọc `DATABASE_URL`. Giá trị local mặc định là SQLite. File `.env` không được commit và app không tự load dotenv.

## Xác minh

`GET /health` phải trả OK, `/docs` mở được và CRUD Todo hoạt động. Test phải chạy trên database in-memory, không làm đổi `todo.db`.

## Kiểm thử nhanh bằng Docker

Chạy bộ kiểm thử trong môi trường Python sạch để tránh phụ thuộc vào thư viện đã cài trên máy cá nhân:

```powershell
docker run --rm -v "${PWD}:/workspace" -w /workspace python:3.12-slim `
  sh -c "pip install -r requirements.txt && pytest -q"
```

Kết quả đạt yêu cầu khi toàn bộ test vượt qua, container kết thúc với mã `0` và không tạo tệp cấu hình bí mật trong repository. Nếu test thất bại, lưu lại tên test, stack trace và phiên bản image trước khi yêu cầu review.

## Xử lý sự cố

Nếu Python không nhận, kiểm tra PATH/launcher. Nếu port bận, dừng process hoặc đổi port. Nếu `422`, đọc validation detail. Nếu DB locked, kiểm tra server trùng và concurrent writers.
