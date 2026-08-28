# Chiến lược kiểm thử

## Mục tiêu

Test bảo vệ API contract, cô lập dữ liệu và giúp refactor an toàn.

## Kiến trúc

`tests/test_todos.py` gọi FastAPI qua TestClient, override database bằng SQLite in-memory và `StaticPool`. Test không chạm `todo.db`.

## Ma trận

| Hành vi | Success | Invalid | Missing | Edge |
|---|---:|---:|---:|---:|
| Create | có | có | n/a | default, optional |
| List/filter | có | có | empty | filter kết hợp, ordering |
| Read | có | path | có | response shape |
| PATCH | có | có | có | omission/null |
| Delete | có | path | có | delete lặp |
| Health | có | n/a | n/a | liveness |

## Regression test

Reproduce nhỏ nhất; đặt tên theo hành vi; arrange tối thiểu; assert status/response; assert persisted state; xác minh fail trước fix và pass sau fix.

## Cô lập

Không dùng production session, không phụ thuộc thứ tự, tạo state trong fixture, không sleep nếu deterministic, restore dependency và không giả định generated ID.

## Lệnh

```bash
pytest -q
pytest -q tests/test_todos.py::test_name_here
```

Luôn chạy full suite trước review.

## Khoảng trống

Combined filter, ordering, malformed payload, invalid enum, PATCH null/omission, DB failure, concurrency và security test khi có auth.

