# Hướng dẫn đóng góp cho TaskFlow

Mỗi thay đổi phải nhỏ, dễ review, có test và phản ánh đúng hệ thống. Tài liệu và kiểm thử là một phần của feature.

## Branch

Dùng tên có mục đích: `feature/add-due-date`, `fix/patch-null-handling`, `docs/improve-runbook` hoặc `test/filter-boundaries`.

## Kiểm tra local

```bash
python -m pip install -r requirements.txt
pytest -q
uvicorn app.main:app --reload
```

## Commit

Một commit chứa một kết quả có ý nghĩa:

- `docs: bổ sung API contract tiếng Việt`
- `test: kiểm tra kết hợp nhiều bộ lọc`
- `fix: giữ nguyên field bị bỏ qua khi PATCH`

Không dùng nội dung mơ hồ như `update`, `changes`, `fix stuff`.

## Pull Request

PR phải nêu vấn đề, hành vi mong muốn, quyết định triển khai, ảnh hưởng API/dữ liệu/bảo mật, lệnh kiểm thử, cách rollback và phần chưa làm.

## Checklist review

- Contract có khớp schema và route?
- Validation có chạy trước mutation?
- Transaction và lỗi có chủ động?
- Có test success và failure?
- Có lộ secret hoặc file sinh local?
- Tài liệu có phân biệt hiện trạng với kế hoạch?
- Thay đổi lớn có ADR?

## Definition of Done

Toàn bộ test pass; hành vi mới có regression test; tài liệu cập nhật; không stage `.env`, database, cache, virtualenv, `node_modules` hoặc log; breaking change ghi rõ; feedback review được xử lý.

