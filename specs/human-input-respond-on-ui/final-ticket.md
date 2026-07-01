# Ticket Title: Action - Human Input - Respond on UI

## Epic Summary

**Epic Name:** Human Input – Respond on UI

**Goal:**
Cho phép automation workflow builder gửi phản hồi (text và/hoặc file đính kèm) về Chat UI cho end-user, hoàn thành vòng tương tác hai chiều giữa hệ thống và người dùng trong workflow có human-in-the-loop.

**Context:**
Action này chỉ có ý nghĩa khi trigger của workflow là Human Input – Chat UI. Nếu trigger Chat UI nhận tin nhắn từ end-user để hệ thống xử lý, thì Respond on UI là node trả kết quả/phản hồi lại end-user trên cùng chat session đó. Text hỗ trợ markdown content. Attachment cho phép kéo binary file từ step trước hoặc nhập URL/ID (logic tương tự node Google Drive – Download File).

**Expected UI Fields:**
- Text
- Attachment

---

## User Stories

### Story 1: Connect and select action

**Story:** As an automation workflow builder, I want to confirm the action type, so that the correct action is selected for responding to the Chat UI.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Action * displays "Respond on UI" as pre-selected value**
Given user đang cấu hình node này,
When user xem field "Action *",
Then dropdown hiển thị "Respond on UI" đã được pre-selected,
And user có thể đổi sang action khác nếu cần

**Edge Cases:**
- Workflow không có trigger Human Input – Chat UI → node Respond on UI vẫn cho user setup bình thường nhưng sẽ không có ý nghĩa khi execute (không có chat session để gửi response).

---

### Story 2: Configure text response

**Story:** As an automation workflow builder, I want to compose a text response with markdown support, so that I can send formatted messages back to the end-user on the Chat UI.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Text field hiển thị dạng textarea trên Configure step**
Given user mở Configure step,
When UI render,
Then field "Text" hiển thị dạng textarea

**AC.02. Text là optional**
Given field "Text" đang trống,
And field "Attachment" cũng trống hoặc có giá trị,
When user click Continue,
Then hệ thống cho phép tiếp tục mà không báo lỗi validation cho field Text

**AC.03. Text hỗ trợ markdown content**
Given user nhập nội dung có markdown syntax (bold, italic, links, lists, code blocks),
When action chạy và gửi response về Chat UI,
Then hệ thống gửi nguyên nội dung markdown về Chat UI mà không strip formatting

**AC.04. Text support dynamic variable**
Given upstream step trả về giá trị text,
When user kéo expression từ upstream step vào field "Text",
Then hệ thống lưu cấu hình để resolve giá trị tại runtime

**AC.05. Text hỗ trợ multiline**
Given user nhập text có nhiều dòng,
When action chạy,
Then hệ thống gửi đúng multiline content về Chat UI

**Edge Cases:**
- Text resolve từ variable thành empty string → action vẫn chạy nếu Attachment có giá trị; nếu cả hai đều empty thì xem AC tại Story 4.

---

### Story 3: Configure attachment

**Story:** As an automation workflow builder, I want to attach a file to the response by dragging a binary from an upstream step or entering a URL/ID, so that I can send files along with text to the end-user.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Attachment field hiển thị trên Configure step**
Given user mở Configure step,
When UI render,
Then field "Attachment" hiển thị

**AC.02. Attachment là optional**
Given field "Attachment" đang trống,
When user click Continue,
Then hệ thống cho phép tiếp tục mà không báo lỗi validation cho field Attachment

**AC.03. Attachment hỗ trợ kéo binary file từ upstream step**
Given upstream step trả về binary output,
When user kéo expression binary từ upstream step vào field "Attachment",
Then hệ thống lưu cấu hình để resolve binary file tại runtime

**AC.04. Attachment hỗ trợ nhập URL hoặc ID**
Given user muốn đính kèm file từ cloud storage,
When user nhập URL hoặc ID vào field "Attachment",
Then logic xử lý tương tự node Google Drive – Download File

**AC.05. Chỉ hỗ trợ 1 attachment**
Given user đã cấu hình 1 attachment,
When user thử thêm attachment thứ hai,
Then hệ thống không cho phép thêm

**Edge Cases:**
- Binary expression resolve thành file rỗng (0 bytes) hoặc corrupted → action fail với error tương ứng.
- URL/ID không hợp lệ hoặc file không tồn tại → action fail với error tương ứng.
- File vượt quá system limit (25 MB) → action fail: "Attachment cannot exceed 25 MB."

---

### Story 4: Execute response and return output

**Story:** As an automation workflow builder, I want the action to send the configured response to the Chat UI and return a clear result, so that downstream steps can verify the response was delivered.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Gửi response thành công với text và/hoặc attachment**
Given user đã cấu hình Text và/hoặc Attachment,
When workflow chạy và resolve values thành công,
Then action gửi response về đúng chat session của trigger Human Input – Chat UI,
And end-user nhận được message trên Chat UI

**AC.02. Cả Text và Attachment đều empty tại runtime**
Given cả "Text" và "Attachment" đều trống hoặc resolve thành empty value tại runtime,
When action chạy,
Then action fail: "At least one of Text or Attachment must have a value."

**AC.03. Return output payload**
Given action gửi response thành công,
When execution hoàn tất,
Then action trả Output Payload gồm các fields tại mục Output

**Edge Cases:**
- Chat session đã bị đóng hoặc hết hạn khi action chạy → action fail: "Chat session is no longer active. The end-user may have disconnected."

---

## Out of Scope

- **Nhiều attachments trong 1 response** — chỉ hỗ trợ 1 attachment per execution.
- **Rich UI elements** (buttons, carousels, quick replies) — chỉ hỗ trợ text + file attachment.
- **Typing indicator / read receipts** — không track trạng thái đã đọc của end-user.
- **Respond on UI cho trigger khác** — chỉ hoạt động với Human Input – Chat UI trigger.
- **Giới hạn ký tự Text** — tạm thời không giới hạn.
- **File picker trực tiếp trong node** — user phải kéo binary từ upstream hoặc nhập URL/ID.
- **Markdown rendering behavior** — thuộc scope Chat UI, không phải node này.

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| success | boolean | Whether action succeeded | No |
| message_id | string | ID của message đã gửi trên Chat UI | No |
| text_sent | string | Nội dung text đã gửi | Yes |
| has_attachment | boolean | Có đính kèm file hay không | No |
| attachment_file_name | string | Tên file đính kèm nếu có | Yes |
| attachment_file_size | number | Kích thước file đính kèm (bytes) nếu có | Yes |
| action | string | Fixed: `respond_on_ui` | No |
| timestamp | string | Thời điểm gửi response theo ISO 8601 | No |

---

## Business Rules

1. Action chỉ có ý nghĩa khi trigger của workflow là Human Input – Chat UI. Tuy nhiên vẫn cho user setup bình thường kể cả khi workflow dùng trigger khác.
3. Text và Attachment đều optional, nhưng ít nhất 1 trong 2 phải có giá trị tại runtime.
4. Text hỗ trợ markdown content và multiline.
5. Text và Attachment đều support dynamic variable (expression từ upstream step).
6. Chỉ hỗ trợ 1 attachment per execution.
7. Attachment logic (URL/ID input) tương tự node Google Drive – Download File.
8. File size limit theo system limit (25 MB).
9. Tạm thời không giới hạn ký tự cho field Text.
10. Response được gửi về đúng chat session mà trigger đã nhận message.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Cả Text và Attachment đều empty tại runtime | Runtime | `At least one of Text or Attachment must have a value.` |
| Chat session đã đóng hoặc expired | Runtime | `Chat session is no longer active. The end-user may have disconnected.` |
| Attachment file corrupted | Runtime | `Attachment appears to be corrupted. Please upload a valid file.` |
| Attachment file vượt quá 25 MB | Runtime | `Attachment cannot exceed 25 MB.` |
| Attachment URL/ID không hợp lệ | Runtime | `Attachment resolved to an invalid value. Check your variable mapping.` |
| Binary expression resolve empty | Runtime | `Attachment resolved to an empty value. Check your variable mapping.` |
| Failed to send response | Runtime | `Failed to send response to Chat UI. Please try again.` |
