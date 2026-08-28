# Hướng dẫn truy cập và bảo mật

## Hiện trạng

TaskFlow là ứng dụng tham chiếu local, chưa có authentication hoặc authorization. Bất kỳ caller nào truy cập API đều có thể đọc, tạo, sửa và xóa mọi Todo. Không expose trực tiếp ra Internet.

## Ranh giới tin cậy

```text
Client local được tin cậy -> TaskFlow API -> Database cấu hình
```

Không có identity, tenant, role hoặc row ownership. Chỉ thêm login UI không giải quyết được; enforcement phải nằm tại API và query boundary.

## Cấu hình

| Biến | Mục đích | Nhạy cảm |
|---|---|---:|
| `DATABASE_URL` | SQLAlchemy connection URL | Có thể chứa credential |

Ứng dụng đọc process environment. Production phải inject secret bằng secret manager của platform.

## Biện pháp kiểm soát hiện có

- Pydantic validate structured input.
- SQLAlchemy ORM tạo parameterized SQL.
- Response model giới hạn dữ liệu trả về.
- Test database cô lập với local database.

## Khoảng trống

- chưa có authn/authz;
- wildcard CORS;
- chưa có TLS termination trong app;
- chưa rate limit/body limit;
- chưa audit log/correlation ID;
- chưa có secret rotation;
- chưa có dependency scanning gate;
- chưa có DB readiness.

## Quy tắc thông tin bí mật

1. Không commit `.env`, token, private key hoặc production URL.
2. `.env.example` chỉ chứa placeholder an toàn.
3. Connection string được xem là secret.
4. Secret đã vào Git history phải rotate ngay; xóa dòng chưa đủ.
5. Không để secret trong screenshot, issue, log hoặc tài liệu chatbot.

## Dữ liệu

Chỉ dùng dữ liệu giả hoặc không nhạy cảm. Không lưu credential, PII, customer data, incident secret hoặc dữ liệu regulated.

## Điều kiện trước khi đưa lên môi trường thật

Cần token verification, authorization rule, CORS allowlist, HTTPS, managed secrets, production database, Alembic, structured log, correlation ID, limits, dependency scanning, backup/restore và negative authorization tests.

## Câu hỏi rà soát bảo mật

- Ai được gọi endpoint?
- Caller được truy cập row nào?
- ID có vượt tenant/owner boundary?
- Request size có giới hạn?
- Giá trị nào có thể lọt vào log?
- Operation có cần audit?
