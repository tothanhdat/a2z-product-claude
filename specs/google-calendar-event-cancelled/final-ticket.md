# Epic Name: Google Calendar – Event Cancelled Trigger (Scheduled)

**Goal:** Cho phép automation workflow builder tự động kích hoạt workflow ngay khi một event trong Google Calendar bị cancel/delete, phục vụ các use case như notify attendees, release booked resources, update CRM/booking records, hoặc trigger reschedule flow.

**Context:** Trigger sử dụng Google Calendar API events.list để detect các events có status chuyển sang `cancelled` trong polling window (giữa last_poll_time và current_poll_time). Connection, calendar selection, search term, và polling interval logic reuse pattern của New or Updated Event Trigger.

**Expected UI Fields:**
- Account *
- Calendar *
- Specific Calendars (conditional)
- Search Term
- Polling Interval *

---

## User Stories

### Story 1: Connect account and select trigger

**Story:** As an automation workflow builder, I want to connect a Google account and confirm the trigger type, so that the Event Cancelled trigger runs under the correct authenticated identity.

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

**AC.04. Trigger * displays "Event Cancelled" as pre-selected value**
Given the user is configuring this node,
When the user views the "Trigger *" field,
Then the dropdown shows "Event Cancelled" already pre-selected,
And the user can change to other available triggers if needed

---

### Story 2: Configure calendar selection scope

**Story:** As an automation workflow builder, I want to chọn calendar monitoring scope, so that trigger chỉ fire cho events thuộc calendars mong muốn.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Calendar dropdown options**
Given user mở dropdown Calendar,
When options được load,
Then hệ thống hiển thị các options:

| UI Label | API value | Description |
|---|---|---|
| All my Calendars | all | Monitor tất cả accessible calendars (excludes holiday và birthday calendars) |
| Specific calendars | specific | Chỉ monitor các calendars được chọn trong Specific Calendars |

**AC.02. Calendar defaults to All my Calendars**
Given user chưa thay đổi Calendar,
When trigger được save,
Then hệ thống sử dụng All my Calendars làm giá trị mặc định

**AC.03. Specific Calendars field visibility**
Given user chọn Specific calendars trong Calendar,
When UI updates,
Then trường Specific Calendars hiển thị,
And trường này là required

**AC.04. Specific Calendars is required when mode = Specific calendars**
Given user chọn Specific calendars mode,
And Specific Calendars để trống,
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "Specific Calendars is required."

**AC.05. Specific Calendars selector behavior**
Given user chọn Specific calendars mode,
When user mở Specific Calendars selector,
Then Calendar selector dùng lại picker và calendar access logic hiện có của Google Calendar integration,
And selector excludes holiday calendars và birthday calendars,
And cho phép user chọn một hoặc nhiều calendars

**Edge Cases:**
- Calendar được chọn trong Specific Calendars bị deleted hoặc revoked access sau config: trigger không fire cho calendar đó, tiếp tục polling cho các calendars còn lại trong scope và surface error theo mục Error Handling.

---

### Story 3: Configure search term filter

**Story:** As an automation workflow builder, I want to filter cancelled events by title keywords, so that trigger chỉ fire cho events có title match search term.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Search Term is optional**
Given user để trống Search Term,
When trigger được save,
Then không hiển thị validation error,
And trigger fire cho tất cả cancelled events thuộc calendar scope (không filter theo title)

**AC.02. Search Term supports manual input and expression**
Given user nhập free-text vào Search Term hoặc kéo expression từ upstream step,
When trigger được save,
Then hệ thống lưu cấu hình để resolve giá trị tại runtime

**AC.03. Search Term filters on event title only**
Given Search Term có giá trị resolve,
When trigger detect cancelled event,
Then chỉ fire cho events có summary (title) chứa search term,
And không filter dựa trên description, location, attendees, hoặc bất kỳ field nào khác

**AC.04. Search Term resolves to empty value at runtime**
Given Search Term được map từ variable,
When trigger chạy và resolved value là empty string hoặc null,
Then hệ thống xử lý như Search Term không có giá trị,
And fire cho tất cả cancelled events thuộc calendar scope

**AC.05. Search Term matching is case-insensitive**
Given Search Term có giá trị resolve,
When trigger so sánh search term với event title,
Then matching không phân biệt hoa thường (case-insensitive),
And chỉ cần substring của title chứa search term là match

---

### Story 4: Configure polling interval

**Story:** As an automation workflow builder, I want to chọn how often the trigger checks for cancelled events, so that workflow fires với tần suất phù hợp với use case.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Polling Interval dropdown options**
Given user mở dropdown Polling Interval,
When options được load,
Then hệ thống hiển thị các options:

| UI Label | API value (minutes) |
|---|---|
| 1 minute | 1 |
| 5 minutes | 5 |
| 15 minutes | 15 |
| 30 minutes | 30 |
| 1 hour | 60 |

**AC.02. Polling Interval defaults to 15 minutes**
Given user chưa thay đổi Polling Interval,
When trigger được save,
Then hệ thống sử dụng 15 minutes làm giá trị mặc định

**AC.03. Polling Interval does not support expressions**
Given field Polling Interval là dropdown cố định,
Then field này không hỗ trợ expression hoặc dynamic variable,
And user chỉ có thể chọn từ danh sách cố định

---

### Story 5: Detect cancelled events and fire workflow

**Story:** As an automation workflow builder, I want trigger fire workflow khi event bị cancel, so that downstream steps có thể xử lý ngay sau khi event bị huỷ.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Trigger fires when event status changes to cancelled**
Given user đã activate trigger,
When một event trong calendar scope chuyển sang status `cancelled` (do user xoá event hoặc cancel event trên Google Calendar),
Then trigger fire workflow một lần cho event đó,
And output trả về các fields tại mục Output

**AC.02. Trigger does not fire for non-cancellation changes**
Given một event trong calendar scope bị thay đổi,
When change không phải là cancellation (VD: attendee decline, title/time edit),
Then trigger không fire cho event đó

**AC.03. Filter by Calendar — All my Calendars**
Given user chọn All my Calendars,
When trigger detect cancelled events,
Then hệ thống monitor tất cả accessible calendars,
And excludes holiday calendars và birthday calendars

**AC.04. Filter by Specific Calendars**
Given user chọn Specific calendars mode,
When trigger detect cancelled events,
Then hệ thống chỉ monitor các calendars được chọn trong Specific Calendars

**AC.05. Filter by Search Term**
Given Search Term có giá trị resolve,
When trigger detect cancelled event,
Then chỉ fire cho events có title (summary) chứa search term

**AC.06. Recurring event — single instance cancelled**
Given user cancel một instance của recurring event,
When trigger detect cancellation,
Then trigger fire một lần cho instance đó,
And `was_recurring = true` trong output

**AC.07. Recurring event — entire series cancelled**
Given user cancel toàn bộ recurring series,
When trigger detect cancellation,
Then trigger fire một lần cho mỗi future instance (instances chưa diễn ra tại thời điểm cancel),
And `was_recurring = true` trong output cho mỗi lần fire,
And past instances (đã xảy ra) không fire

**AC.08. Historical cancellations are not processed**
Given trigger được activate lần đầu,
When có events đã bị cancel trước thời điểm activation,
Then trigger không fire cho các events đó

**Edge Cases:**
- Cùng một event bị cancel nhiều lần (cancel → restore → cancel) trong các polling windows khác nhau: trigger fire cho mỗi lần cancel.

---

### Story 6: Output payload cho downstream steps

**Story:** As an automation workflow builder, I want trigger trả về detailed cancelled event information, so that downstream steps có thể xử lý event data hiệu quả.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Output contains full cancelled event information**
Given trigger fires for a cancelled event,
When downstream steps access the output,
Then output trả về các fields mô tả tại mục Output

**Edge Cases:**
- Event không có description, location, hoặc attendees: các fields tương ứng trả về null hoặc empty array.

---

### Story 7: Handle authentication and permission errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. OAuth token expired or revoked**
Given the connected account's OAuth token has expired or been revoked,
When the trigger attempts to poll,
Then the trigger fails with: "Authentication failed. Please reconnect your Google account.",
And the polling tick is marked as failed

**AC.02. Insufficient permission on calendar**
Given the connected account does not have read access to a selected calendar,
When the trigger polls,
Then the trigger surfaces: "Cannot read calendar: the connected account does not have read access to this calendar. Check the sharing settings.",
And the polling tick continues for remaining calendars in scope

**AC.03. Calendar not found**
Given a selected calendar ID does not exist or has been deleted,
When the trigger polls,
Then the trigger surfaces: "Calendar not found or access has been revoked.",
And the polling tick continues for remaining calendars in scope

**AC.04. Rate limit exceeded — automatic retry with exponential backoff**
Given the Google Calendar API returns HTTP 429 or rate limit error,
When the error occurs,
Then the trigger surfaces: "Google Calendar rate limit reached.",
And the polling tick is marked as failed

---

## Dependencies

**Between Stories:**
- Story 1 (Connect account) must complete before all other stories can run.
- Story 7 (Error Handling) is a cross-cutting concern that applies to all other stories.

**External Dependencies:**
- **Google Calendar API** — events.list (GET)
- **OAuth 2.0** — requires read scope on Google Calendar
- **Connected Account** — must have read access to the selected calendars

---

## Out of Scope

- Trigger cho event creation, update, hoặc completion — đã cover bởi trigger "New Event", "New or Updated Event", "Event Ends".
- Attendee decline (responseStatus thay đổi) — không coi là cancellation; không fire trigger.
- Filter theo event properties khác title (description, location, attendees, organizer, status, organizer email) — chỉ Search Term filter trên title.
- Custom polling interval ngoài preset — chỉ hỗ trợ 1, 5, 15, 30 phút và 1 giờ.
- Instant webhook-based detection — trigger này theo polling, không dùng push notifications.
- Historical cancellation processing — events bị cancel trước khi trigger activate không được fire.
- Restore detection — trigger không fire khi cancelled event được restore lại.
- Batch processing nhiều cancelled events trong một workflow execution — fire một workflow cho mỗi event.

---

## Inputs

| Field | Type | Required | Default | Static Selection | Manual Input | Mapped Variable | Notes |
|---|---|---|---|---|---|---|---|
| Account | Connected Account | Yes | None | Yes | No | No | Dùng lại logic Connected Account hiện có của Google Calendar integration. |
| Calendar | Dropdown | Yes | All my Calendars | Yes | No | No | Options: xem Story 2, AC.01. |
| Specific Calendars | Multi-select calendar picker | Conditional | None | Yes | No | No | Required khi Calendar = Specific calendars. Calendar selector dùng lại picker hiện có của Google Calendar integration. |
| Search Term | Text input | No | None | No | Yes | Yes | Free-text filter on event title (summary) only. Empty = no filter. |
| Polling Interval | Dropdown | Yes | 15 minutes | Yes | No | No | Options: xem Story 4, AC.01. Không support expression. |

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| event_id | string | ID của event bị cancel | No |
| calendar_id | string | ID của calendar chứa event | No |
| calendar_name | string | Tên calendar | No |
| summary | string | Tiêu đề event | No |
| description | string | Mô tả event nếu có | Yes |
| start_time | string | Thời gian bắt đầu theo ISO 8601 | No |
| end_time | string | Thời gian kết thúc theo ISO 8601 | No |
| timezone | string | Timezone của event | No |
| location | string | Địa điểm event nếu có | Yes |
| attendees | array | Danh sách attendees, mỗi item chỉ có email | No |
| organizer_email | string | Email của organizer | No |
| organizer_name | string | Tên organizer nếu có | Yes |
| event_link | string | Link đến event trong Google Calendar | No |
| was_recurring | boolean | `true` nếu event là instance của recurring series, `false` nếu single event | No |
| updated_at | string | Thời điểm event bị thay đổi lần cuối (bao gồm cancellation) theo ISO 8601 | No |
| trigger_mode | string | Fixed value: polling | No |

---

## Business Rules

1. Account và Calendar là bắt buộc.
2. Calendar mặc định là All my Calendars.
3. Specific Calendars là required khi Calendar = Specific calendars.
4. Khi mode = All my Calendars, hệ thống monitor tất cả accessible calendars ngoại trừ holiday calendars và birthday calendars.
5. Khi mode = Specific calendars, hệ thống chỉ monitor các calendars được chọn (excludes holiday và birthday calendars trong selector).
6. Polling Interval mặc định là 15 phút.
7. Polling Interval chỉ hỗ trợ preset values: 1, 5, 15, 30 phút và 1 giờ.
8. Polling Interval không support expression hoặc dynamic variable.
9. Search Term là optional; khi không có giá trị, trigger fire cho tất cả cancelled events thuộc calendar scope.
10. Search Term chỉ filter trên summary (title) của event, không filter trên các fields khác.
11. Search Term matching không phân biệt hoa thường (case-insensitive) và áp dụng substring match.
12. Search Term support cả manual input và expression.
13. Trigger fire khi event chuyển sang status `cancelled` trong polling window.
14. Trigger không fire cho các thay đổi không phải cancellation: attendee decline, edit title/time.
15. Cancel single instance của recurring event: fire một lần cho instance đó.
16. Cancel toàn bộ recurring series: fire một lần cho mỗi future instance; past instances không fire.
17. Khi trigger được activate lần đầu, polling bắt đầu từ thời điểm activation. Events bị cancel trước thời điểm activation không được xử lý.
18. Trigger không fire khi cancelled event được restore lại.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Account trống | Config-time, khi field touched và để trống | Account is required. |
| Specific Calendars trống khi mode = Specific calendars | Config-time, khi field touched và để trống | Specific Calendars is required. |
| OAuth token expired hoặc revoked | Runtime, khi polling tick chạy | Authentication failed. Please reconnect your Google account. |
| Không đủ quyền đọc calendar | Runtime, khi polling tick chạy | Cannot read calendar: the connected account does not have read access to this calendar. Check the sharing settings. |
| Calendar không tồn tại hoặc mất quyền | Runtime, khi polling tick chạy | Calendar not found or access has been revoked. |
| Rate limit của Google Calendar API | Runtime, khi polling tick chạy | Google Calendar rate limit reached. |
| Network / timeout | Runtime, khi polling tick chạy | Dùng lại rule retry của hệ thống. |
| Lỗi provider khác | Runtime, khi polling tick chạy | Google Calendar API returned an unexpected error. Please try again later. |

---

## Assumptions

- Google Calendar API trả về đầy đủ thông tin cho cancelled events (bao gồm event metadata và `updated` timestamp).
- Platform có scheduler infrastructure để chạy polling theo các interval preset (1, 5, 15, 30 phút, 1 giờ).
- Connected Account đã được authenticate với đủ scopes để read calendar events.
- Mỗi cancellation event được trigger detect một lần duy nhất (de-dup logic dùng lại pattern hiện có của trigger polling khác).
