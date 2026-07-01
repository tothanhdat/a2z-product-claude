# Action - Google Chat - Send a Message

## Epic Summary

**Epic Name:** Google Chat – Send a Message

**Goal:**
Cho phép automation workflow builder gửi tin nhắn đến một space hoặc direct conversation trên Google Chat, phục vụ các use case như thông báo tự động, cập nhật trạng thái, hoặc gửi kết quả xử lý từ workflow đến team.

**Context:**
Đây là action đầu tiên của Google Chat integration trên platform. Connection sử dụng OAuth và dùng lại logic Connected Account hiện có của hệ thống. Action chỉ gửi plain text message, không hỗ trợ rich text formatting. Space dropdown load danh sách spaces (bao gồm group spaces và direct messages) từ Google Chat API.

**Expected UI Fields:**
- Account *
- Action *
- Space *
- Message *

---

## User Stories

### Story 1: Connect account and select action

**Story:** As an automation workflow builder, I want to connect an account and confirm the action type, so that the Send a Message action runs under the correct authenticated identity.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Account * is required — cannot continue if not selected**
Given the Connect step is open,
When the "Account *" field has no account selected,
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Account is required."

**AC.02. User selects or connects an account via the "Select" button**
Given the user clicks the "Select" button,
Then the system displays a list of already-connected accounts,
And the user can select an account from the list,
And if no account exists, the system redirects to the OAuth flow to connect a new one

**AC.03. Action * displays "Send a Message" as pre-selected value**
Given the user is configuring this node,
When the user views the "Action *" field,
Then the dropdown shows "Send a Message" already pre-selected,
And the user can change to other available actions if needed

---

### Story 2: Configure space and message

**Story:** As an automation workflow builder, I want to select a target space and compose a message, so that the action sends the correct content to the right destination.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Space dropdown load danh sách spaces từ connected account**
Given user đã connect account thành công và mở Configure step,
When user mở dropdown "Space *",
Then hệ thống load danh sách spaces (bao gồm group spaces và direct conversations) từ Google Chat API qua account đã connect

**AC.02. Space là required — không có default value**
Given the Configure step is open,
When the "Space *" field has no space selected,
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Space is required."

**AC.03. Space không support expression**
Given field "Space *" là dropdown,
Then field này không hỗ trợ expression/dynamic variable,
And user chỉ có thể chọn từ danh sách được load

**AC.04. Message là required field**
Given field "Message *" đang trống,
When user đã touch field và để trống,
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Message is required."

**AC.05. Message support expression**
Given upstream step trả về giá trị text,
When user kéo expression từ upstream step vào field "Message *",
Then hệ thống lưu cấu hình để resolve giá trị tại runtime

**AC.06. Message không được vượt quá 4,096 characters (config-time)**
Given user nhập văn bản trực tiếp vào field "Message *",
When nội dung vượt quá 4,096 characters,
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Message must be 4096 characters or less."

**AC.07. Message hỗ trợ multiline**
Given user nhập text có nhiều dòng vào field "Message *",
When action chạy,
Then hệ thống gửi đúng nội dung multiline đến Google Chat

**Edge Cases:**
- Space đã chọn bị deleted hoặc user mất quyền access sau config → action fail tại runtime.
- Dropdown Space trả về empty list (account không có space nào) → hiển thị empty state, không block UI.

---

### Story 3: Execute send message and return output

**Story:** As an automation workflow builder, I want the action to send the message and return a clear result, so that downstream steps can confirm the message was sent and reference it.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Send message successfully**
Given Space và Message đã được cấu hình hợp lệ,
And connected account có quyền gửi message trong space đã chọn,
When workflow chạy,
Then hệ thống gửi plain text message đến space đã chọn trên Google Chat

**AC.02. Return output payload**
Given action gửi message thành công,
When execution hoàn tất,
Then action trả Output Payload gồm các fields tại mục Output

**AC.03. Message resolved to empty value at runtime**
Given field "Message *" được map từ variable,
When action chạy và resolved value là empty string hoặc null,
Then action fail: "Message resolved to an empty value. Check your variable mapping."

**AC.04. Mapped Message vượt quá giới hạn tại runtime**
Given field "Message *" được map từ variable,
When action chạy và resolved value vượt quá 4,096 characters,
Then action fail: "Message must be 4096 characters or less."

**Edge Cases:**
- Provider rate limit, timeout, hoặc temporary failure → Dùng lại logic retry của hệ thống.

---

### Story 4: Handle authentication and permission errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. OAuth token expired or revoked**
Given the connected account's OAuth token has expired or been revoked,
When the action attempts to execute,
Then the action fails with: "Authentication failed. Please reconnect your Google Chat account.",
And the workflow execution is marked as failed

**AC.02. Insufficient permission on resource**
Given the connected account does not have required access to the space,
When the action runs,
Then the action surfaces: "Cannot send message: the connected account does not have write access to this space. Check the sharing settings.",
And the space name and account email are included in the error detail

**AC.03. Resource not found**
Given the space ID does not exist or has been deleted,
When the action runs,
Then the action surfaces: "Space not found. It may have been deleted or moved.",
And the workflow execution is marked as failed

---

## Out of Scope

- **Rich text / card formatting** — chỉ hỗ trợ plain text trong phiên bản này.
- **Reply to thread** — gửi message vào thread cụ thể sẽ được xem xét ở phiên bản sau.
- **Create new direct message** — chỉ gửi đến spaces đã có trong dropdown, không tạo DM mới.
- **Send attachment / file** — gửi file kèm message.
- **Bulk send** — gửi cùng message đến nhiều spaces.
- **Message scheduling** — hẹn giờ gửi message.

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| success | boolean | Whether action succeeded | No |
| message_id | string | ID của message đã gửi | No |
| space_name | string | Tên space đã gửi message đến | No |
| sender | string | Email của account đã gửi message | No |
| message_text | string | Nội dung message đã gửi | No |
| created_at | string | Thời điểm message được tạo theo ISO 8601 | No |
| action | string | Fixed: `send_message` | No |
| timestamp | string | Execution timestamp ISO 8601 | No |

---

## Business Rules

1. Action chỉ gửi plain text message, không hỗ trợ rich text formatting.
2. Space là required, không có default value — user phải chọn trước khi Continue.
3. Space dropdown load danh sách spaces (bao gồm group spaces và direct conversations) từ Google Chat API.
4. Space không support expression — chỉ chọn từ dropdown.
5. Message là required, support cả manual input và expression.
6. Message max 4,096 characters — validate config-time cho static input, runtime cho expression.
7. Message hỗ trợ multiline.
8. Connection dùng OAuth, dùng lại logic Connected Account hiện có của hệ thống.
9. Dùng lại logic retry của hệ thống.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Account chưa được chọn | Config-time | `Account is required.` |
| Space chưa được chọn | Config-time | `Space is required.` |
| Message để trống | Config-time | `Message is required.` |
| Message vượt quá 4,096 characters (static hoặc resolved) | Config-time / Runtime | `Message must be 4096 characters or less.` |
| Message expression resolve empty | Runtime | `Message resolved to an empty value. Check your variable mapping.` |
| OAuth token expired/revoked | Runtime | `Authentication failed. Please reconnect your Google Chat account.` |
| Không đủ quyền gửi message | Runtime | `Cannot send message: the connected account does not have write access to this space. Check the sharing settings.` |
| Space không tồn tại hoặc đã bị xoá (bao gồm selected space bị xoá sau config) | Runtime | `Space not found. It may have been deleted or moved.` |
| Space dropdown trả về empty list | Config-time | `No spaces available. Please check your connection or permissions.` |
| Google Chat API unavailable | Runtime | `Google Chat is currently unavailable. Please try again later.` |
| Rate limit / timeout | Runtime | Dùng lại logic retry của hệ thống. |
