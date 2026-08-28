# Tổng quan dự án TaskFlow

## Bài toán

Tài liệu onboarding backend thường rời rạc: kỹ sư thấy route hoặc model nhưng không biết request đi qua đâu, contract nào là nguồn chuẩn, thay đổi cần test gì và giới hạn production nằm ở đâu. TaskFlow cung cấp dự án nhỏ nhưng hoàn chỉnh để thực hành toàn bộ chuỗi này.

## Người sử dụng

| Vai trò | Mục tiêu |
|---|---|
| Backend Engineer | Sửa API an toàn, hiểu transaction và persistence |
| QA Engineer | Thiết kế contract test và regression test |
| DevOps/SRE | Chạy và chẩn đoán service |
| Security Reviewer | Xác định trust boundary và lỗ hổng |
| Tech Lead | Review compatibility và quyết định kiến trúc |
| Chatbot | Trả lời có nguồn từ code và tài liệu |

## Hành trình nghiệp vụ

1. Tạo công việc với title, description và priority.
2. Lọc danh sách theo `is_done`, `priority`.
3. Xem một công việc theo ID.
4. Cập nhật một phần mà không ghi đè field bị bỏ qua.
5. Đánh dấu hoàn thành.
6. Xóa vĩnh viễn công việc.

## Quy tắc

- `title` bắt buộc, dài 1–200 ký tự.
- `priority` chỉ nhận `low`, `medium`, `high`; mặc định `medium`.
- Todo mới mặc định chưa hoàn thành.
- PATCH chỉ đổi field client gửi.
- ID không tồn tại trả `404`.
- Delete là hard delete.
- Danh sách mới nhất trước và chưa phân trang.

## Ngoài phạm vi

Chưa có user, auth, role, tenant, owner, deadline, tag, project, comment, notification, queue, migration, metrics hoặc audit log. Đề xuất tương lai không được mô tả như tính năng đã có.

## Tiêu chí onboarding

Kỹ sư mới phải tự chạy dự án, giải thích request flow, tìm đúng file sửa, thêm thay đổi có test, mô tả đúng rủi ro bảo mật và xử lý lỗi phổ biến.

## Thuật ngữ

| Thuật ngữ | Ý nghĩa |
|---|---|
| Todo | Entity công việc trong database |
| Contract | Input, output và lỗi HTTP client quan sát |
| Schema | Model Pydantic, trừ khi nói database schema |
| Session | Đơn vị làm việc SQLAlchemy theo request |
| Grounded answer | Câu trả lời truy ngược được về source |

