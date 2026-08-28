# Onboarding kỹ sư tuần đầu

## Mục tiêu

Kỹ sư chạy được TaskFlow, giải thích kiến trúc, trace request, thêm thay đổi có test và nhận diện production gap.

## Ngày 1

Theo setup guide; chạy `pytest -q`; mở `/docs`; thực hiện CRUD; giải thích vì sao `/health` không kiểm tra DB. Lưu test output và request/response, không commit file local.

## Ngày 2

Trace `POST /todos` qua `main.py`, `schemas.py`, `routes.py`, `models.py`, `database.py`; sau đó trace invalid priority và missing ID.

## Ngày 3

Tìm dependency override; giải thích `StaticPool`; thêm negative/boundary test; xác minh `todo.db` không đổi.

## Ngày 4

Chọn thay đổi nhỏ; tạo branch tập trung, commit có nghĩa, cập nhật test/docs và viết PR có risk cùng evidence.

## Ngày 5

Thực hành runbook; liệt kê security gap; giải thích khi SQLite không phù hợp; đọc ADR; review PR đầu tiên.

## Câu hỏi kiểm tra

Contract ở đâu? Session đóng khi nào? PATCH omission là gì? Missing ID ra sao? Vì sao `create_all` không phải migration? Test dùng DB nào? Có expose Internet được chưa?

## Khi cần hỗ trợ

Cung cấp lệnh, expected/actual, traceback đầu, môi trường đã che secret và những cách đã thử.

