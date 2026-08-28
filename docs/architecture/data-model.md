# Mô hình dữ liệu

## Thực thể Todo

| Field | Kiểu | Bắt buộc khi tạo | Ý nghĩa |
|---|---|---:|---|
| `id` | integer | tự sinh | Primary key |
| `title` | string | có | Tóm tắt công việc, 1–200 ký tự |
| `description` | string/null | không | Bối cảnh bổ sung |
| `is_done` | boolean | không | Mặc định `false` |
| `priority` | enum | không | `low|medium|high`, mặc định `medium` |
| `created_at` | datetime | tự sinh | Thời điểm tạo |
| `updated_at` | datetime | tự sinh | Thời điểm cập nhật |

Khai báo persistence chuẩn nằm trong `app/models.py`; validation API nằm trong `app/schemas.py`. Khi đổi field phải review cả hai.

## Vai trò lược đồ

- `TodoCreate`: dữ liệu client được gửi khi tạo.
- `TodoUpdate`: field mutable dạng optional để PATCH phân biệt bỏ qua.
- `TodoRead`: response public ổn định và serialize từ ORM.

Không trả ORM object từ endpoint mới nếu thiếu response model, vì field database thêm sau có thể bị lộ ngoài ý muốn.

## Bất biến đã được cưỡng chế

- priority thuộc enum hợp lệ;
- create bắt buộc title;
- ID duy nhất do database sinh;
- is_done và priority có default;
- endpoint xử lý ID không tồn tại.

## Giả định chưa được cưỡng chế đầy đủ

- title nên có nội dung, không chỉ khoảng trắng;
- timestamp nên được hiểu thống nhất theo UTC;
- description không nên quá lớn;
- các record cùng `created_at` chưa có secondary ordering.

Nếu giả định thành yêu cầu, phải thêm validation và test.

## Quan hệ

Hiện không có foreign key hoặc relationship. Todo chưa có owner, project, label, comment hoặc dependency. Thêm các khái niệm này sẽ ảnh hưởng auth và delete semantics, cần ADR và migration.

## Danh sách kiểm tra khi thêm trường

1. Xác định ý nghĩa, null và omission.
2. Cập nhật SQLAlchemy model.
3. Cập nhật schema create/update/read phù hợp.
4. Quyết định default và backfill.
5. Tạo migration trước production.
6. Cập nhật API docs.
7. Test create, read, PATCH, invalid input và row cũ.

Ví dụ thêm `due_at` phải trả lời timezone, có cho phép quá khứ, có thể clear bằng null, cần filter/sort gì và backfill row cũ ra sao.
