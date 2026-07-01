# Ticket — Human Input – Chat UI (Trigger) — v2

## Epic Summary

**Epic Name:** Human Input – Chat UI

**Goal:**
Cho phép builder embed một giao diện chat vào workflow để nhận tin nhắn từ end-user, kích hoạt workflow tự động mỗi khi có tin nhắn mới gửi đến.

**Context:**
Thuộc nhóm Human Input triggers, song song với "Web Form". Trigger này tạo Published Chat URL để phân phối cho end-user — URL logic kế thừa từ Human Input Web Form đã phát triển. Chat Panel được dùng ở hai context: (1) Test trigger node — builder gửi 1 tin nhắn để verify trigger output, chỉ execute trigger node; (2) "Open Chat" ở workflow level — builder chat tự do, execute toàn bộ workflow, response từ action "Respond on UI" (ticket riêng) hiển thị trong chat. Configure tab có hai field cần cấu hình: Directive (tin nhắn mở đầu hiển thị trong chat) và Bot Name (tên hiển thị của bot trong chat UI).

**Expected UI Fields (Configure tab):**
- Chat URL (read-only, Published URL, Copy button)
- Directive
- Bot Name *
- Add Respond on UI step (button → dialog)

**Expected UI Fields (Advanced tab):**
- Authentication (Basic Auth)

**Changes from v1:**
- Bỏ Draft URL — test qua Chat Panel thay vì mở Draft URL ở tab mới
- Test trigger: chỉ execute trigger node, gửi 1 tin nhắn rồi đóng Chat Panel, output hiển thị ở Tab Output
- Open Chat (workflow level): nút "Test Run" đổi thành "Open Chat" khi trigger là Chat UI, execute toàn bộ workflow
- Session management (Open Chat): mỗi lần mở = phiên mới, các message trong phiên chia sẻ cùng session ID

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

### Story 2: View and copy Published Chat URL

**Story:** As an automation workflow builder, I want to view and copy the Published Chat URL, so that I can distribute the link to end-users when the workflow is live.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Published Chat URL hiển thị trên Configure tab**
Given user mở tab Configure,
When Chat URL section được render,
Then hệ thống hiển thị Published URL trong ô read-only kèm mô tả "Published Chat URL used after publishing the workflow",
And kèm nút "Copy" để sao chép URL vào clipboard

**AC.02. Copy button copies Published URL**
Given user đang xem Chat URL section,
When user click nút "Copy",
Then Published URL được copy vào clipboard,
And nút "Copy" đổi sang trạng thái "Copied!" để xác nhận

**AC.03. Workflow not published — Published URL shows error**
Given workflow chưa được publish,
When end-user mở Published Chat URL,
Then chat UI hiển thị: "This chat is not yet available. The workflow has not been published.",
And end-user không thể gửi tin nhắn

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

**AC.07. "Add Respond on UI step" button hiển thị dưới Bot Name**
Given user mở tab Configure,
When UI render,
Then nút "Add Respond on UI step" hiển thị phía dưới field "Bot Name *"

**AC.08. Click "Add Respond on UI step" — dialog hiển thị**
Given user click nút "Add Respond on UI step",
Then hệ thống hiển thị dialog với nội dung:
- Title: "Add Respond on UI step"
- Description: "Without a Respond on UI step, your chat workflow can receive messages but can't reply. Add one to send text, files, or markdown back to the user."
- Visual: minh hoạ workflow gồm Human Input (Chat UI) → Human Input (Respond on UI)
- Helper text: "A new Respond on UI step will be added and connected right after this trigger."
- Buttons: "Cancel" và "Add Respond on UI step"

**AC.09. Confirm add — Respond on UI node added to workflow**
Given dialog "Add Respond on UI step" đang hiển thị,
When user click nút "Add Respond on UI step",
Then hệ thống thêm node "Respond on UI" vào workflow ngay sau trigger node hiện tại,
And node mới được tự động connect với trigger node,
And dialog đóng lại

**AC.10. Cancel — dialog closes, no change**
Given dialog "Add Respond on UI step" đang hiển thị,
When user click nút "Cancel" hoặc nhấn X,
Then dialog đóng lại,
And workflow không thay đổi

**Edge Cases:**
- Bot Name chỉ chứa spaces: coi là trống → hiển thị "Bot Name is required."

---

### Story 4: Test trigger via Chat Panel

**Story:** As an automation workflow builder, I want to test the Chat UI trigger by sending a message through a Chat Panel, so that I can verify the trigger captures input correctly before building the rest of the workflow.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Test/Retest button opens Chat Panel**
Given user đang ở tab Test của trigger node,
When user click nút "Test" hoặc "Retest",
Then hệ thống mở Chat Panel ngay trong editor,
And Chat Panel hiển thị giao diện chat (Bot Name, Directive nếu có)

**AC.02. Builder sends message — trigger executes and Chat Panel closes**
Given builder đang ở Chat Panel,
When builder nhập và gửi 1 tin nhắn,
Then hệ thống execute trigger node (chỉ trigger, không chạy các step tiếp theo trong workflow),
And Chat Panel đóng ngay sau khi nhận tin nhắn,
And Tab Output hiển thị Output Payload gồm các fields tại mục Outputs

**AC.03. Retest opens new Chat Panel**
Given Tab Output đang hiển thị kết quả test trước đó,
When builder click nút "Retest",
Then hệ thống mở Chat Panel mới,
And builder có thể gửi tin nhắn mới để test lại

---

### Story 5: Open Chat — execute workflow via Chat Panel

**Story:** As an automation workflow builder, I want to open a Chat Panel from the workflow level to test the entire workflow end-to-end, so that I can verify the full chat experience including responses from downstream nodes.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. "Test Run" button displays as "Open Chat" when trigger is Chat UI**
Given workflow có trigger là "Chat UI",
When builder xem nút "Test Run" (nút execute workflow đã có sẵn),
Then nút hiển thị label "Open Chat" thay vì "Test Run"

**AC.02. Open Chat opens Chat Panel with new session**
Given builder click nút "Open Chat",
Then hệ thống mở Chat Panel ngay trong editor,
And Chat Panel hiển thị giao diện chat (Bot Name, Directive nếu có),
And hệ thống khởi tạo một phiên chat mới với session ID mới

**AC.03. Builder sends message — workflow executes end-to-end**
Given builder đang ở Chat Panel (workflow có node "Respond on UI"),
When builder nhập và gửi tin nhắn,
Then chat UI hiển thị tin nhắn builder vừa gửi,
And hệ thống execute toàn bộ workflow (real execution),
And response từ node "Respond on UI" hiển thị trong Chat Panel dưới dạng bot message

**AC.04. Typing indicator while workflow is executing**
Given builder đã gửi tin nhắn trong Chat Panel,
When workflow đang execute,
Then chat UI hiển thị typing indicator (dưới dạng bot đang nhập),
And typing indicator biến mất khi nhận được response từ node "Respond on UI" hoặc khi workflow kết thúc (không có response node)

**AC.05. Response contains attachment — click to download**
Given node "Respond on UI" trả về response có đính kèm file,
When file hiển thị trong Chat Panel và user click vào file,
Then hệ thống download file về device của user

**AC.06. Multiple messages in same session share session ID**
Given builder đã gửi tin nhắn đầu tiên trong Chat Panel,
When builder gửi thêm tin nhắn tiếp theo mà không đóng Chat Panel,
Then tin nhắn mới sử dụng cùng session ID với tin nhắn trước đó,
And mỗi tin nhắn trigger một workflow execution riêng biệt

**AC.07. Close and reopen Chat Panel — new session**
Given builder đang ở Chat Panel,
When builder đóng Chat Panel (nhấn X hoặc close),
Then phiên chat hiện tại kết thúc,
And khi builder nhấn "Open Chat" lại, hệ thống mở phiên chat mới với session ID mới

**AC.08. No response node — chat shows no bot reply**
Given workflow không có node "Respond on UI",
When builder gửi tin nhắn trong Chat Panel,
Then workflow vẫn execute bình thường,
And chat UI chỉ hiển thị tin nhắn của builder, không hiển thị bot response

**AC.09. Workflow fails — chat shows no error**
Given workflow gặp lỗi trong quá trình execute (node fail, timeout, v.v.),
When builder gửi tin nhắn trong Chat Panel,
Then chat UI không hiển thị error message,
And builder kiểm tra lỗi trong execution log

---

### Story 6: Configure Basic Auth for Chat URL

**Story:** As an automation workflow builder, I want to protect the Chat URL with Basic Auth, so that only authorized users can access the chat.

**Priority:** Should-have

**Acceptance Criteria:**

Basic Auth configuration cho Chat UI follows toàn bộ spec và behavior đã định nghĩa tại **MART-1937: Human Input – Web Form – Bổ sung Authentication (Basic Auth)**.

---

### Story 7: Handle trigger errors

**Story:** As an automation workflow builder, I want clear error messages when the Chat UI trigger encounters issues, so that I can quickly identify and resolve the problem.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Workflow is paused or inactive**
Given workflow đang ở trạng thái inactive hoặc bị pause,
When end-user gửi tin nhắn qua Published Chat URL,
Then chat UI hiển thị: "This chat is currently unavailable. Please try again later.",
And trigger không fire workflow execution

---

## Out of Scope

- **Draft URL** — không còn Draft URL; builder test qua Chat Panel.
- **Custom chat UI styling** (màu sắc, logo, font, theme) — defer sang phase sau.
- **Custom Auth methods** (OAuth, API Key, JWT, v.v.) — ngoài scope; chỉ support Basic Auth theo MART-1937.
- **File attachment trong chat** — end-user gửi file — ngoài scope ticket này.
- **Error display trong chat khi workflow fail** — builder kiểm tra lỗi trong execution log.

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| session_id | string | ID phiên chat — các tin nhắn trong cùng phiên chia sẻ cùng session ID | No |
| message | string | Nội dung tin nhắn từ end-user | No |
| timestamp | string | Thời điểm nhận tin nhắn (ISO 8601) | No |

---

## Business Rules

1. Published Chat URL dùng lại logic URL generation và lifecycle hiện có của Human Input Web Form.
2. Directive là optional — nếu để trống, chat UI vẫn hiển thị ô nhập tin nhắn mà không có opening message.
3. Bot Name validation theo file name standard (MART-1712): max 100 ký tự, chỉ letters/numbers/spaces/hyphens/underscores được phép.
4. Tất cả fields trong Trigger không support expression (Static Value only).
5. **Test trigger (Story 4):** Khi bấm Test/Retest ở tab Test của trigger node, hệ thống mở Chat Panel. Builder gửi 1 tin nhắn → chỉ execute trigger node (không chạy workflow) → đóng Chat Panel → output hiển thị ở Tab Output.
6. **Open Chat (Story 5):** Khi trigger là Chat UI, nút "Test Run" ở workflow level đổi label thành "Open Chat". Builder chat tự do, mỗi tin nhắn execute toàn bộ workflow (real execution). Side-effects của node downstream sẽ xảy ra thật.
7. Mỗi lần nhấn "Open Chat" = khởi tạo phiên chat mới với session ID mới. Các tin nhắn gửi trong cùng phiên chia sẻ cùng session ID.
8. Response trong Chat Panel (Open Chat) đến từ node "Respond on UI" — nếu workflow không có node này, chat không hiển thị response.
9. Khi workflow fail trong quá trình execute (Open Chat), Chat Panel không hiển thị error — builder kiểm tra trong execution log.
10. Basic Auth configuration dùng lại toàn bộ logic đã định nghĩa tại MART-1937.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Bot Name rỗng | Config-time, real-time | "Bot Name is required." |
| Bot Name chứa ký tự không hợp lệ | Config-time, real-time | "Bot Name contains invalid characters. Please use only letters, numbers, space, hyphens, and underscores." |
| Bot Name vượt 100 ký tự | Config-time, real-time | "Bot Name must be 100 characters or less." |
| Workflow chưa publish, end-user mở Published URL | Runtime | "This chat is not yet available. The workflow has not been published." |
| Workflow inactive, end-user gửi tin nhắn | Runtime | "This chat is currently unavailable. Please try again later." |
