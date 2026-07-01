# User Story - Webhook Trigger & VipTalk - Environment-Portable Webhook URLs

## Epic Summary

**Epic Name:** Webhook Trigger & VipTalk – Environment-Portable Webhook URLs

**Goal:**
Khi user copy workflow từ môi trường này sang môi trường khác (ví dụ: dev → staging → prod), các Webhook URL được tạo bởi Webhook Trigger và VipTalk node phải tự động hiển thị đúng domain của môi trường đích — không cần user chỉnh tay.

**Context:**
Human Input node đã áp dụng hành vi này. Webhook Trigger và VipTalk node hiện đang lưu full URL (ví dụ: `https://api-dev.a2zflow.ai/webhook/abc`), khiến URL bị sai môi trường khi workflow được copy sang môi trường khác. Ticket này align 2 node còn lại về cùng behavior với Human Input.

---

## User Stories

### Story 1: Webhook URL tự động resolve đúng môi trường

**Story:** As an automation workflow builder, I want the webhook URL shown in Webhook Trigger and VipTalk nodes to automatically reflect the current environment's domain, so that I can copy workflows between environments without manually updating URLs.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Webhook URL hiển thị đúng domain của môi trường hiện tại**
Given user đang xem cấu hình của Webhook Trigger hoặc VipTalk node,
When node được mở trong bất kỳ môi trường nào (dev / staging / prod),
Then Webhook URL hiển thị cho user phải dùng domain của môi trường đó,
And URL vẫn là full URL hợp lệ mà user có thể copy và dùng được ngay

**AC.02. Workflow được copy sang môi trường khác — URL tự cập nhật**
Given một workflow có chứa Webhook Trigger hoặc VipTalk node đã được cấu hình,
When workflow đó được copy hoặc import sang môi trường khác,
Then Webhook URL trong node tự động hiển thị đúng domain của môi trường đích,
And user không cần chỉnh sửa URL thủ công

**AC.03. Behavior của Human Input làm chuẩn**
Given Human Input node đã hoạt động đúng với environment-portable URL,
When Webhook Trigger và VipTalk được cập nhật,
Then behavior của 2 node này phải nhất quán với Human Input

---

## Out of Scope

- **Auto-migration workflow cũ** — Các workflow hiện có đang lưu full URL sẽ không được tự động migrate. User có thể dùng tính năng Replace Node để cập nhật thủ công.
- **Các node khác** — Ticket này chỉ cover Webhook Trigger và VipTalk. Các node khác (nếu có vấn đề tương tự) sẽ xử lý trong ticket riêng.
- **Thay đổi UI/UX** — Không thay đổi giao diện cấu hình node, chỉ thay đổi cách URL được resolve khi hiển thị.
