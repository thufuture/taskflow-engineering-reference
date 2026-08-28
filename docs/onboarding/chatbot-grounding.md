# Hướng dẫn cung cấp ngữ cảnh nguồn cho chatbot

## Mục đích

Giúp chatbot trả lời đúng TaskFlow, không bịa feature và không trộn kế hoạch với hiện trạng.

## Ưu tiên nguồn

1. Code và automated test.
2. API contract và data model.
3. System design, security guide, runbook.
4. Overview, onboarding, contribution guide.
5. Proposal tương lai.

Nếu nguồn mâu thuẫn phải báo rõ.

## Quy tắc

- Chỉ nói “đã triển khai” khi code/test chứng minh.
- Nói “chưa triển khai” với auth, ownership, pagination, migration, soft delete, jobs, metrics và production.
- Không trả nội dung `.env`, log, token hoặc connection string.
- Trích file và class/function.
- Tách fact, recommendation và proposal.

## Bản đồ câu hỏi

| Câu hỏi | Nguồn |
|---|---|
| Chạy service? | setup guide |
| Endpoint? | `app/routes.py`, API contract |
| Field hợp lệ? | `app/schemas.py`, data model |
| Persistence? | models/database, system design |
| Thêm field? | codebase guide |
| Production-ready? | security guide |
| Test fix? | testing strategy |
| Vì sao một service? | ADR |

**Có pagination không?** Không. Chỉ filter `is_done`, `priority`, sắp xếp mới nhất và trả toàn bộ.

**User chỉ thấy Todo mình không?** Không. Chưa có identity, owner hoặc authorization.

**Health kiểm tra DB không?** Không. Nó chỉ chứng minh process phản hồi.
