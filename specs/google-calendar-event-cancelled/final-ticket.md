## Epic Summary

**Epic Name:** Google Calendar – Event Cancelled

**Goal:**
Cho phép automation workflow builder tự động phát hiện khi một event trong Google Calendar bị huỷ, để trigger workflow xử lý hậu kỳ như notify attendees, cập nhật CRM/booking, hoặc reschedule.

**Context:**
Tiếp nối Google Calendar integration đã có (New or Updated Event đã Done). Ticket này bổ sung Trigger "Event Cancelled" — phát hiện event bị huỷ qua scheduled polling. Calendar selector reuse pattern của New or Updated Event.

**Expected UI Fields:**
- Calendar Selection Mode *
- Calendar (conditional — hiển thị khi chọn Specific calendars)

---

## User Stories

### Story 1: Connect account and select trigger

**Story:** As an automation workflow builder, I want to connect an account and confirm the trigger type, so that the Event Cancelled trigger runs under the correct authenticated identity.

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

**AC.03. Action * displays "Event Cancelled" as pre-selected value**
Given the user is configuring this node,
When the user views the "Action *" field,
Then the dropdown shows "Event Cancelled" already pre-selected,
And the user can change to other available triggers if needed

---

### Story 2: Configure calendar scope and detect cancelled events

**Story:** As an automation workflow builder, I want to chọn calendar cần theo dõi và nhận dữ liệu khi event bị huỷ, so that trigger chỉ fire đúng scope mong muốn và downstream steps có đủ thông tin để xử lý.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Calendar Selection Mode dropdown options**
Given user mở dropdown "Calendar Selection Mode",
When options được load,
Then hệ thống hiển thị các options sau:

| UI Label | API value | Description |
|---|---|---|
| All my calendars | all | Monitor tất cả calendars mà account có quyền truy cập |
| Specific calendars | specific | Chỉ monitor các calendars được chọn thủ công |

**AC.02. Calendar Selection Mode defaults to "All my calendars"**
Given user chưa thay đổi Calendar Selection Mode,
When Configure step được mở,
Then "All my calendars" hiển thị là giá trị mặc định

**AC.03. Calendar field appears when "Specific calendars" is selected**
Given user chọn "Specific calendars" trong Calendar Selection Mode,
When UI cập nhật,
Then trường "Calendar *" hiển thị phía dưới,
And trường này là required

**AC.04. Calendar is required when mode = Specific calendars**
Given Calendar Selection Mode = "Specific calendars",
And "Calendar *" chưa có calendar nào được chọn sau khi touched,
Then the "Continue" button is disabled,
And inline validation error hiển thị ngay dưới field: "At least one calendar must be selected when using specific calendars."

**AC.05. Calendar field is hidden when mode = All my calendars**
Given Calendar Selection Mode = "All my calendars",
When UI cập nhật,
Then trường "Calendar *" ẩn đi,
And không validate trường này

**AC.06. Trigger fires when a cancelled event is detected**
Given trigger được activate,
When polling detect event có status chuyển sang "cancelled" trong calendar scope,
Then trigger fires một lần cho event đó,
And output trả về các fields tại mục Outputs

**AC.07. Trigger does not fire for non-cancellation changes**
Given một event trong calendar scope bị thay đổi,
When thay đổi không phải cancellation (VD: attendee decline, chỉnh title/time),
Then trigger không fire

**Edge Cases:**
- Calendar được chọn bị xóa hoặc mất quyền truy cập sau khi config: trigger bỏ qua calendar đó, tiếp tục polling các calendars còn lại, surface error theo mục Error Handling.
- Event bị huỷ rồi restore rồi huỷ lại trong các polling window khác nhau: trigger fire cho mỗi lần huỷ được detect.

---

### Story 3: Handle authentication and permission errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. OAuth token expired or revoked**
Given the connected account's OAuth token has expired or been revoked,
When the trigger attempts to poll,
Then the trigger fails with: "Authentication failed. Please reconnect your Google Calendar account.",
And the polling tick is marked as failed

**AC.02. Insufficient permission on calendar**
Given the connected account does not have read access to a selected calendar,
When the trigger polls,
Then the trigger surfaces: "Cannot read calendar: the connected account does not have read access to this calendar. Check the sharing settings.",
And the polling tick continues for remaining calendars in scope

**AC.03. Calendar not found**
Given a selected calendar no longer exists or access has been revoked,
When the trigger polls,
Then the trigger surfaces: "Calendar not found or access has been revoked.",
And the polling tick continues for remaining calendars in scope

---

## Inputs

| Field | Type | Required | Default | Mapped Variable | Notes |
|---|---|---|---|---|---|
| Connection | Connected Account | Yes | None | No | Dùng lại logic Connected Account hiện có. |
| Calendar Selection Mode | Dropdown | Yes | All my calendars | No | Options: xem AC.01 Story 2. |
| Calendar | Multi-select | Conditional | None | No | Required khi Calendar Selection Mode = Specific calendars. Calendar selector dùng lại picker và calendar access logic hiện có của Google Calendar integration. |

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| event_id | string | ID của event bị huỷ | No |
| calendar_id | string | ID của calendar chứa event | No |
| calendar_name | string | Tên calendar | No |
| summary | string | Tiêu đề event | Yes |
| start_at | string | Thời điểm bắt đầu (ISO 8601) | No |
| end_at | string | Thời điểm kết thúc (ISO 8601) | No |
| organizer_email | string | Email của organizer | Yes |
| organizer_name | string | Tên organizer | Yes |
| attendees | array | Danh sách attendees, mỗi item gồm email | Yes |
| status | string | Luôn là "cancelled" | No |
| was_recurring | boolean | `true` nếu là instance của recurring series | No |
| updated_at | string | Thời điểm event bị huỷ (ISO 8601) | No |
| timestamp | string | Thời điểm trigger phát hiện (ISO 8601) | No |
| trigger_mode | string | Luôn là "polling" | No |

---

## Business Rules

1. Calendar Selection Mode mặc định là "All my calendars".
2. Calendar selector dùng lại picker và calendar access logic hiện có của Google Calendar integration.
3. Khi mode = All my calendars: trigger monitor tất cả calendars mà account có quyền truy cập.
4. Khi mode = Specific calendars: trigger chỉ monitor các calendars đã chọn.
5. "Cancelled" bao gồm: (a) organizer xóa/huỷ toàn bộ event; (b) một occurrence đơn lẻ trong recurring series bị huỷ. Không bao gồm attendee decline.
6. Trigger không fire cho events bị huỷ trước thời điểm activation.
7. Polling interval theo platform standard.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Calendar Selection Mode = Specific calendars, không có calendar nào được chọn | Config-time | At least one calendar must be selected when using specific calendars. |
| OAuth token expired hoặc revoked | Runtime | Authentication failed. Please reconnect your Google Calendar account. |
| Không đủ quyền đọc calendar | Runtime | Cannot read calendar: the connected account does not have read access to this calendar. Check the sharing settings. |
| Calendar không tồn tại hoặc đã bị xóa | Runtime | Calendar not found or access has been revoked. |
| Google Calendar API không phản hồi / lỗi tạm thời | Runtime | Google Calendar is currently unavailable. Please try again later. |

---

## Out of Scope

- **Attendee decline** — thay đổi responseStatus không phải cancellation; không fire trigger.
- **Expand Recurring Events toggle** — trigger luôn fire cho cả series-level và occurrence-level cancellation, không có tuỳ chọn tắt.
- **Instant/webhook detection** — ticket này dùng polling; push notification sẽ xem xét riêng nếu cần.
- **Historical cancellations** — events bị huỷ trước thời điểm trigger activate không được xử lý.
- **Search Term / title filter** — không nằm trong scope; nếu cần filter có thể thêm Filter node downstream.
