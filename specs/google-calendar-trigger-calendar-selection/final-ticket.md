# User Story - Google Calendar Trigger - Calendar Selection

## Epic Summary

**Epic Name:** Google Calendar Trigger – Calendar Selection

**Goal:**
Cho phép user chọn một calendar cố định để monitor, so that trigger chỉ nhận notifications từ đúng calendar mong muốn.

**Context:**
Thay thế "Calendar Selection Mode" (dropdown All/Specific) và "Specific Calendars" (multi-select) bằng một trường "Calendar" duy nhất (single-select). Default là calendar đầu tiên trong danh sách API trả về. Holiday calendars và birthday calendars bị exclude khỏi danh sách.

**Expected UI Fields:**
- Calendar * (single-select, default = first calendar from API)

---

## User Stories

### Story 2: Configure calendar selection

**Story:** As an automation workflow builder, I want to chọn một calendar cố định để monitor, so that trigger chỉ nhận notifications từ calendar mong muốn.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Calendar field loads accessible calendars**
Given user mở dropdown Calendar,
When options được hiển thị,
Then hệ thống load danh sách accessible calendars của connected account,
And excludes holiday calendars và birthday calendars khỏi danh sách.

> Calendar selector dùng lại picker và calendar access logic hiện có của Google Calendar integration.

**AC.02. Calendar default is first calendar from API**
Given user chưa thay đổi Calendar,
When config panel được mở,
Then hệ thống tự động pre-select calendar đầu tiên trong danh sách API trả về

**AC.03. Single calendar selection only**
Given user mở dropdown Calendar,
When user chọn một calendar,
Then chỉ một calendar được selected (không hỗ trợ multi-select),
And user có thể thay đổi bằng cách chọn lại calendar khác trong dropdown

**Edge Cases:**
- Calendar đã chọn bị deleted hoặc revoked access sau khi config: trigger không fire, ghi log warning.
- User thay đổi Calendar khi trigger đang active: existing webhook stopped, new webhook registered với calendar mới.
- Connected account không có calendar nào sau khi exclude holiday/birthday calendars: dropdown hiển thị empty state với message "No calendars found for the connected account."

---

## Out of Scope

- **Multi-calendar monitoring** — không hỗ trợ chọn nhiều calendars đồng thời; defer sang ticket riêng nếu cần.
- **"All my Calendars" mode** — đã loại bỏ; không có option monitor toàn bộ calendars của account.
