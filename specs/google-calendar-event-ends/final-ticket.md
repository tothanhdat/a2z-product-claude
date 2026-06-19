# Epic Name: Google Calendar – Event Ends Trigger (Scheduled)

**Goal:** Cho phép automation workflow builder tự động kích hoạt workflow ngay sau khi một event trong Google Calendar đã kết thúc, phục vụ các use case như post-meeting follow-up, auto-send meeting notes, log time tracking, hoặc trigger reminders cho action items.

**Context:** Trigger sử dụng Google Calendar API events.list (GET) với singleEvents=true để query các events có end_time rơi vào polling window (giữa last_poll_time và current_poll_time). Connection và calendar selection logic dùng lại pattern hiện có.

**Expected UI Fields:**
- Account *
- Calendar *
- Specific Calendars (conditional)
- Search Term
- Polling Interval *

---

## User Stories

### Story 1: Connect account and select trigger

**Story:** As an automation workflow builder, I want to connect a Google account and confirm the trigger type, so that the Event Ends trigger runs under the correct authenticated identity.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01.** Account * is required — cannot continue if not selected

Given the Connect step is open
When the "Account *" field has no account selected
Then the "Continue" button is disabled
And if the user clicks "Continue", a validation error is shown: "Account is required."

**AC.02.** User selects or connects an account via the "Select" button

Given the user clicks the "Select" button
Then the system displays a list of already-connected accounts
And the user can select an account from the list
And if no account exists, the system redirects to the OAuth flow to connect a new one

**AC.03.** Selected account displays name and email for confirmation

Given the user has selected an account
Then the "Account" field displays the account name and email (e.g., Nguyen (nguyennnp@company.io))
And the user can switch accounts by clicking "Select" again

**AC.04.** Trigger * displays "Event Ends" as pre-selected value

Given the user is configuring this node
When the user views the "Trigger *" field
Then the dropdown shows "Event Ends" already pre-selected

---

### Story 2: Configure calendar selection scope

**Story:** As an automation workflow builder, I want to chọn calendar monitoring scope, so that trigger chỉ fire cho events thuộc calendars mong muốn.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01.** Calendar is required

Given Calendar để trống
When user click Continue hoặc Save
Then hiển thị validation error: "Calendar is required."

**AC.02.** Calendar dropdown options

Given user mở dropdown Calendar
When options được load
Then hệ thống hiển thị các options:

| UI Label | API value | Description |
|---|---|---|
| All my Calendars | all | Monitor tất cả accessible calendars (excludes holiday và birthday calendars) |
| Specific calendars | specific | Chỉ monitor các calendars được chọn trong Specific Calendars |

**AC.03.** Calendar defaults to All my Calendars

Given user chưa thay đổi Calendar
When trigger được save
Then hệ thống sử dụng All my Calendars làm giá trị mặc định

**AC.04.** Specific Calendars field visibility

Given user chọn Specific calendars trong Calendar
When UI updates
Then trường Specific Calendars hiển thị
And trường này là required

**AC.05.** Specific Calendars is required when mode = Specific calendars

Given user chọn Specific calendars mode
And Specific Calendars để trống
When user click Continue hoặc Save
Then hiển thị validation error: "Specific Calendars is required."

**AC.06.** Specific Calendars selector behavior

Given user chọn Specific calendars mode
When user mở Specific Calendars selector
Then Calendar selector dùng lại picker và calendar access logic hiện có của Google Calendar integration
And selector excludes holiday calendars và birthday calendars
And cho phép user chọn một hoặc nhiều calendars

**Edge Cases:**
- Calendar được chọn trong Specific Calendars bị deleted hoặc revoked access sau config: trigger không fire cho calendar đó, tiếp tục polling cho các calendars còn lại trong scope và surface error theo mục Error Handling.

---

### Story 3: Configure search term filter

**Story:** As an automation workflow builder, I want to filter events by title keywords, so that trigger chỉ fire cho events có title match search term.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01.** Search Term is optional

Given user để trống Search Term
When trigger được save
Then không hiển thị validation error
And trigger fire cho tất cả events thuộc calendar scope (không filter theo title)

**AC.02.** Search Term supports manual input and expression

Given user nhập free-text vào Search Term hoặc kéo expression từ upstream step
When trigger được save
Then hệ thống lưu cấu hình để resolve giá trị tại runtime

**AC.03.** Search Term filters on event title only

Given Search Term có giá trị resolve
When trigger query Google Calendar
Then chỉ fire cho events có summary (title) chứa search term
And không filter dựa trên description, location, attendees, hoặc bất kỳ field nào khác

**AC.04.** Search Term resolves to empty value at runtime

Given Search Term được map từ variable
When trigger chạy và resolved value là empty string hoặc null
Then hệ thống xử lý như Search Term không có giá trị
And fire cho tất cả events thuộc calendar scope

**AC.05.** Search Term matching is case-insensitive

Given Search Term có giá trị resolve
When trigger so sánh search term với event title
Then matching không phân biệt hoa thường (case-insensitive)
And chỉ cần substring của title chứa search term là match

**Edge Cases:**
- Search Term match nhiều events trong cùng polling window: fire workflow một lần cho mỗi event match.

---

### Story 4: Configure polling interval

**Story:** As an automation workflow builder, I want to chọn how often the trigger checks for ended events, so that workflow fires với tần suất phù hợp với use case.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01.** Polling Interval is required

Given Polling Interval để trống
When user click Continue hoặc Save
Then hiển thị validation error: "Polling Interval is required."

**AC.02.** Polling Interval dropdown options

Given user mở dropdown Polling Interval
When options được load
Then hệ thống hiển thị các options:

| UI Label | API value (minutes) |
|---|---|
| 1 minute | 1 |
| 5 minutes | 5 |
| 15 minutes | 15 |
| 30 minutes | 30 |
| 1 hour | 60 |

**AC.03.** Polling Interval defaults to 15 minutes

Given user chưa thay đổi Polling Interval
When trigger được save
Then hệ thống sử dụng 15 minutes làm giá trị mặc định

**AC.04.** Polling Interval does not support expressions

Given field Polling Interval là dropdown cố định
Then field này không hỗ trợ expression hoặc dynamic variable
And user chỉ có thể chọn từ danh sách cố định

---

### Story 5: Detect ended events and fire workflow

**Story:** As an automation workflow builder, I want trigger fire workflow khi event kết thúc, so that downstream steps có thể xử lý ngay sau khi event end.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01.** Polling query uses end_time window

Given trigger được activate
When mỗi polling tick chạy
Then hệ thống kiểm tra các events có end_time rơi vào khoảng (last_poll_time, current_poll_time]

**AC.02.** Filter by Calendar — All my Calendars

Given user chọn All my Calendars
When polling query chạy
Then hệ thống query tất cả accessible calendars
And excludes holiday calendars và birthday calendars

**AC.03.** Filter by Specific Calendars

Given user chọn Specific calendars mode
When polling query chạy
Then hệ thống chỉ query các calendars được chọn trong Specific Calendars

**AC.04.** Filter by Search Term

Given Search Term có giá trị resolve
When polling query chạy
Then hệ thống chỉ fire cho events có title (summary) chứa search term

**Edge Cases:**
- Event bị shorten làm end_time rơi vào quá khứ (đã qua) nhưng vẫn nằm trong polling window: fire workflow vì end_time đang ở trong window.
- Event bị extend làm end_time vượt qua polling window hiện tại: không fire trong poll này; sẽ fire khi end_time mới rơi vào polling window tương lai.
- Event bị edit nhiều lần trong cùng polling window: trigger chỉ fire một lần cho event đó.

---

### Story 6: Output payload cho downstream steps

**Story:** As an automation workflow builder, I want trigger trả về detailed event information, so that downstream steps có thể xử lý event data hiệu quả.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01.** Output contains full event information

Given trigger fires for an ended event
When downstream steps access the output
Then output trả về các fields mô tả tại mục Output

**Edge Cases:**
- Event không có description, location, hoặc attendees: các fields tương ứng trả về null hoặc empty array.

---

### Story 7: Handle authentication and permission errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01.** OAuth token expired or revoked

Given the connected account's OAuth token has expired or been revoked
When the trigger attempts to poll
Then the trigger fails with: "Authentication failed. Please reconnect your Google account."
And the polling tick is marked as failed

**AC.02.** Calendar not found

Given a selected calendar ID does not exist or has been deleted
When the trigger polls
Then the trigger surfaces: "Calendar not found. It may have been deleted or moved."
And the polling tick continues for remaining calendars in scope

---

## Out of Scope

- Trigger cho event creation, update, hoặc deletion — đã cover bởi trigger "New Event" và "New or Updated Event".
- Trigger fire trước khi event end (pre-end notification) — không thuộc semantic "Event Ends".
- Filter theo event properties khác title (description, location, attendees, organizer, status) — chỉ Search Term filter trên title.
- Expand Recurring Events option — recurring event instances fire individually theo end_time của từng instance, không cần option riêng.
- Custom polling interval ngoài preset — chỉ hỗ trợ 1, 5, 15, 30 phút và 1 giờ.
- Instant webhook-based detection — trigger này theo polling, không dùng push notifications.
- Historical event processing — events kết thúc trước khi trigger activate không được fire.
- Batch processing nhiều ended events trong một workflow execution — fire một workflow cho mỗi event.

---

## Inputs

| Field | Type | Required | Default | Static Selection | Manual Input | Mapped Variable | Notes |
|---|---|---|---|---|---|---|---|
| Account | Connected Account | Yes | None | Yes | No | No | Dùng lại logic Connected Account hiện có của Google Calendar integration. |
| Calendar | Dropdown | Yes | All my Calendars | Yes | No | No | Options: xem Story 2, AC.02. |
| Specific Calendars | Multi-select calendar picker | Conditional | None | Yes | No | No | Required khi mode = Specific calendars. Calendar selector dùng lại picker hiện có của Google Calendar integration. |
| Search Term | Text input | No | None | No | Yes | Yes | Free-text filter on event title (summary) only. Empty = no filter. |
| Polling Interval | Dropdown | Yes | 15 minutes | Yes | No | No | Options: xem Story 4, AC.02. Không support expression. |

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| event_id | string | ID của event vừa kết thúc | No |
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
| detected_at | string | Thời điểm trigger detect event ended theo ISO 8601 | No |
| trigger_mode | string | Fixed value: polling | No |

---

## Business Rules

1. Account, Calendar, và Polling Interval là bắt buộc.
2. Calendar mặc định là All my Calendars.
3. Specific Calendars là required khi Calendar = Specific calendars.
4. Khi mode = All my Calendars, hệ thống monitors tất cả accessible calendars ngoại trừ holiday calendars và birthday calendars.
5. Khi mode = Specific calendars, hệ thống chỉ monitors các calendars được chọn (excludes holiday và birthday calendars trong selector).
6. Polling Interval mặc định là 15 phút.
7. Polling Interval chỉ hỗ trợ preset values: 1, 5, 15, 30 phút và 1 giờ.
8. Polling Interval không support expression hoặc dynamic variable.
9. Search Term là optional; khi không có giá trị, trigger fire cho tất cả events thuộc calendar scope.
10. Search Term chỉ filter trên summary (title) của event, không filter trên các fields khác.
11. Search Term matching không phân biệt hoa thường (case-insensitive) và áp dụng substring match.
12. Search Term support cả manual input và expression.
13. Trigger fires cho mỗi event có end_time rơi vào polling window (last_poll_time, current_poll_time].
14. Trigger fires một lần cho mỗi event matching điều kiện.
15. Khi trigger được activate lần đầu, polling window bắt đầu từ thời điểm activation. Events kết thúc trước thời điểm activation không được xử lý.

---

## Error Handling

| Scenario | Behavior / Message |
|---|---|
| Account trống | Account is required. |
| Calendar trống | Calendar is required. |
| Specific Calendars trống khi mode = Specific calendars | Specific Calendars is required. |
| Polling Interval trống | Polling Interval is required. |
| Lỗi authentication | Authentication failed. Please reconnect your Google account. |
| Không đủ quyền truy cập calendar | Cannot read calendar: the connected account does not have read access to this calendar. Check the sharing settings. |
| Calendar không tồn tại hoặc mất quyền | Calendar not found. It may have been deleted or moved. |
| Rate limit | Sử dụng rule của hệ thống. |
| Network / timeout | Dùng lại rule của hệ thống. |
| Lỗi provider khác | Google Calendar API returned an unexpected error. Please try again later. |

---

## Assumptions

- Google Calendar API events.list với singleEvents=true trả về đầy đủ recurring instances trong time range với end_time chính xác cho từng instance.
- Platform có scheduler infrastructure để chạy polling theo các interval preset (1, 5, 15, 30 phút, 1 giờ).
- Connected Account đã được authenticate với đủ scopes để read calendar events.
