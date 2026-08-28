# Quy ước coding và review

## Mục tiêu

Quy ước giúp code TaskFlow dễ đọc, dễ test và tránh thay đổi API ngoài ý muốn.

## Python

Mọi thay đổi hành vi phải đi kèm kiểm thử hồi quy mô tả đúng trường hợp từng gây lỗi. Tên kiểm thử cần thể hiện điều kiện đầu vào và kết quả mong đợi để người rà soát hiểu ý định mà không phải đọc toàn bộ phần cài đặt.

- Dùng type hint cho function public.
- Handler ngắn, thể hiện rõ load, validate, mutate, commit và response.
- Không giữ SQLAlchemy session ở global scope.
- Không bắt `Exception` nếu không thể xử lý hoặc bổ sung ngữ cảnh.
- Tên function mô tả hành vi: `create_todo`, `list_todos`.
- Comment giải thích lý do và trade-off, không lặp lại code.

## API

- Pydantic schema là contract.
- Khai báo `response_model` và status code.
- Missing resource trả `404`; invalid input để FastAPI trả `422`.
- PATCH dùng `exclude_unset=True`.
- Breaking change cần tài liệu và regression test.

## Database

- Session theo request.
- Mutation liên quan phải nằm cùng transaction.
- Không dùng `create_all` thay migration production.
- Test dùng database in-memory cô lập.

## Commit và review

Commit nhỏ theo một mục đích. Reviewer kiểm tra contract, transaction, negative test, secret, compatibility và tài liệu. Không approve khi chỉ test happy path.

