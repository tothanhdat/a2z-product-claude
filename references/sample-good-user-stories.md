# Sample Good User Stories (Delta Tickets)

## Mục đích

File này chứa các ví dụ delta ticket chất lượng tốt để skill `write-user-story` tham chiếu khi viết.

**Tiêu chí "good" của user story:**
- Epic Summary ngắn gọn, rõ parent ticket / sprint gốc.
- Delta-only: không mô tả lại Connect step, Account field, hay behavior đã tồn tại.
- Mỗi AC là một observable behavior, dùng Given/When/Then chuẩn.
- Dropdown fields có embed table (UI Label + API Value).
- Validation real-time: dùng "ngay", Continue bị disabled, không click-to-validate.
- Out of Scope liệt kê rõ những gì user có thể tưởng là in-scope.

---

## Example 1 — Human Input – Web Form (Authentication)

> **Pattern này minh hoạ:** Delta ticket bổ sung tính năng bảo mật cho trigger đã có. Scope hẹp (1 story), không có Inputs/Outputs section riêng vì không có field output mới. AC.06 cover flow tạo credential inline.

---

## Epic Summary

**Epic Name:** Human Input – Web Form (Authentication)

**Goal:**
Bổ sung tính năng Authentication (Basic Auth) cho Web Form Trigger, để automation workflow builder kiểm soát được ai có thể truy cập và submit form.

**Context:**
Tiếp nối ticket MART-1866. Action Human Input web form đã có nền tảng từ sprint 12. Ticket này tập trung vào bảo mật truy cập form thông qua Basic Auth.

**Expected UI Fields:**
- Tab Advanced: Authentication (Dropdown)
- Tab Advanced: Basic Auth credential (Dropdown — conditional, hiển thị khi Authentication = Basic Auth)

---

## User Stories

### Story 1: Configure form authentication

**Story:** As an automation workflow builder, I want to configure authentication for the form, so that I can control who can access and submit the form.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Authentication dropdown options**
Given user đang ở tab Advanced,
When user mở dropdown "Authentication",
Then hệ thống hiển thị các options:

| UI Label | API Value |
|---|---|
| None | none |
| Basic Auth | basic_auth |

**AC.02. Authentication defaults to "None"**
Given user chưa thay đổi config,
When tab Advanced được mở,
Then hệ thống mặc định chọn "None"

**AC.03. None — form accessible without authentication**
Given Authentication = "None",
When respondent mở form URL,
Then form hiển thị ngay lập tức

**AC.04. Basic Auth — conditional credential field appears**
Given user chọn Authentication = "Basic Auth",
When UI cập nhật,
Then hệ thống hiển thị thêm dropdown field "Basic Auth *" với placeholder "Select credential",
And hiển thị nút "+ Create new credential"

**AC.05. Basic Auth — credential is required**
Given user chọn Authentication = "Basic Auth",
And chưa chọn credential,
Then the "Continue" button is disabled

**AC.06. Create new credential — form within panel**
Given user click "+ Create new credential",
When form mở ra,
Then hiển thị các fields: "Connection Name *", "Username *", "Password *" (masked),
And hiển thị nút "Connect",
And khi user chưa nhập đủ các thông tin required,
Then nút "Connect" bị disabled

**AC.07. Basic Auth — respondent sees authentication page**
Given Authentication = "Basic Auth",
When respondent mở form URL,
Then platform hiển thị trang authentication riêng biệt với form "Username", "Password" và nút "Sign In"

**AC.08. Basic Auth — successful authentication**
Given respondent nhập đúng credentials,
When click "Sign In",
Then redirect sang trang form chính

**AC.09. Basic Auth — failed authentication**
Given respondent nhập sai username hoặc password,
When click "Sign In",
Then respondent vẫn ở lại trang đăng nhập,
And hệ thống hiển thị thông báo lỗi: "Incorrect username or password. Please try again.",
And không redirect sang trang form chính

---

## Example 2 — Workflow Detail – Display Tags

> **Pattern này minh hoạ:** Delta ticket bổ sung UI display cho platform feature. Scope tối giản (1 story, 2 ACs). Không có Inputs/Outputs vì không có node field mới — chỉ là thay đổi UI rendering.

---

## Epic Summary

**Epic Name:** Workflow Detail – Display Tags

**Goal:**
Hiển thị tags của workflow trong trang Workflow Detail để user có thể nhanh chóng identify và categorize workflow.

**Context:**
Tags đã được hiển thị ngoài danh sách workflow nhưng chưa hiển thị trong Workflow Detail page. Ticket này bổ sung tags bên dưới workflow name, giúp user dễ nhận biết workflow thuộc category nào.

**Expected UI Fields:**
- Tags (badge display dưới workflow name)

---

## User Stories

### Story 1: Display workflow tags in Workflow Detail

**Story:** As a workflow builder, I want to see workflow tags displayed in the Workflow Detail page, so that I can quickly identify the workflow's category and purpose.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Tags display under workflow name**
Given workflow có tags,
When user mở Workflow Detail page,
Then tags hiển thị dưới workflow name,
And mỗi tag hiển thị dạng badge với màu sắc tương ứng

**AC.02. Workflow without tags — no tag section shown**
Given workflow không có tags,
When user mở Workflow Detail page,
Then không hiển thị tag section,
And layout vẫn hiển thị bình thường

**Edge Cases:**
- Workflow có tối đa 3 tags.
- Tag name quá dài: hiển thị full text, không truncate.
