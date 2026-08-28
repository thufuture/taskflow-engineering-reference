# Hướng dẫn mã nguồn và cấu trúc thư mục

## Thứ tự đọc

Đọc `main.py` để hiểu composition; `schemas.py` để hiểu contract; `routes.py` để hiểu behavior; `models.py` và `database.py` để hiểu persistence; cuối cùng đọc test như executable specification.

## Thay đổi thường gặp

Thêm field phải cập nhật ORM, create/update/read schema, migration plan, API docs và test. Thêm filter phải dùng typed query parameter, chỉ áp predicate khi có giá trị và test filter kết hợp.

## Chiều phụ thuộc

```text
main -> routes -> schemas/models/database
tests -> public app + isolated fixtures
```

Không import route vào model/database. Không giữ session global.

## Gỡ lỗi

Reproduce nhỏ nhất, đọc status/detail, xác nhận database đích, trace handler, đối chiếu schema, kiểm tra transaction rồi thêm failing regression test trước khi sửa.
