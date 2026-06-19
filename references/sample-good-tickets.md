---
inclusion: manual
---

# Sample Good Tickets

## Mục đích

File này cung cấp các ví dụ ticket chuẩn cho action/trigger trong workflow automation platform.

Mục tiêu là giúp AI và team nội bộ tạo ticket:
- ngắn gọn hơn
- rõ business behavior
- đủ testable cho Dev/QC
- không lặp lại các platform behavior đã có sẵn
- thống nhất cách viết tiếng Việt, giữ thuật ngữ English khi cần

Các ví dụ trong file này là **Lean Ticket examples**. Khi user không yêu cầu full technical specification, dùng style này làm mặc định.

> **Conventions & Rules:** Xem chi tiết tại `ticket-template-lean.md` (structure, language rules, lean rules), `error-message-catalog.md` (error messages), `input-output-conventions.md` (input/output conventions, standard reference sentences).

---

# Lean Example 1
# Ticket title: Trigger - Google Calendar - New Event

## Epic Summary

**Epic Name:** Google Calendar – New Event Trigger (Instant)

**Goal:**  
Cho phép automation workflow builder tự động kích hoạt workflow instantly (trong vài giây) khi có event mới được tạo trong Google Calendar account đã kết nối, phục vụ các use case nhạy cảm về thời gian như instant notifications, real-time CRM updates, immediate task creation, hoặc urgent team alerts.

**Context:**  
Trigger này sử dụng Google Calendar Push Notifications (webhook-based mechanism) để nhận real-time notifications từ Google khi có event mới được tạo. Connection dùng lại logic Connected Account hiện có của Google Calendar integration. Trigger chỉ fire khi event mới được tạo (created), không fire cho events đã tồn tại trước đó hoặc events được updated. Trigger hỗ trợ monitoring tất cả calendars accessible hoặc chỉ specific calendars được chọn.

**Expected UI Fields:**
- Connection *
- Calendar Selection Mode *
- Specific Calendars (conditional)
- Expand Recurring Events

---

## User Stories

### Story 1: Cấu hình connection và calendar selection mode

**Story:**  
As an automation workflow builder, I want to chọn Google Calendar connection và cấu hình calendar monitoring scope, so that trigger có thể nhận instant notifications từ calendars mong muốn.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Connection sử dụng behavior hiện có**  
Given user cấu hình trigger này, When user chọn `Connection`, Then trường này sử dụng logic Connected Account hiện có của Google Calendar integration.

**AC.02. Connection là bắt buộc**  
Given `Connection` để trống, When user click Continue hoặc Save, Then hiển thị lỗi validation: `Connection is required.`

**AC.03. Calendar Selection Mode là bắt buộc**  
Given `Calendar Selection Mode` để trống, When user click Continue hoặc Save, Then hiển thị lỗi validation: `Calendar Selection Mode is required.`

**AC.04. Các option của Calendar Selection Mode**  
Given user mở dropdown `Calendar Selection Mode`, When options được load, Then hệ thống hiển thị: `All my Calendars`, `Specific calendars`.

**AC.05. Giá trị mặc định của Calendar Selection Mode**  
Given user chưa thay đổi `Calendar Selection Mode`, When trigger được save, Then hệ thống sử dụng `All my Calendars` làm giá trị mặc định.

**AC.06. Behavior của All my Calendars**  
Given user chọn `All my Calendars`, When trigger được activate, Then hệ thống monitors tất cả accessible calendars của connected account, And excludes holiday calendars và birthday calendars.

**AC.07. Specific Calendars field visibility**  
Given user chọn `Specific calendars` trong `Calendar Selection Mode`, When UI updates, Then trường `Specific Calendars` hiển thị và là required.

**AC.08. Specific Calendars là required khi mode = Specific calendars**  
Given user chọn `Specific calendars` mode, And `Specific Calendars` để trống, When user click Continue hoặc Save, Then hiển thị lỗi validation: `Specific Calendars is required when Calendar Selection Mode is Specific calendars.`

**AC.09. Specific Calendars selector behavior**  
Given user chọn `Specific calendars` mode, When user mở `Specific Calendars` selector, Then hệ thống load danh sách accessible calendars của connected account, And cho phép user chọn một hoặc nhiều calendars.

**AC.10. Expand Recurring Events defaults to Off**  
Given user chưa thay đổi `Expand Recurring Events`, When trigger được save hoặc execute, Then hệ thống sử dụng `Off` (disabled) làm giá trị mặc định.

**AC.11. Expand Recurring Events behavior dùng lại logic hiện có**  
Given user cấu hình `Expand Recurring Events`, When trigger xử lý recurring events, Then hệ thống sử dụng logic recurring event expansion hiện có của trigger "New Event or Update", And khi enabled thì expand recurring events thành individual instances, And khi disabled thì chỉ trả về master event.

**Edge Cases:**
- Calendar được chọn trong Specific Calendars bị deleted hoặc revoked access sau config: trigger không fire cho calendar đó, ghi log warning.
- User thay đổi Calendar Selection Mode hoặc Specific Calendars khi trigger đang active: existing webhook stopped, new webhook registered với updated scope.

### Story 2: Webhook registration và instant event detection

**Story:**  
As an automation workflow builder, I want trigger tự động register webhook với Google Calendar và nhận instant notifications, so that workflow fires ngay khi có event mới được tạo.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Webhook registration khi trigger activate**  
Given trigger được activate lần đầu, When activation completes, Then hệ thống register webhook với Google Calendar API cho các calendars được monitor, And lưu webhook channel information để quản lý lifecycle.

**AC.02. Webhook receives push notifications**  
Given webhook đã được registered, When có event mới được tạo trong monitored calendar, Then Google Calendar gửi push notification đến webhook endpoint của hệ thống, And notification được nhận trong vài giây.

**AC.03. Trigger fires cho mỗi new event**  
Given webhook nhận push notification về event mới, When hệ thống xử lý notification, Then trigger fires workflow một lần cho event đó, And truyền event details vào workflow execution.

**AC.04. Chỉ fire cho created events**  
Given webhook nhận notification, When notification type là event created, Then trigger fires workflow, And khi notification type là event updated hoặc deleted, Then trigger không fire.

**AC.05. Filter theo calendar selection**  
Given user đã cấu hình `Specific calendars`, When webhook nhận notification từ calendar không nằm trong danh sách đã chọn, Then trigger không fire workflow.

**AC.06. Exclude holiday và birthday calendars**  
Given user chọn `All my Calendars` mode, When webhook nhận notification từ holiday calendar hoặc birthday calendar, Then trigger không fire workflow.

**Edge Cases:**
- Webhook nhận notification từ calendar đã bị deleted: trigger không fire, ghi log.
- Webhook nhận invalid hoặc malformed notification: notification bị reject, không fire workflow.
- Webhook nhận duplicate notifications cho cùng event: deduplication mechanism prevent duplicate workflow executions.
- Google Calendar gửi notification delay (rare case): trigger vẫn fires nhưng có thể chậm hơn vài giây/phút.

### Story 3: Webhook lifecycle management

**Story:**  
As an automation workflow builder, I want hệ thống tự động quản lý webhook lifecycle, so that trigger hoạt động ổn định và không để lại stale webhooks.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Webhook renewal trước khi expire**  
Given webhook channel có expiration time, When expiration time sắp đến (trước 24 giờ), Then hệ thống tự động renew webhook channel, And update channel information.

**AC.02. Webhook cleanup khi trigger deactivate**  
Given trigger đang active với registered webhook, When trigger bị deactivated hoặc deleted, Then hệ thống stop webhook channel với Google Calendar API, And cleanup webhook registration data.

**AC.03. Webhook re-registration khi config thay đổi**  
Given trigger đang active, When user thay đổi `Calendar Selection Mode` hoặc `Specific Calendars`, Then hệ thống stop existing webhook, And register new webhook với updated calendar scope.

**Edge Cases:**
- Webhook renewal fails: hệ thống retry renewal, nếu vẫn fail thì attempt re-register new webhook.
- Webhook cleanup fails khi trigger deactivate: hệ thống ghi log warning nhưng vẫn complete deactivation.
- Multiple config changes trong thời gian ngắn: hệ thống debounce và chỉ re-register webhook một lần với config cuối cùng.
- Webhook verification timeout: registration fails, user nhận error message yêu cầu retry.

### Story 4: Output payload cho downstream steps

**Story:**  
As an automation workflow builder, I want trigger trả về detailed event information, so that các actions downstream có thể xử lý event data một cách hiệu quả.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Output payload khi trigger fires**  
Given trigger fires cho new event, When downstream steps đọc output, Then trigger trả Output Payload gồm các fields tại mục Output.

**Edge Cases:**
- Event không có description, location, hoặc attendees: các fields tương ứng trả về null hoặc empty array.
- Event data incomplete từ Google Calendar API: trigger vẫn fires với available data, missing fields trả về null.
- Recurring event với Expand Recurring Events enabled: trigger fires multiple times (một lần cho mỗi instance).

---

## Out of Scope

- Trigger cho event updates hoặc deletions.
- Trigger cho event cancellations.
- Polling-based detection mechanism.
- Batch processing nhiều events trong một workflow execution.
- Custom webhook endpoint configuration.
- Manual webhook registration/deregistration.
- Event filtering theo event properties (title, attendees, location, etc.) ngoài calendar scope.
- Historical event processing (events created trước khi trigger activate).

---

## Inputs

| Field | Type | Required | Default | Static selection | Manual input | Mapped Variable | Notes |
|---|---|---:|---|---:|---:|---:|---|
| Connection | Connected Account | Yes | None | Yes | No | No | Dùng lại logic Connected Account hiện có. |
| Calendar Selection Mode | Dropdown | Yes | All my Calendars | Yes | No | No | Options: All my Calendars, Specific calendars. |
| Specific Calendars | Multi-select calendar picker | Conditional | None | Yes | No | No | Required khi mode = Specific calendars. Cho phép chọn một hoặc nhiều calendars. |
| Expand Recurring Events | Toggle | No | Off | Yes | No | No | Dùng lại logic recurring event expansion của trigger "New Event or Update". |

---

## Outputs

| Field | Type | Description |
|---|---|---|
| event_id | string | ID của event mới |
| calendar_id | string | ID của calendar chứa event |
| calendar_name | string | Tên calendar |
| summary | string | Tiêu đề event |
| description | string/null | Mô tả event nếu có |
| start_time | string | Thời gian bắt đầu theo ISO 8601 |
| end_time | string | Thời gian kết thúc theo ISO 8601 |
| timezone | string | Timezone của event |
| location | string/null | Địa điểm event nếu có |
| is_all_day_event | boolean | Event có phải all-day event không |
| attendees | array | Danh sách attendees với email, display_name, response_status, is_organizer |
| organizer_email | string | Email của organizer |
| organizer_name | string/null | Tên organizer nếu có |
| event_link | string | Link đến event trong Google Calendar |
| event_created_at | string | Thời điểm event được tạo theo ISO 8601 |
| is_recurring_instance | boolean | Event có phải là instance của recurring event không |
| recurrence_id | string/null | ID của recurring event master nếu event là instance |
| detected_at | string | Thời điểm trigger nhận notification theo ISO 8601 |
| trigger_mode | string | `instant` để phân biệt với polling triggers |

---

## Business Rules

1. Connection và Calendar Selection Mode là bắt buộc.
2. Calendar Selection Mode mặc định là `All my Calendars`.
3. Specific Calendars là required khi Calendar Selection Mode = `Specific calendars`.
4. Khi mode = `All my Calendars`, hệ thống monitors tất cả accessible calendars ngoại trừ holiday calendars và birthday calendars.
5. Khi mode = `Specific calendars`, hệ thống chỉ monitors các calendars được chọn trong `Specific Calendars`.
6. Trigger sử dụng Google Calendar Push Notifications (webhook) để nhận instant notifications.
7. Trigger chỉ fires cho events mới được tạo (created), không fires cho events được updated hoặc deleted.
8. Trigger fires một lần cho mỗi new event.
9. Webhook channel được tự động registered khi trigger activate.
10. Webhook channel được tự động renewed trước khi expire (trước 24 giờ).
11. Webhook channel được tự động stopped và cleaned up khi trigger deactivate hoặc deleted.
12. Khi Calendar Selection Mode hoặc Specific Calendars thay đổi, existing webhook được stopped và new webhook được registered với updated scope.
13. Trigger không fires cho events được tạo trước khi trigger activate (không process historical events).
14. Holiday calendars và birthday calendars được xác định dựa trên calendar metadata từ Google Calendar API.
15. Webhook verification requests từ Google Calendar phải được respond correctly để complete registration.
16. Notification latency thường trong vài giây (instant), nhưng có thể delay trong rare cases do Google infrastructure.
17. Expand Recurring Events mặc định là `Off` (disabled).
18. Expand Recurring Events behavior dùng lại logic hiện có của trigger "New Event or Update":
    - Khi enabled: recurring events được expand thành individual instances, mỗi instance fires workflow riêng biệt.
    - Khi disabled: chỉ master event của recurring series được return, không expand instances.
19. Khi Expand Recurring Events enabled và có recurring event mới được tạo, trigger fires cho master event và tất cả instances trong series.

---

## Error Handling

| Scenario | Behavior / Message |
|---|---|
| Connection trống | `Connection is required.` |
| Calendar Selection Mode trống | `Calendar Selection Mode is required.` |
| Specific Calendars trống khi mode = Specific calendars | `Specific Calendars is required when Calendar Selection Mode is Specific calendars.` |
| Webhook registration fails | `Failed to register webhook with Google Calendar. Please check your connection and try again.` |
| Webhook verification fails | `Webhook verification failed. Please reconnect your Google account and try again.` |
| Lỗi authentication | `Google authentication failed. Please reconnect your Google account.` |
| Không đủ quyền | `The connected account does not have sufficient rights to access calendar events.` |
| Calendar không tồn tại hoặc mất quyền | `One or more selected calendars are no longer accessible. Please update your calendar selection.` |
| Webhook channel expired và renewal fails | `Webhook channel expired and renewal failed. Trigger will attempt to re-register automatically.` |
| Invalid webhook notification | Notification bị reject, không fire workflow, ghi log: `Received invalid webhook notification.` |
| Lỗi provider | `Google Calendar API returned an unexpected error. Please try again later.` |

---

## Assumptions

1. Google Calendar API Push Notifications endpoint và webhook mechanism hoạt động stable và reliable.
2. Platform có webhook infrastructure để nhận và xử lý incoming notifications từ Google Calendar.
3. Platform có mechanism để quản lý webhook lifecycle (registration, renewal, cleanup).
4. Connected Account đã được authenticate với đủ scopes để access calendar events và register webhooks.
5. Platform có khả năng detect và filter holiday calendars và birthday calendars dựa trên calendar metadata.
6. Webhook notifications từ Google Calendar bao gồm đủ thông tin để identify event type (created/updated/deleted).
7. Platform có retry và error handling mechanism cho webhook registration failures.
8. Expand Recurring Events logic đã được implement trong trigger "New Event or Update" và có thể reuse cho trigger này.

---

# Lean Example 2
# Ticket Title: Action - Google Drive - Delete File

## Epic Summary

**Epic Name:** Google Drive – Delete File

**Goal:**  
Cho phép workflow builder xoá vĩnh viễn một file trên Google Drive bằng File ID hoặc File URL, hỗ trợ cả nhập giá trị trực tiếp và kéo expression từ upstream step.

**Context:**  
Action này dùng để xoá vĩnh viễn một file khỏi Google Drive. File sau khi xoá sẽ không được chuyển vào Trash và không thể khôi phục.

Để hỗ trợ user không biết File ID, UI có button `Add search file step`. Chức năng này đã có sẵn và sẽ thêm node Search file phía trước để user lấy output `file_id`, sau đó kéo expression vào field của Delete File.

**Expected UI Fields:**
- Add search file step
- File Input Type
- File ID
- File URL

---

## User Stories

### Story 1: Configure file by File ID or File URL

**Story:**  
As an automation workflow builder, I want to provide either a File ID or File URL, so that the action can identify and permanently delete the correct Google Drive file.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. File Input Type default là File ID**  
Given user mở Configure step của action Delete File, When UI được hiển thị, Then field File Input Type default là File ID.

**AC.02. User có thể chọn File ID hoặc File URL**  
Given user mở field File Input Type, When options được hiển thị, Then hệ thống hiển thị 2 options: File ID và File URL.

**AC.03. Hiển thị field theo File Input Type**  
Given user chọn File ID, When UI render, Then hệ thống chỉ hiển thị field File ID.  
Given user chọn File URL, When UI render, Then hệ thống chỉ hiển thị field File URL.

**AC.04. File ID là required khi chọn File ID**  
Given File Input Type = File ID, When field File ID đang trống và user click Continue, Then hệ thống hiển thị validation error: `File ID is required.`

**AC.05. File URL là required khi chọn File URL**  
Given File Input Type = File URL, When field File URL đang trống và user click Continue, Then hệ thống hiển thị validation error: `File URL is required.`

**AC.06. File ID support manual input và expression**  
Given user chọn File Input Type = File ID, When user nhập trực tiếp File ID hoặc kéo expression từ upstream step vào field File ID, Then hệ thống lưu cấu hình để resolve File ID tại runtime.

**AC.07. File URL support manual input và expression**  
Given user chọn File Input Type = File URL, When user nhập trực tiếp Google Drive File URL hoặc kéo expression từ upstream step vào field File URL, Then hệ thống lưu cấu hình để resolve File URL tại runtime.

**AC.08. File URL được dùng để xác định file cần xoá**  
Given File Input Type = File URL, When workflow chạy và File URL resolve thành giá trị hợp lệ, Then hệ thống xác định file tương ứng từ URL và thực hiện xoá file đó.

**AC.09. Clear value khi đổi input type**  
Given user đã nhập giá trị ở File ID, When user đổi File Input Type sang File URL, Then hệ thống clear giá trị File ID.  
Given user đã nhập giá trị ở File URL, When user đổi File Input Type sang File ID, Then hệ thống clear giá trị File URL.

**Edge Cases:**
- File ID hoặc File URL resolve thành empty value tại runtime → action fail với message tương ứng:
  - `File ID resolved to an empty value. Check your variable mapping.`
  - `File URL resolved to an empty value. Check your variable mapping.`
- File URL không xác định được file hợp lệ → action fail: `File URL must contain a valid Google Drive file.`
- File không tồn tại hoặc connected account không có access → action fail: `File not found. It may have been deleted or the connected account may not have access.`
- File ID hoặc File URL trỏ tới folder → action fail: `Only files can be deleted. Folder deletion is not supported.`

---

### Story 2: Reuse Add search file step

**Story:**  
As an automation workflow builder, I want to quickly add a Search file step before Delete File, so that I can find the target file first and map its File ID into this action.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Button hiển thị phía trên file input fields**  
Given user đang ở Configure step của action Delete File, When panel được hiển thị, Then button Add search file step nằm phía trên field File Input Type.

**AC.02. Button dùng lại chức năng đã có sẵn**  
Given user click Add search file step, When hệ thống xử lý, Then hệ thống sử dụng lại chức năng đã phát triển để thêm node Search file phía trước node Delete File.

**AC.03. User tự map output từ Search file**  
Given node Search file đã được thêm phía trước, When user cấu hình node Delete File, Then user có thể kéo expression từ output `file_id` của node Search file vào field File ID.

**Edge Cases:**
- Workflow đã có Search file step trước đó → user vẫn có thể tự kéo expression `file_id` vào field File ID.
- User click Add search file step nhiều lần → hệ thống xử lý theo behavior hiện có của chức năng Add search file step.

---

### Story 3: Execute permanent file deletion and return output

**Story:**  
As an automation workflow builder, I want the action to permanently delete the selected file and return a clear result, so that downstream steps can use the deletion outcome.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Delete by File ID successfully**  
Given File Input Type = File ID, And File ID resolve thành file hợp lệ, And connected account có quyền xoá file, When workflow chạy, Then hệ thống xoá vĩnh viễn file đó, And file không được chuyển vào Trash.

**AC.02. Delete by File URL successfully**  
Given File Input Type = File URL, And File URL resolve thành file hợp lệ, And connected account có quyền xoá file, When workflow chạy, Then hệ thống xoá vĩnh viễn file đó, And file không được chuyển vào Trash.

**AC.03. Return output payload**  
Given action xoá file thành công, When execution hoàn tất, Then action trả Output Payload gồm các fields tại mục Output.

**AC.04. Re-run với file đã bị xoá**  
Given file đã bị xoá ở execution trước, When workflow chạy lại với cùng File ID hoặc File URL, Then action fail vì file không còn tồn tại.

**Edge Cases:**
- Connected account mất quyền xoá file tại runtime → action fail: `Cannot delete file: the connected account does not have sufficient rights.`
- Google authentication expired hoặc revoked → action fail: `Google authentication failed. Please reconnect your Google account.`
- Provider rate limit, timeout, hoặc temporary failure → Dùng lại logic retry của hệ thống.

---

## Out of Scope

- File picker trực tiếp trong action Delete File.
- Search file trực tiếp bên trong Delete File.
- Auto map output từ Search file vào Delete File.
- Xoá folder.
- Move file vào Trash.
- Bulk delete nhiều files.
- Restore file sau khi xoá.
- Delete bằng file name, path, hoặc query.

---

## Inputs

| Field | Type | Required | Default | Manual Input | Expression | Description |
|---|---|---:|---|---:|---:|---|
| File Input Type | enum | Yes | File ID | No | No | Chọn cách xác định file cần xoá: File ID hoặc File URL. |
| File ID | string | Conditional | None | Yes | Yes | File ID cần xoá. Hiển thị khi File Input Type = File ID. |
| File URL | string | Conditional | None | Yes | Yes | Google Drive File URL cần xoá. Hiển thị khi File Input Type = File URL. |

---

## Outputs

| Field | Type | Description |
|---|---|---|
| success | boolean | Trạng thái delete thành công. |
| input_type | string | Cách xác định file: `file_id` hoặc `file_url`. |
| file_id | string | File ID đã được xoá. |
| file_url | string / null | File URL đã sử dụng, nếu user chọn File URL. |
| operation_result | string | Fixed value: `file_permanently_deleted`. |
| timestamp | string | Thời điểm xoá file theo ISO 8601. |

---

## Business Rules

1. Action chỉ xoá file, không xoá folder.
2. File bị xoá vĩnh viễn, không chuyển vào Trash.
3. User phải chọn File ID hoặc File URL.
4. File Input Type default là File ID.
5. Chỉ hiển thị field tương ứng với File Input Type.
6. Khi đổi File Input Type, hệ thống clear giá trị field cũ.
7. File ID và File URL đều support manual input và expression.
8. Add search file step chỉ reuse chức năng đã có sẵn, không tạo logic search mới trong action Delete File.
9. User tự kéo expression từ output `file_id` của Search file vào field File ID.
10. Output Payload phải đủ để downstream steps biết file nào đã bị xoá và kết quả xoá là gì.
11. Execution Log phải ghi nhận input type, file identifier, operation result, actor, timestamp, và trạng thái execution.

---

## Validation Rules

### Config-time Validation

| Scenario | Message |
|---|---|
| File Input Type empty | `File Input Type is required.` |
| File Input Type = File ID và File ID empty | `File ID is required.` |
| File Input Type = File URL và File URL empty | `File URL is required.` |
| Manual File URL không hợp lệ | `File URL must be a valid Google Drive file URL.` |

### Runtime Validation

| Scenario | Message |
|---|---|
| File ID expression resolve empty | `File ID resolved to an empty value. Check your variable mapping.` |
| File URL expression resolve empty | `File URL resolved to an empty value. Check your variable mapping.` |
| File URL không xác định được file hợp lệ | `File URL must contain a valid Google Drive file.` |
| File không tồn tại hoặc không có access | `File not found. It may have been deleted or the connected account may not have access.` |
| Input trỏ tới folder | `Only files can be deleted. Folder deletion is not supported.` |
| Không đủ quyền delete | `Cannot delete file: the connected account does not have sufficient rights.` |
| Google authentication failed | `Google authentication failed. Please reconnect your Google account.` |
| Rate limit / timeout / temporary provider failure | Dùng lại logic retry của hệ thống. |

---

# Lean Example 3
# Ticket Title: Action - OpenAI - Text to Speech

## Epic Summary

**Epic Name:** OpenAI – Text to Speech

**Goal:**
Cho phép automation workflow builder chuyển đổi văn bản thành audio file MP3 bằng OpenAI TTS API, với khả năng chọn model, voice, speed và đặt tên file output.

**Context:**
Action sử dụng OpenAI Audio API endpoint `POST /v1/audio/speech` để generate audio từ text input. Hệ thống mặc định export dạng MP3, không cho user chọn format trên UI. Pattern tương tự action Gemini – Text to Speech (đang In progress) nhưng dùng OpenAI provider với connection type riêng. Connection dùng lại logic Connected Account hiện có của OpenAI integration (cùng connection với Summarize và Conversation đã Done).

**Expected UI Fields:**
- Connection *
- Model *
- Text *
- Voice
- Speed
- File Name *

---

## User Stories

---

### Story 1: Connect Account and select action

**Story:** As an automation workflow builder, I want to connect an account and confirm the action type, so that the Text to Speech action runs under the correct authenticated identity.

**Priority:** Must-have

**Acceptance Criteria:**

AC.01. Account * is required — cannot continue if not selected
Given the Connect step is open
When the "Account *" field has no account selected
Then the "Continue" button is disabled
And if the user clicks "Continue", a validation error is shown: "Account is required."

AC.02. User selects or connects an account
Như hiện trạng của hệ thống

AC.03. Action * displays "Text to Speech" as pre-selected value
Given the user is configuring this node
When the user views the "Action *" field
Then the dropdown shows "Text to Speech" already pre-selected

---

### Story 2: Select model and enter text

**Story:** As an automation workflow builder, I want to select an OpenAI TTS model and enter the text to convert, so that the action generates audio with the correct model and content.

**Priority:** Must-have

**Acceptance Criteria:**

AC.01. Model là required field — không có default
Given field `Model` chưa được chọn, When user click Continue, Then validation error hiển thị: `Model is required.`

AC.02. Model dropdown hiển thị các models hỗ trợ TTS
Given user mở dropdown `Model`, When options được load, Then hệ thống hiển thị các options:

| UI Label | API value | Description |
|---|---|---|
| TTS-1 | tts-1 | Optimized for speed |
| TTS-1 HD | tts-1-hd | Optimized for quality |

AC.03. Model không support dynamic variable
Given field `Model` là dropdown, Then field này không hỗ trợ expression/dynamic variable, And user chỉ có thể chọn từ danh sách cố định.

AC.04. Text là required field
Given field `Text` đang trống, When user click Continue, Then validation error hiển thị: `Text is required.`

AC.05. Text không được vượt quá 4,096 characters (config-time)
Given user nhập văn bản trực tiếp vào field `Text`, When nội dung vượt quá 4,096 characters, Then system block Continue, And validation error hiển thị: `Text is too long. Please shorten it and try again.`

AC.06. Text support dynamic variable
Given upstream step trả về giá trị text, When user map dynamic variable vào field `Text`, Then hệ thống resolve giá trị tại runtime và sử dụng nó để generate speech.

AC.07. Text không được resolve thành empty value tại runtime
Given field `Text` được map từ variable, When action chạy và resolved value là empty string hoặc null, Then action fail: `Text resolved to an empty value. Check your variable mapping.`

AC.08. Mapped Text vượt quá giới hạn tại runtime
Given field `Text` được map từ variable, When action chạy và resolved value vượt quá 4,096 characters, Then action fail: `Text is too long. Please shorten the mapped value and try again.`

AC.09. Text hỗ trợ multiline
Given user nhập text có nhiều dòng, When action chạy, Then hệ thống xử lý đúng multiline text và generate speech tương ứng.

**Edge Cases:**
- Text chứa special characters hoặc emojis → API xử lý hoặc skip characters không hỗ trợ.

---

### Story 3: Configure voice and speed

**Story:** As an automation workflow builder, I want to configure voice and speed, so that I can customize the audio output tone and pace.

**Priority:** Must-have

**Acceptance Criteria:**

AC.01. Voice có default value
Given user chưa thay đổi `Voice`, When action được save hoặc execute, Then hệ thống sử dụng `Alloy` làm giá trị mặc định.

AC.02. Voice dropdown hiển thị danh sách cố định
Given user mở dropdown `Voice`, When options được hiển thị, Then hệ thống hiển thị các options:

| UI Label | API value |
|---|---|
| Alloy | alloy |
| Echo | echo |
| Fable | fable |
| Onyx | onyx |
| Nova | nova |
| Shimmer | shimmer |

AC.03. Voice không support dynamic variable
Given field `Voice` là dropdown cố định, Then field này không hỗ trợ expression/dynamic variable.

AC.04. Speed default là Normal
Given user chưa thay đổi `Speed`, When action được save hoặc execute, Then hệ thống sử dụng `Normal` làm giá trị mặc định.

AC.05. Speed dropdown hiển thị preset options
Given user mở dropdown `Speed`, When options được hiển thị, Then hệ thống hiển thị các options:

| UI Label | API value | Description |
|---|---|---|
| Slow | 0.75 | Tốc độ chậm |
| Normal | 1.0 | Tốc độ bình thường |
| Fast | 1.5 | Tốc độ nhanh |

AC.06. Speed không support dynamic variable
Given field `Speed` là dropdown cố định, Then field này không hỗ trợ expression/dynamic variable.

**Edge Cases:**
- Không có edge case đặc biệt vì cả Voice và Speed đều là dropdown cố định, không có runtime resolution.

---

### Story 4: Specify output file name

**Story:** As an automation workflow builder, I want to specify the name of the generated audio file, so that I can organize and identify files easily in downstream steps.

**Priority:** Must-have

**Acceptance Criteria:**

AC.01. File Name là required field
Given field `File Name` đang trống, When user click Continue, Then validation error hiển thị: `File Name is required.`

AC.02. File Name support manual input và dynamic variable
Given user nhập giá trị text hoặc kéo expression từ upstream step vào field `File Name`, Then hệ thống lưu cấu hình để resolve tại runtime.

AC.03. File Name tự động append .mp3 extension nếu chưa có
Given user nhập file name không có extension hoặc extension khác `.mp3`, When action chạy, Then hệ thống tự động append `.mp3`.

AC.04. File Name validation cho special characters (config-time)
Given user nhập file name chứa ký tự không hợp lệ (`/`, `\`, `:`, `*`, `?`, `"`, `<`, `>`, `|`), When user click Continue, Then validation error hiển thị: `File Name contains invalid characters. Please use only letters, numbers, space, hyphens, and underscores.`

AC.05. File Name không được vượt quá 100 characters
Given user nhập file name dài hơn 100 characters, When user click Continue, Then validation error hiển thị: `File Name must be 100 characters or less.`

AC.06. File Name không được resolve thành empty value tại runtime
Given field `File Name` được map từ variable, When action chạy và resolved value là empty hoặc chỉ chứa spaces, Then action fail: `File Name cannot be empty or contain only spaces.`

---

### Story 5: Execute text-to-speech conversion and return result

**Story:** As an automation workflow builder, I want the action to convert text to speech and return a usable audio file URL, so that downstream steps can use the audio output.

**Priority:** Must-have

**Acceptance Criteria:**

AC.01. Audio file được lưu vào platform storage dạng MP3
Given OpenAI API trả về audio data thành công, When action xử lý response, Then hệ thống lưu audio file dạng MP3 vào platform storage, And generate một URL accessible cho file đó.

AC.02. Output payload sau khi conversion thành công
Given action generate audio thành công, When execution hoàn tất, Then action trả Output Payload gồm các fields tại mục Output.

AC.03. Output field `file` trả về binary object cho downstream expression
Given action generate audio thành công, When downstream step cần sử dụng audio file dạng binary (upload, send, convert), Then downstream step có thể kéo expression từ field `file` của node này, And `file` chứa đầy đủ metadata theo Binary File standard: `data`, `fileName`, `id`, `mimeType` (`audio/mpeg`), `size`, `success`.

AC.04. Audio URL có thời hạn sử dụng
Given audio file được generate thành công, When hệ thống trả về `audio_url`, Then URL expiration dùng lại rule expired file của hệ thống.

---

### Story 6: Handle authentication and permission errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

AC.01. Quota exceeded
Given model đã hết quota
When action chạy
Then action fail: "Quota for model {model_name} has been exceeded. Please select another model."

---

## Out of Scope

- Chọn output format — hệ thống fix cứng MP3, không expose trên UI.
- Custom speed value — chỉ hỗ trợ preset, không cho nhập số tự do.
- Dynamic variable cho Voice, Speed, Model — cả 3 là dropdown cố định.

---

## Inputs

| Field | Type | Required | Default | Manual Input | Expression | Description |
|---|---|---|---|---|---|---|
| Connection | Connected Account | Yes | None | No | No | Dùng lại logic Connected Account hiện có của OpenAI integration. |
| Model | Dropdown | Yes | None (empty) | No | No | Options: TTS-1, TTS-1 HD. |
| Text | Textarea | Yes | None | Yes | Yes | Text cần chuyển thành speech. Max 4,096 characters. |
| Voice | Dropdown | No | Alloy | No | No | Options: Alloy, Echo, Fable, Onyx, Nova, Shimmer. |
| Speed | Dropdown | No | Normal | No | No | Options: Slow, Normal, Fast. |
| File Name | Text input | Yes | None | Yes | Yes | Tên file MP3 output. Auto-append `.mp3` nếu chưa có. |

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| success | boolean | Whether action succeeded | No |
| file | Binary (object) | Audio file dạng binary. Downstream steps có thể kéo expression `file` để sử dụng như binary input (upload, send, convert). Structure: `data`, `fileName`, `id`, `mimeType` (`audio/mpeg`), `size`, `success`. | No |
| audio_url | string | URL of the generated MP3 file (có expiration theo rule expired file của hệ thống) | No |
| file_name | string | Name of the generated file (with `.mp3`) | No |
| duration | number | Audio duration in seconds | Yes |
| file_size | number | File size in bytes | No |
| format | string | Always `mp3` | No |
| model_used | string | Model used: `tts-1` or `tts-1-hd` | No |
| voice_used | string | Voice used for generation | No |
| speed_used | string | Speed preset used: `slow`, `normal`, `fast` | No |
| action | string | Fixed: `text_to_speech` | No |
| executed_at | string | Execution timestamp ISO 8601 | No |

---

## Business Rules

1. Action chỉ generate một MP3 audio file trong mỗi execution.
2. Output format luôn là MP3, không expose cho user chọn.
3. Model là required, không có default — user phải chọn trước khi Continue.
4. Voice default là `Alloy` nếu user không thay đổi.
5. Speed default là `Normal` (1.0) nếu user không thay đổi.
6. Text max 4,096 characters — validate config-time cho static input, runtime cho expression.
7. File Name tự động append `.mp3` nếu user chưa nhập extension.
8. Voice, Speed, Model là dropdown cố định — không support dynamic variable.
9. Text và File Name support cả manual input và dynamic variable.
10. `audio_url` phải accessible và có expiration policy theo rule expired file của hệ thống.
11. `model_used`, `voice_used`, `speed_used` phải trả về actual values được sử dụng.
12. Output field `file` trả về audio dạng binary theo Binary File standard, cho phép downstream steps kéo expression để sử dụng trực tiếp (upload to Drive, send via Telegram, etc.).

---

## Assumptions

1. Connection type là OpenAI, dùng cùng connection với Summarize và Conversation đã Done.
2. Generated audio files nhỏ hơn 25MB per file.
3. Danh sách models (tts-1, tts-1-hd) và voices là cố định, không cần load dynamic từ API.
4. Speed presets map sang API values: Slow=0.75, Normal=1.0, Fast=1.5.

