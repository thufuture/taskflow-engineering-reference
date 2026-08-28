# Thiết lập môi trường trên máy cá nhân

## Yêu cầu

- Python phù hợp với dependency hiện tại;
- Git;
- terminal PowerShell, cmd hoặc shell Unix;
- port 8000 còn trống.

Không cần PostgreSQL, Redis hoặc Node.js để chạy TaskFlow.

## Cài đặt trên Windows

```powershell
cd D:\duong-dan\taskflow-engineering-reference
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Nếu PowerShell chặn script, dùng cmd với `.venv\Scripts\activate.bat` hoặc điều chỉnh execution policy theo quy định máy.

## Cấu hình cơ sở dữ liệu

Ứng dụng chỉ đọc `DATABASE_URL` từ process environment và không tự load file `.env`.

PowerShell:

```powershell
$env:DATABASE_URL="sqlite:///./todo.db"
```

Không commit `.env`. `.env.example` chỉ chứa giá trị local an toàn.

## Chạy kiểm thử trước

```powershell
pytest -q
```

Test override `get_db` và dùng SQLite in-memory, không ghi vào `todo.db`.

## Chạy máy chủ

```powershell
uvicorn app.main:app --reload
```

Khi startup, `init_db` gọi `create_all`. Đây không phải migration system.

## Kiểm thử nhanh

```powershell
Invoke-RestMethod http://127.0.0.1:8000/health
Invoke-RestMethod http://127.0.0.1:8000/todos
```

Tạo Todo:

```powershell
$body = @{ title = "Đọc API contract"; priority = "high" } | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/todos -ContentType application/json -Body $body
```

## Lỗi thường gặp

| Triệu chứng | Nguyên nhân | Cách xử lý |
|---|---|---|
| `python` không nhận | Chưa cài/PATH sai | Dùng Python Launcher hoặc cài đúng PATH |
| `ModuleNotFoundError` | Chưa activate/cài dependency | Activate venv, pip install |
| Port 8000 bận | Server khác đang chạy | Dừng process hoặc dùng `--port 8001` |
| Không tạo DB | Thư mục không có quyền ghi | Kiểm tra path và permission |
| `422` | Payload sai contract | Đọc `detail` và schema |
| SQLite locked | Nhiều process ghi | Dừng server trùng, kiểm tra transaction |

## Danh sách kiểm tra hoàn tất

Test pass; health trả OK; OpenAPI mở được; CRUD hoạt động; không có secret hay database được stage.
