# Hotfix - Webhook Trigger & VipTalk - Migrate Existing Full URLs to Relative Path

## Epic Summary

**Epic Name:** Hotfix – Migrate Existing Webhook Full URLs to Relative Path

**Goal:**
Đảm bảo các workflow hiện có (được tạo trước khi áp dụng environment-portable URL) không bị broken sau khi hệ thống chuyển sang lưu relative path.

**Context:**
Tiếp nối ticket [webhook-viptalk-environment-portable-url]. Các workflow được tạo trước đó có Webhook Trigger hoặc VipTalk node đang lưu full URL (ví dụ: `https://api-dev.a2zflow.ai/webhook/abc`). Sau khi hệ thống cập nhật sang relative path, dữ liệu cũ cần được xử lý để đảm bảo nhất quán.

---

## User Stories

### Story 1: Existing workflows continue to work correctly after the migration

**Story:** As an automation workflow builder, I want my existing workflows with Webhook Trigger or VipTalk nodes to continue working correctly after the system update, so that I don't experience unexpected breakage or wrong webhook URLs.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Existing workflows hoạt động đúng sau khi migration**
Given một workflow đã tồn tại có chứa Webhook Trigger hoặc VipTalk node với full URL được lưu trước đây,
When migration hoàn tất,
Then workflow đó vẫn hoạt động đúng — webhook nhận request bình thường,
And Webhook URL hiển thị trong node phản ánh đúng domain của môi trường hiện tại

**AC.02. Không có workflow nào bị ảnh hưởng ngoài ý muốn**
Given toàn bộ workflow trên hệ thống,
When migration được thực thi,
Then chỉ các node bị ảnh hưởng (Webhook Trigger, VipTalk) được cập nhật,
And các node khác và cấu hình khác trong workflow không thay đổi

**AC.03. Migration có thể verify được**
Given migration đã chạy xong,
When team kiểm tra,
Then có thể xác nhận được số lượng record đã được migrate thành công,
And có thể xác nhận không còn full URL nào được lưu trong các node thuộc scope

---

## Out of Scope

- **Nodes ngoài scope** — Chỉ xử lý Webhook Trigger và VipTalk. Các node khác (nếu có vấn đề tương tự) sẽ được xử lý riêng.
- **Rollback tự động** — Không yêu cầu tự động rollback; nếu cần rollback thì do team kỹ thuật xử lý thủ công.
- **Thông báo tới user** — Không cần notify user vì behavior từ góc độ user không thay đổi.

---

## Assumptions

1. Migration được thực thi **sau** khi ticket [webhook-viptalk-environment-portable-url] đã được deploy lên môi trường tương ứng.
2. Việc xác định "full URL vs relative path" và cơ chế migrate là do Engineering tự quyết định.
