# Hướng dẫn codebase

## Thứ tự đọc

1. `README.md`: phạm vi và lệnh chạy.
2. `app/main.py`: composition.
3. `app/schemas.py`: contract public.
4. `app/routes.py`: hành vi endpoint.
5. `app/models.py`: persistence.
6. `app/database.py`: engine và session.
7. `tests/test_todos.py`: hành vi thực thi.

## Source map

```text
app/main.py       FastAPI, middleware, router, health
app/routes.py     CRUD và filter
app/schemas.py    request/response model
app/models.py     table Todo
app/database.py   DATABASE_URL, engine, session
tests/            test API database in-memory
```

## Contract layer

Khi thêm field, phải quyết định riêng: có nhận khi create, có cho PATCH, có trả về, nullable hay optional, default ở đâu. Đây là quyết định sản phẩm chứ không phải thao tác model máy móc.

## Route layer

Route có trách nhiệm lấy input, nhận session, query entity, map missing row thành `404`, commit mutation, refresh entity, khai báo response model và status code. Chỉ thêm service layer khi rule dùng chung, transaction phức tạp hoặc handler khó test.

## Persistence layer

`models.py` định nghĩa table. `database.py` tạo engine từ `DATABASE_URL`, session factory và đóng session sau request. Không giữ session global và không để test dùng file database developer.

## Recipe thêm field

1. Định nghĩa semantics trong product/API docs.
2. Cập nhật ORM.
3. Cập nhật schema liên quan.
4. Lập migration plan.
5. Test create/read/PATCH.
6. Cập nhật ví dụ và data model.

## Recipe thêm filter

Thêm query parameter có type, chỉ áp predicate khi parameter có mặt, test matching/non-matching/kết hợp/invalid, mô tả ordering và pagination nếu có.

## Review checklist

- Validation trước mutation?
- Omission và null có chủ ý?
- Có partial commit?
- Status code đúng contract?
- Missing resource nhất quán `404`?
- Test cô lập?
- Có security/compatibility impact?
- Comment giải thích “tại sao”?

## Dependency direction

```text
main -> routes -> schemas/models/database
tests -> public app behavior + isolated fixtures
```

Không import route vào model/database để tránh cycle.

## Debug sequence

Reproduce bằng curl nhỏ nhất; đọc status/detail; kiểm tra `DATABASE_URL`; trace handler; đối chiếu schema; kiểm tra model/transaction; thêm regression test trước khi fix.

