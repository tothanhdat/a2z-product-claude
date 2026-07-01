# Ticket — Human Input – Chat UI (Trigger)

## Epic Summary

**Epic Name:** Human Input – Chat UI

**Goal:**
Cho phép builder embed một giao diện chat vào workflow để nhận tin nhắn từ end-user, kích hoạt workflow tự động mỗi khi có tin nhắn mới gửi đến.

**Context:**
Thuộc nhóm Human Input triggers, song song với "Web Form". Trigger này tạo Chat URL (Draft/Published) để phân phối cho end-user — URL logic kế thừa hoàn toàn từ Human Input Web Form đã phát triển. Configure tab có hai field cần cấu hình: Directive (tin nhắn mở đầu hiển thị trong chat) và Bot Name (tên hiển thị của bot trong chat UI).

**Expected UI Fields (Configure tab):**
- Chat URL (read-only, Draft/Published toggle, Copy button)
- Directive
- Bot Name *

**Expected UI Fields (Advanced tab):**
- Authentication (Basic Auth)

---

## User Stories

### Story 1: Select trigger type

**Story:** As an automation workflow builder, I want to select "Chat UI" as the trigger type, so that my workflow is set up to receive chat messages from end-users.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Trigger * displays "Chat UI" as pre-selected value**
Given user thêm Human Input node vào workflow,
When user mở Connect step,
Then field "Trigger *" hiển thị "Chat UI" đã pre-selected,
And user có thể thay đổi sang trigger type khác nếu cần

---

### Story 2: View and copy Chat URL

**Story:** As an automation workflow builder, I want to view and copy the Chat URL (Draft and Published), so that I can distribute the correct link to end-users or use it for testing.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Chat URL section — Draft tab active by default**
Given user mở tab Configure,
When Chat URL section được render,
Then tab "Draft" được chọn mặc định,
And hệ thống hiển thị Draft URL trong ô read-only bên dưới,
And kèm nút "Copy" để sao chép URL vào clipboard

**AC.02. Switch to Published URL**
Given user đang xem Chat URL section,
When user click tab "Published",
Then hệ thống hiển thị Published URL,
And nút "Copy" cập nhật để copy Published URL

**AC.03. Copy button copies correct URL per active tab**
Given user mở Chat URL section,
When user click nút "Copy" theo từng tab:

| Active Tab | URL được copy vào clipboard |
|---|---|
| Draft | Draft Chat URL |
| Published | Published Chat URL |

Then URL tương ứng được copy vào clipboard,
And nút "Copy" đổi sang trạng thái "Copied!" để xác nhận

**AC.04. End-user sends message via Published URL — trigger fires**
Given workflow đã publish và đang active,
When end-user gửi tin nhắn qua Published Chat URL,
Then trigger fire và workflow execution bắt đầu,
And Output Payload chứa các fields tại mục Outputs

---

### Story 3: Configure Directive and Bot Name

**Story:** As an automation workflow builder, I want to configure the bot's opening message and display name, so that end-users have a clear and personalized chat experience.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Directive — optional field with default value**
Given user mở tab Configure,
When Directive field được render,
Then field hiển thị default value: "Hello! How can I help you today?",
And field là optional (không có dấu `*`),
And field không cho phép kéo expression (Static Value only)

**AC.02. Bot Name * — required, cannot continue if empty**
Given field "Bot Name *" đang trống sau khi user đã touched,
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Bot Name is required."

**AC.03. Bot Name — invalid character validation**
Given user nhập ký tự không hợp lệ vào "Bot Name *" (ký tự đặc biệt ngoài letters, numbers, spaces, hyphens, underscores),
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Bot Name contains invalid characters. Please use only letters, numbers, space, hyphens, and underscores."
*(Danh sách ký tự hợp lệ / không hợp lệ follow đúng File Name rule tại MART-1712)*

**AC.04. Bot Name — max length validation**
Given user nhập hơn 100 ký tự vào "Bot Name *",
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Bot Name must be 100 characters or less."
*(Max length follow File Name rule tại MART-1712)*

**AC.05. Bot Name — valid input allows continuing**
Given user nhập Bot Name hợp lệ (≤ 100 ký tự, ký tự cho phép),
Then the "Continue" button is enabled,
And không có validation error hiển thị

**AC.06. Directive empty — chat UI shows no opening message**
Given user để trống Directive field,
When end-user mở Chat URL,
Then chat UI hiển thị ô nhập tin nhắn mà không có opening message

**Edge Cases:**
- Bot Name chỉ chứa spaces: coi là trống → hiển thị "Bot Name is required."

---

### Story 4: Test Chat UI

**Story:** As an automation workflow builder, I want to open the Draft Chat URL directly from the Test step, so that I can interact with the chat UI and verify the trigger works before publishing.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Test/Retest button opens Draft Chat URL in new tab**
Given user đang ở tab Test,
When user click nút "Test" hoặc "Retest",
Then hệ thống mở Draft Chat URL ở tab mới trên browser,
And builder có thể tương tác với chat UI để kiểm tra trigger hoạt động đúng

**AC.02. Builder sends message via Draft URL — trigger fires in test mode**
Given builder mở Draft Chat URL (từ tab Test hoặc trực tiếp),
When builder gửi tin nhắn,
Then chat UI hiển thị tin nhắn builder vừa gửi,
And kết quả test xuất hiện trong Test panel, hiển thị Output Payload gồm các fields tại mục Outputs

---

### Story 5: Configure Basic Auth for Chat URL

**Story:** As an automation workflow builder, I want to protect the Chat URL with Basic Auth, so that only authorized users can access the chat.

**Priority:** Should-have

**Acceptance Criteria:**

Basic Auth configuration cho Chat UI follows toàn bộ spec và behavior đã định nghĩa tại **MART-1937: Human Input – Web Form – Bổ sung Authentication (Basic Auth)**.

---

### Story 6: Handle trigger errors

**Story:** As an automation workflow builder, I want clear error messages when the Chat UI trigger encounters issues, so that I can quickly identify and resolve the problem.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Workflow is paused or inactive**
Given workflow đang ở trạng thái inactive hoặc bị pause,
When end-user gửi tin nhắn qua Published Chat URL,
Then chat UI hiển thị: "This chat is currently unavailable. Please try again later.",
And trigger không fire workflow execution

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| message | string | Nội dung tin nhắn từ end-user | No |
| timestamp | string | Thời điểm nhận tin nhắn (ISO 8601) | No |
| trigger_mode | string | `"instant"` | No |

---

## Business Rules

1. Chat URL (Draft/Published) dùng lại logic URL generation và lifecycle hiện có của Human Input Web Form.
2. Directive là optional — nếu để trống, chat UI vẫn hiển thị ô nhập tin nhắn mà không có opening message.
3. Bot Name validation theo file name standard (MART-1712): max 100 ký tự, chỉ letters/numbers/spaces/hyphens/underscores được phép.
4. Tất cả fields trong Trigger không support expression (Static Value only).
5. Nút "Open Chat" trong Configure tab không phát triển.
6. Khi bấm Test/Retest ở tab Test, hệ thống mở Draft URL ở tab mới — behavior giống Web Form.
7. Basic Auth configuration dùng lại toàn bộ logic đã định nghĩa tại MART-1937.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Bot Name rỗng | Config-time, real-time | "Bot Name is required." |
| Bot Name chứa ký tự không hợp lệ | Config-time, real-time | "Bot Name contains invalid characters. Please use only letters, numbers, space, hyphens, and underscores." |
| Bot Name vượt 100 ký tự | Config-time, real-time | "Bot Name must be 100 characters or less." |
| Workflow inactive, end-user gửi tin nhắn | Runtime | "This chat is currently unavailable. Please try again later." |

---

## Out of Scope

- **Nút "Open Chat" trong Configure tab** — không phát triển trong ticket này.
- **Custom chat UI styling** (màu sắc, logo, font, theme) — defer sang phase sau.
- **Custom Auth methods** (OAuth, API Key, JWT, v.v.) — ngoài scope; chỉ support Basic Auth theo MART-1937.
- **Response Mode cho Chat UI** — nếu cần cấu hình phản hồi sau khi workflow chạy xong, tách ticket riêng.
- **File attachment trong chat** — end-user gửi file — ngoài scope ticket này.
