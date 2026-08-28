# Hợp đồng API của Todo

## Quy ước

- Base URL local: `http://127.0.0.1:8000`.
- Body dùng `application/json`.
- Lỗi theo cấu trúc `detail` của FastAPI.
- ID do database sinh.
- Datetime được serialize dạng ISO.

## Biểu diễn dữ liệu

```json
{
  "id": 1,
  "title": "Đọc tài liệu kiến trúc",
  "description": "Hiểu request flow và transaction",
  "is_done": false,
  "priority": "high",
  "created_at": "2026-08-28T10:15:00",
  "updated_at": "2026-08-28T10:15:00"
}
```

## `GET /health`

Trả `200 {"status":"ok"}` khi web process phản hồi. Không kiểm tra database.

## `POST /todos`

```json
{
  "title": "Bổ sung contract test",
  "description": "Kiểm tra priority sai và ID không tồn tại",
  "priority": "high"
}
```

- `201`: tạo thành công.
- `422`: title hoặc priority không hợp lệ.

## `GET /todos`

Query tùy chọn: `is_done` boolean và `priority` thuộc `low|medium|high`.

```bash
curl "http://127.0.0.1:8000/todos?is_done=false&priority=high"
```

Trả mảng JSON, sắp xếp `created_at` giảm dần. Không có kết quả trả `[]`. Chưa có pagination.

## `GET /todos/{todo_id}`

- `200`: tìm thấy.
- `404`: ID không tồn tại.
- `422`: ID không phải số nguyên.

## `PATCH /todos/{todo_id}`

```json
{"is_done": true, "priority": "low"}
```

Chỉ field xuất hiện được cập nhật. Field bỏ qua giữ nguyên. Trả `200`, `404` hoặc `422`. Không mặc định coi `null` giống bỏ qua; phải đối chiếu Pydantic và database.

## `DELETE /todos/{todo_id}`

- `204 No Content`: xóa vĩnh viễn.
- `404`: ID không tồn tại.

## Khả năng tương thích

Xóa/đổi tên response field, đổi kiểu hoặc enum, biến field tùy chọn thành bắt buộc, đổi status code, delete semantics hoặc filter đều là breaking change và cần regression test.
