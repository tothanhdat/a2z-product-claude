# Ticket: Trigger - Google Calendar - New Event (Instant)

## Epic Summary

**Epic Name:** Google Calendar – New Event Trigger (Instant)

**Goal:**
Cho phép automation workflow builder tự động kích hoạt workflow instantly (trong vài giây) khi có event mới được tạo trong Google Calendar account đã kết nối, phục vụ các use case nhạy cảm về thời gian như instant notifications, real-time CRM updates, immediate task creation, hoặc urgent team alerts.

**Context:**
Trigger này sử dụng Google Calendar Push Notifications (webhook-based mechanism) để nhận real-time notifications từ Google khi có event mới được tạo. Connection dùng lại logic Connected Account hiện có của Google Calendar integration. Trigger chỉ fire khi event mới được tạo (created), không fire cho events đã tồn tại trước đó hoặc events được updated. Trigger hỗ trợ monitoring tất cả calendars accessible hoặc chỉ specific calendars được chọn.

**Expected UI Fields:**
- Account *
- Calendar Selection Mode *
- Specific Calendars (conditional)
- Search Term

---

## User Stories

### Story 1: Connect account and select trigger

**Story:** As an automation workflow builder, I want to connect an account and confirm the trigger type, so that the New Event trigger runs under the correct authenticated identity.

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

**AC.03. Selected account displays name and email for confirmation**
Given the user has selected an account,
Then the "Account" field displays the account name and email (e.g., Nguyen (nguyennnp@company.io)),
And the user can switch accounts by clicking "Select" again

**AC.04. Trigger * displays "New Event" as pre-selected value**
Given the user is configuring this node,
When the user views the "Trigger *" field,
Then the dropdown shows "New Event" already pre-selected,
And the user can change to other available triggers if needed

---

### Story 2: Configure calendar selection mode

**Story:** As an automation workflow builder, I want to cấu hình calendar monitoring scope, so that trigger chỉ nhận notifications từ calendars mong muốn.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Calendar Selection Mode options**
Given user mở dropdown Calendar Selection Mode,
When options được hiển thị,
Then hệ thống hiển thị các options:

| UI Label | API value |
|---|---|
| All my Calendars | all |
| Specific calendars | specific |

**AC.02. Calendar Selection Mode default is All my Calendars**
Given user chưa thay đổi Calendar Selection Mode,
When trigger được save,
Then hệ thống sử dụng All my Calendars làm giá trị mặc định.

**AC.03. Behavior of All my Calendars**
Given user chọn All my Calendars,
When trigger được activate,
Then hệ thống monitors tất cả accessible calendars của connected account,
And excludes holiday calendars và birthday calendars.

**AC.04. Specific Calendars field visibility**
Given user chọn Specific calendars trong Calendar Selection Mode,
When UI updates,
Then trường Specific Calendars hiển thị và là required.

**AC.05. Specific Calendars is required when visible**
Given user chọn Specific calendars mode,
And Specific Calendars để trống,
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Specific Calendars is required."

**AC.06. Specific Calendars selector behavior**
Given user chọn Specific calendars mode,
When user mở Specific Calendars selector,
Then hệ thống load danh sách accessible calendars của connected account (excludes holiday calendars và birthday calendars),
And cho phép user chọn một hoặc nhiều calendars.

**Edge Cases:**
- Calendar được chọn trong Specific Calendars bị deleted hoặc revoked access sau config: trigger không fire cho calendar đó, ghi log warning.
- User thay đổi Calendar Selection Mode hoặc Specific Calendars khi trigger đang active: existing webhook stopped, new webhook registered với updated scope.

---

### Story 3: Configure search term filter

**Story:** As an automation workflow builder, I want to filter new events by title keywords, so that trigger chỉ fire cho events có title match search term.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Search Term is optional**
Given user để trống Search Term,
When trigger được save,
Then không hiển thị validation error,
And trigger fire cho tất cả new events thuộc calendar scope.

**AC.02. Search Term supports manual input and expression**
Given user nhập free-text vào Search Term hoặc kéo expression từ upstream step,
When trigger được save,
Then hệ thống lưu cấu hình để resolve giá trị tại runtime.

**AC.03. Search Term filters by event title only**
Given Search Term có giá trị resolve,
When trigger detect new event,
Then chỉ fire cho events có summary (title) chứa search term (substring match, case-insensitive),
And không filter dựa trên description, location, attendees, hoặc bất kỳ field nào khác.

**AC.04. Search Term resolved to empty value at runtime**
Given Search Term được map từ variable,
When trigger chạy và resolved value là empty string hoặc null,
Then hệ thống xử lý như Search Term không có giá trị,
And fire cho tất cả new events thuộc calendar scope.

---

### Story 4: Instant event detection via webhook

**Story:** As an automation workflow builder, I want trigger tự động nhận instant notifications từ Google Calendar, so that workflow fires ngay khi có event mới được tạo.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Trigger fires for each new event**
Given webhook nhận push notification về event mới,
When hệ thống xử lý notification,
Then trigger fires workflow một lần cho event đó,
And truyền event details vào workflow execution.

**AC.02. Only fire for created events**
Given webhook nhận notification về một event,
When notification type là event created,
Then trigger fires workflow cho event đó.

**AC.03. Ignore updated and deleted events**
Given webhook nhận notification về một event,
When notification type là event updated hoặc deleted,
Then trigger không fire workflow.

**AC.04. Filter by calendar selection**
Given user đã cấu hình Specific calendars,
When webhook nhận notification từ calendar không nằm trong danh sách đã chọn,
Then trigger không fire workflow.

**AC.05. Exclude holiday and birthday calendars in All my Calendars mode**
Given user chọn All my Calendars mode,
When webhook nhận notification từ holiday calendar hoặc birthday calendar,
Then trigger không fire workflow.

**AC.06. Deduplicate notifications**
Given webhook nhận duplicate notifications cho cùng event,
When hệ thống xử lý,
Then trigger chỉ fires workflow một lần cho event đó.

**Edge Cases:**
- Webhook nhận notification từ calendar đã bị deleted: trigger không fire, ghi log.
- Google Calendar gửi notification delay (rare case): trigger vẫn fires nhưng có thể chậm hơn vài giây/phút.

---

### Story 5: Output payload for downstream steps

**Story:** As an automation workflow builder, I want trigger trả về detailed event information, so that các actions downstream có thể xử lý event data.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Output payload when trigger fires**
Given trigger fires cho new event,
When downstream steps đọc output,
Then trigger trả Output Payload gồm các fields tại mục Output.

**Edge Cases:**
- Event không có description, location, hoặc attendees: các fields tương ứng trả về null hoặc empty array.

---

### Story 6: Handle authentication and permission errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. OAuth token expired or revoked**
Given the connected account's OAuth token has expired or been revoked,
When the trigger attempts to execute,
Then the trigger fails with: "Authentication failed. Please reconnect your Google account.",
And the workflow execution is marked as failed

**AC.02. Insufficient permission on resource**
Given the connected account does not have required access to calendar events,
When the trigger runs,
Then the trigger surfaces: "The connected account does not have sufficient rights to access calendar events."

**AC.03. Calendar not found or access revoked**
Given one or more selected calendars no longer exist or access has been revoked,
When the trigger runs,
Then the trigger surfaces: "Calendar not found or access has been revoked.",
And the workflow execution is marked as failed

**AC.04. Rate limit exceeded — automatic retry with exponential backoff**
Given the API returns HTTP 429 or rate limit error,
When the error occurs,
Then the trigger surfaces: "Google Calendar rate limit reached. Retrying with exponential backoff (max 3 attempts).",
And if all retries fail, the workflow execution is marked as failed

---

## Out of Scope

- **Trigger cho event updates hoặc deletions** — covered by separate trigger types.
- **Trigger cho event cancellations** — separate use case.
- **Polling-based detection mechanism** — trigger này chỉ dùng instant webhook.
- **Batch processing nhiều events trong một workflow execution** — mỗi event fires riêng.
- **Custom webhook endpoint configuration** — hệ thống tự quản lý webhook.
- **Manual webhook registration/deregistration** — hệ thống tự động lifecycle.
- **Event filtering theo event properties ngoài title** — chỉ hỗ trợ Search Term filter theo title.
- **Historical event processing** — events created trước khi trigger activate không được process.
- **Expand recurring events thành individual instances** — chỉ fire cho master event.

---

## Inputs

| Field | Type | Required | Default | Static Selection | Manual Input | Mapped Variable | Notes |
|---|---|---|---|---|---|---|---|
| Account | Connected Account | Yes | None | Yes | No | No | Dùng lại logic Connected Account hiện có. |
| Calendar Selection Mode | Dropdown | Yes | All my Calendars | Yes | No | No | Options: xem AC.01 Story 2. |
| Specific Calendars | Multi-select calendar picker | Conditional | None | Yes | No | No | Required khi mode = Specific calendars. |
| Search Term | Text input | No | None | No | Yes | Yes | Optional. Filter event theo title. |

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| event_id | string | ID của event mới | No |
| calendar_id | string | ID của calendar chứa event | No |
| calendar_name | string | Tên calendar | No |
| summary | string | Tiêu đề event | No |
| description | string | Mô tả event nếu có | Yes |
| start_time | string | Thời gian bắt đầu theo ISO 8601 | No |
| end_time | string | Thời gian kết thúc theo ISO 8601 | No |
| timezone | string | Timezone của event | No |
| location | string | Địa điểm event nếu có | Yes |
| is_all_day_event | boolean | Event có phải all-day event không | No |
| attendees | array | Danh sách attendees (email only) | No |
| organizer_email | string | Email của organizer | No |
| organizer_name | string | Tên organizer nếu có | Yes |
| event_link | string | Link đến event trong Google Calendar | No |
| event_created_at | string | Thời điểm event được tạo theo ISO 8601 | No |
| is_recurring_instance | boolean | Event có phải là instance của recurring event không | No |
| recurrence_id | string | ID của recurring event master nếu event là instance | Yes |
| detected_at | string | Thời điểm trigger nhận notification theo ISO 8601 | No |
| trigger_mode | string | `instant` để phân biệt với polling triggers | No |

---

## Business Rules

1. Account và Calendar Selection Mode là bắt buộc.
2. Calendar Selection Mode mặc định là All my Calendars.
3. Specific Calendars là required khi Calendar Selection Mode = Specific calendars.
4. Khi mode = All my Calendars, hệ thống monitors tất cả accessible calendars ngoại trừ holiday calendars và birthday calendars.
5. Khi mode = Specific calendars, hệ thống chỉ monitors các calendars được chọn.
6. Trigger chỉ fires cho events mới được tạo (created), không fires cho events được updated hoặc deleted.
7. Trigger fires một lần cho mỗi new event. Khi user tạo recurring event, trigger fires một lần cho master event — không expand thành individual instances.
8. Khi Calendar Selection Mode hoặc Specific Calendars thay đổi, existing webhook được stopped và new webhook được registered với updated scope.
9. Hệ thống tự động renew webhook channel trước khi expire.
10. Trigger không fires cho events được tạo trước khi trigger activate.
11. Holiday calendars và birthday calendars được xác định dựa trên calendar metadata từ Google Calendar API.
12. Search Term là optional. Khi để trống hoặc resolve thành empty, trigger fire cho tất cả new events thuộc calendar scope.
13. Khi Search Term có giá trị, trigger chỉ fire cho events có summary (title) chứa search term — matching case-insensitive, substring.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Account trống | Config-time | Account is required. |
| Specific Calendars trống khi mode = Specific calendars | Config-time | Specific Calendars is required. |
| Lỗi authentication | Runtime | Authentication failed. Please reconnect your Google account. |
| Không đủ quyền | Runtime | The connected account does not have sufficient rights to access calendar events. |
| Calendar không tồn tại hoặc mất quyền | Runtime | Calendar not found or access has been revoked. |
| Lỗi provider | Runtime | Google Calendar API returned an unexpected error. Please try again later. |

---

## Assumptions

1. Google Calendar API Push Notifications endpoint và webhook mechanism hoạt động stable.
2. Platform có webhook infrastructure để nhận và xử lý incoming notifications từ Google Calendar.
3. Connected Account đã được authenticate với đủ scopes để access calendar events và register webhooks.
4. Platform có khả năng detect và filter holiday calendars và birthday calendars dựa trên calendar metadata.
