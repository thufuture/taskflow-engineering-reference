# Thiết lập môi trường phát triển local

## Prerequisites

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

## Troubleshooting

Nếu Python không nhận, kiểm tra PATH/launcher. Nếu port bận, dừng process hoặc đổi port. Nếu `422`, đọc validation detail. Nếu DB locked, kiểm tra server trùng và concurrent writers.

