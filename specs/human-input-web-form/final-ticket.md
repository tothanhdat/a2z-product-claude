# Ticket — Human Input – Web Form Trigger

## Epic Summary

**Epic Name:** Human Input – Web Form Trigger

**Goal:**
Cho phép automation workflow builder tạo một web form tuỳ chỉnh (title, description, các fields) và chia sẻ form URL cho người dùng bên ngoài. Khi form được submit, workflow tự động trigger và nhận toàn bộ dữ liệu submit làm input cho các downstream steps.

**Context:**
Trigger này không sử dụng external provider API — form được tạo và host hoàn toàn trên platform. User cấu hình form fields (label, type, required, placeholder, description, default value) trong Configure step. Hệ thống auto-generate 2 URLs: Draft URL (preview/test) và Publish URL (production — chỉ active khi workflow Published). Khi form submit, platform nhận data và trigger workflow execution. Node này không yêu cầu Connected Account vì là internal platform feature.

**Expected UI Fields:**
- Form URL (Draft + Publish — auto-generated, read-only)
- Title *
- Description
- Form Fields (dynamic list via Add button)
- Label *, Type *, Description, Placeholder, Default Value toggle, Required Field toggle
- Button Label
- Review Form (button)
- Tab Test: Test button, Retest button, Output display
- Tab Advanced: Response Mode *, Authentication *, Basic Auth credential (conditional — khi Authentication = Basic Auth)

---

## User Stories

### Story 1: Configure form title, description, submit button, and preview form

**Story:** As an automation workflow builder, I want to set the form title, description, submit button label, and preview the form in real-time, so that I can see exactly how the form will look before testing or publishing.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Title is required**
Given user đang ở Configure step của trigger Web Form,
And field Title để trống sau khi đã touched,
Then nút Continue bị disabled,
And inline validation error hiển thị ngay dưới field: "Title is required."

**AC.02. Title supports manual input only**
Given user nhập giá trị text vào field Title,
Then hệ thống lưu giá trị static đã nhập

Note: Trigger là node đầu tiên trong workflow — không có upstream step, do đó Title không support Expression/Dynamic Variable.

**AC.03. Description is optional**
Given user không nhập Description,
When user click Save,
Then hệ thống save thành công mà không hiển thị validation error cho field này

**AC.04. Description supports manual input only**
Given user nhập giá trị text vào field Description,
Then hệ thống lưu giá trị static đã nhập

Note: Tương tự Title, Description không support Expression vì trigger không có upstream step.

**AC.05. Button Label default is "Submit"**
Given user chưa thay đổi Button Label,
When form được render,
Then nút submit hiển thị label "Submit"

**AC.06. Button Label supports manual input**
Given user nhập giá trị khác vào Button Label (ví dụ: "Send", "Register"),
When form được render,
Then nút submit hiển thị đúng label user đã nhập

**AC.07. Review Form button displays below Button Label field**
Given user đang ở Configure step,
When UI renders,
Then button "Review Form" hiển thị bên dưới field Button Label

**AC.08. Review Form opens side panel preview**
Given user click button "Review Form",
When side panel opens,
Then hệ thống hiển thị preview của form theo đúng config hiện tại (Title, Description, Form Fields, Button Label),
And preview form chỉ để xem, không cho phép điền giá trị hoặc submit. chỉ xem và cho phép nhập giá trị, không cho phép submit.

**AC.09. Preview form updates in real-time**
Given side panel preview đang mở,
When user thay đổi bất kỳ config nào (Title, Description, Form Fields, Button Label),
Then preview form tự động update ngay lập tức để reflect changes

**AC.10. Preview form closes when user clicks outside or close button**
Given side panel preview đang mở,
When user click nút close hoặc click outside panel,
Then side panel đóng lại,
And user quay về Configure step

**Edge Cases:**
- Button Label để trống → hệ thống dùng default "Submit".
- Title chứa HTML tags → tags được escape và hiển thị as plain text trên form (không render HTML).
- Preview form side panel mở trong lúc user đang edit field → preview update real-time, không đóng panel.

---

### Story 2: Configure form fields

**Story:** As an automation workflow builder, I want to add and configure multiple form fields with different types, so that the form collects exactly the data I need from respondents.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Add field button creates a new field entry**
Given user đang ở Configure step,
When user click nút "Add",
Then hệ thống thêm một form field entry mới với các sub-fields: Label, Type, Description, Placeholder, Default Value toggle, Required Field toggle

**AC.02. Label is required for each field**
Given user đã thêm một form field,
And field Label để trống sau khi đã touched,
Then nút Continue bị disabled,
And inline validation error hiển thị ngay dưới field: "Label is required."

**AC.03. Type is required for each field — dropdown options**
Given user mở dropdown Type của một form field,
When options được hiển thị,
Then hệ thống hiển thị các option như design

**AC.04. Type is required — validation**
Given field Type chưa được chọn,
Then nút Continue bị disabled,
And inline validation error hiển thị ngay dưới field: "Type is required."

**AC.05. Description is optional per field**
Given user không nhập Description cho một form field,
When form được render,
Then field đó hiển thị mà không có helper text phía dưới

**AC.06. Placeholder is optional per field**
Given user nhập Placeholder cho form field,
When form được render,
Then input field hiển thị placeholder text tương ứng

**AC.07. Default Value toggle — shows input when enabled**
Given user bật toggle Default Value cho một form field có Type = Text Input, Text Area, Email, Number, hoặc Phone,
When toggle chuyển sang On,
Then hệ thống hiển thị thêm text input để user nhập giá trị mặc định

**AC.07a. Default Value toggle hidden for Dropdown, Checkbox, and Radio**
Given user chọn Type = Dropdown, Checkbox, hoặc Radio cho một form field,
When UI updates,
Then toggle Default Value không hiển thị cho field đó

**AC.07b. Default Value toggle for DateTime — shows date/time and timezone inputs when enabled**
Given user bật toggle Default Value cho một form field có Type = DateTime,
When toggle chuyển sang On,
Then hệ thống hiển thị thêm 2 inputs: Date Time picker và Timezone selector,
And cả 2 inputs tái sử dụng component Date Time và Timezone picker có sẵn của hệ thống

**AC.07c. Default Value toggle for DateTime — hides inputs when disabled**
Given user tắt toggle Default Value trên field có Type = DateTime,
When toggle chuyển sang Off,
Then Date Time picker và Timezone selector bị ẩn,
And giá trị đã chọn trước đó bị clear

**AC.08. Default Value toggle — hides input when disabled**
Given user tắt toggle Default Value,
When toggle chuyển sang Off,
Then input default value bị ẩn,
And giá trị default trước đó bị clear

**AC.09. Default Value renders on form**
Given user đã bật toggle Default Value và nhập giá trị,
When form được render cho người điền,
Then field đó hiển thị giá trị mặc định đã cấu hình,
And người điền form có thể thay đổi giá trị này

**AC.10. Required Field toggle behavior**
Given user bật toggle Required Field cho một form field,
When form được render,
Then field label hiển thị dấu "*" kế bên,
And form không cho phép submit nếu field đó để trống

**AC.11. Required Field toggle default is Off**
Given user thêm form field mới,
When toggle Required Field chưa được thay đổi,
Then mặc định là Off (field không bắt buộc)

**AC.12. Maximum 20 fields per form**
Given form đã có 20 fields,
When user click "Add",
Then nút "Add" bị disabled hoặc hệ thống hiển thị message: "Maximum number of fields reached (20)."

**AC.13. User can reorder form fields**
Given form có nhiều fields,
When user drag-and-drop một field,
Then thứ tự fields được cập nhật,
And form render theo thứ tự mới

**AC.14. User can delete a form field**
Given form có ít nhất 1 field,
When user click nút delete trên một field entry,
Then field đó bị xoá khỏi danh sách,
And các fields còn lại giữ nguyên cấu hình

**AC.15. Form must have at least 1 field**
Given form không có field nào,
Then nút Continue bị disabled,
And inline validation error hiển thị ngay phía trên nút "Add": "At least one form field is required."

**Edge Cases:**
- Duplicate Label values giữa các fields → hệ thống cho phép (không block), nhưng output keys sẽ được deduplicate bằng cách append suffix (_1, _2, ...). Ví dụ: email, email_1, email_2

---

### Story 3: Configure Dropdown, Checkbox, Radio options

**Story:** As an automation workflow builder, I want to define custom options for Dropdown, Checkbox and Radio fields, so that respondents can select from predefined choices.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Dropdown type shows options configuration**
Given user chọn Type = Dropdown cho một form field,
When UI updates,
Then hệ thống hiển thị khu vực cấu hình options,
And cho phép user thêm nhiều option values

**AC.02. Checkbox type shows options configuration**
Given user chọn Type = Checkbox cho một form field,
When UI updates,
Then hệ thống hiển thị khu vực cấu hình options,
And cho phép user thêm nhiều option values

**AC.03. Radio type shows options configuration**
Given user chọn Type = Radio cho một form field,
When UI updates,
Then hệ thống hiển thị khu vực cấu hình options,
And cho phép user thêm nhiều option values

**AC.04. At least 1 option is required for Dropdown**
Given user chọn Type = Dropdown,
And không thêm option nào,
Then nút Continue bị disabled,
And inline validation error hiển thị ngay dưới khu vực options: "At least one option is required for Dropdown field."

**AC.05. At least 1 option is required for Checkbox**
Given user chọn Type = Checkbox,
And không thêm option nào,
Then nút Continue bị disabled,
And inline validation error hiển thị ngay dưới khu vực options: "At least one option is required for Checkbox field."

**AC.06. At least 1 option is required for Radio**
Given user chọn Type = Radio,
And không thêm option nào,
Then nút Continue bị disabled,
And inline validation error hiển thị ngay dưới khu vực options: "At least one option is required for Radio field."

**AC.07. Maximum 10 options per Dropdown, Checkbox, Radio field**
Given field Type = Dropdown hoặc Checkbox hoặc Radio đã có 10 options,
When user cố thêm option mới,
Then nút "Add option" bị disabled hoặc hệ thống hiển thị: "Maximum number of options reached (10)."

**AC.08. Each option requires a value**
Given user thêm option nhưng để trống giá trị,
Then nút Continue bị block,
And hiển thị inline error: "Option value cannot be empty."

**AC.09. Options hidden when type changes away from Dropdown/Checkbox/Radio**
Given user đã cấu hình options cho Dropdown field,
When user đổi Type sang Text Input (hoặc type khác không phải Dropdown/Checkbox/Radio),
Then khu vực options bị ẩn,
And dữ liệu options trước đó bị clear

**Edge Cases:**
- Duplicate option values trong cùng field → hệ thống cho phép nhưng form render sẽ hiển thị duplicate options.
- Option value chứa only whitespace → treat as empty, validation error: "Option value cannot be empty."

---

### Story 4: Test form with Draft URL and capture test data

**Story:** As an automation workflow builder, I want to test the form by opening Draft URL in a new tab and capturing submission data, so that I can verify the form works correctly before publishing.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Draft URL generated when trigger node is created**
Given user khởi tạo trigger Web Form,
Then hệ thống auto-generate Draft URL và hiển thị sẵn trong UI,
And user có thể copy URL và không thể edit URL

**AC.02. Publish URL generated when trigger node is created**
Given user khởi tạo trigger Web Form,
Then hệ thống auto-generate Publish URL và hiển thị sẵn trong UI,
And user có thể copy URL và không thể edit URL

**AC.03. Test button available in Test tab**
Given user mở tab Test của trigger node,
When tab renders,
Then hệ thống hiển thị button "Test" và button "Retest" (disabled nếu chưa test lần nào)

**AC.04. Test button opens Draft URL in new tab**
Given user click button "Test" trong tab Test,
When action executes,
Then hệ thống mở Draft URL trong tab mới của browser,
And Draft URL hiển thị form theo config tại thời điểm click Test,
And label của button "Test" được thay thành "Testing" kèm loading icon (tái sử dụng behavior loading button hiện có)

**AC.05. Test button in "Testing" state while waiting for submission**
Given button "Test" đang hiển thị trạng thái "Testing",
When user chưa submit form trên Draft URL tab,
Then button tiếp tục giữ trạng thái "Testing" với loading icon,
And user không thể click button trong trạng thái này

**AC.06. Draft URL form submission captures test data**
Given user điền và submit form trên Draft URL tab,
When submission thành công,
Then form page hiển thị message: "Thank you! Your response has been recorded Your response has been submitted successfully. You can now close this page.",
And form không cho phép submit lại,
And submission data được capture và hiển thị trong tab Output của trigger node,
And button "Test" trở về label "Test" và trạng thái normal

**AC.07. Test data displayed in Output tab**
Given submission data đã được capture,
When user mở tab Output của trigger node,
Then hệ thống hiển thị submission data với đầy đủ field values và metadata (submission_id, submitted_at, form_url, form_title),
And user có thể sử dụng data này để test downstream steps

**AC.08. Test timeout after 5 minutes**
Given button "Test" đang hiển thị trạng thái "Testing",
And user không submit form trong vòng 5 phút,
When timeout xảy ra,
Then button "Test" trở về label "Test" và trạng thái normal,
And hiển thị message trong tab Test Toast Timeout: "Test timed out. Please click Retest to try again." "No response was received within the timeout period. Please try again."

**AC.09. Retest button enabled after first test**
Given user đã click "Test" ít nhất một lần,
When tab Test renders,
Then button "Retest" được enabled

**AC.10. Retest button opens updated Draft URL**
Given user đã thay đổi form config sau lần test trước,
When user click button "Retest",
Then hệ thống mở Draft URL trong tab mới với config mới nhất,
And button "Retest" chuyển sang trạng thái loading,
And behavior tương tự AC.05–AC.08

**AC.11. Draft URL only updates when Test/Retest is clicked**
Given user thay đổi form config (Title, Fields, etc.),
And chưa click Test hoặc Retest,
When bất kỳ ai mở Draft URL,
Then form vẫn hiển thị theo config của lần Test/Retest gần nhất,
And config changes chưa được reflect trên Draft URL

**AC.12. Publish URL only active when workflow is Published**
Given workflow chưa được Publish,
When người dùng mở Publish URL,
Then hiển thị message: "This form is not available yet."

**AC.13. Publish URL active after workflow Published**
Given workflow đã được Publish,
When người dùng mở Publish URL,
Then form hiển thị đúng theo config tại thời điểm workflow được Publish,
And submit trên Publish URL sẽ trigger workflow execution

**AC.14. Publish URL updates when workflow is Re-published**
Given workflow đã Published và user thay đổi form config,
When user Re-publish workflow,
Then Publish URL update để reflect config mới,
And form submissions sau đó sử dụng config mới

**Edge Cases:**
- User đóng Draft URL tab trước khi submit → button Test/Retest vẫn loading cho đến timeout (5 phút).
- User click Test nhiều lần liên tiếp → mỗi lần click mở tab mới, nhưng chỉ submission gần nhất được capture vào Output.
- User copy Draft URL và mở trực tiếp trên browser mà chưa click Test lần nào → form hiển thị error page với message: "Form preview is not ready. Please return to the workflow builder and click Test to load this form." Không hiển thị form.
- Workflow bị Unpublish → Publish URL trở lại trạng thái "This form is not available yet.", Draft URL vẫn hoạt động bình thường.

---

### Story 5: Form submission and workflow trigger on Publish URL

**Story:** As an automation workflow builder, I want the workflow to trigger automatically when someone submits the form on Publish URL, so that submitted data flows into downstream steps for processing.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Form validates required fields before submit**
Given form có fields được đánh dấu Required (toggle On),
When người điền form để trống required field và click Submit,
Then form hiển thị inline validation error: "{Field Label} is required.",
And form không được submit

**AC.02. Email field validation on form**
Given form field có Type = Email,
When người điền form nhập giá trị không đúng email format và click Submit,
Then form hiển thị validation error: "Please enter a valid email address (e.g., user@example.com)."

**AC.03. Number field validation on form**
Given form field có Type = Number,
When người điền form nhập giá trị không phải số và click Submit,
Then form hiển thị validation error: "Please enter a valid number (e.g., 42)."

**AC.04. Phone field validation on form**
Given form field có Type = Phone,
When người điền form nhập giá trị không đúng phone format và click Submit,
Then form hiển thị validation error: "Please enter a valid phone number."

**AC.05. Successful form submission on Publish URL triggers workflow**
Given form được submit thành công trên Publish URL,
When platform nhận form data,
Then workflow execution được trigger,
And toàn bộ form field values được truyền vào execution làm input cho downstream steps

**AC.06. Output payload contains all submitted field values**
Given workflow được trigger bởi form submission trên Publish URL,
When downstream steps đọc trigger output,
Then output bao gồm tất cả field values dạng key-value pairs,
And kèm metadata: submission_id, submitted_at, form_url, form_title

**AC.07. Multiple submissions trigger multiple executions**
Given workflow đang active,
When nhiều người submit form cùng lúc hoặc liên tiếp trên Publish URL,
Then mỗi submission trigger một workflow execution riêng biệt

**AC.08. Draft URL submission does not trigger workflow**
Given người dùng submit form trên Draft URL,
When submission thành công,
Then workflow execution không được trigger,
And data chỉ được capture vào tab Output của trigger node để test

**Edge Cases:**
- Form data chứa very long text (Text Area) → accept.
- Concurrent submissions trên Publish URL → mỗi submission tạo execution riêng, không ảnh hưởng lẫn nhau.

---

### Story 6: Response Mode configuration for Publish URL

**Story:** As an automation workflow builder, I want to configure when the form respondent receives a response on Publish URL, so that I can control the user experience after form submission.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Response Mode is required — dropdown options**
Given user mở tab Advanced,
When user mở dropdown Response Mode,
Then hệ thống hiển thị các options:

| UI Label | API value | Description |
|---|---|---|
| Form Is Submitted | form_submitted | As soon as this node receives the form submission |
| Workflow Finishes | workflow_finishes | When the last node of the workflow is executed |

**AC.02. Response Mode default is "Form Is Submitted"**
Given user chưa thay đổi Response Mode,
Then hệ thống sử dụng Form Is Submitted làm giá trị mặc định

**AC.03. Response Mode only applies to Publish URL**
Given Response Mode được cấu hình,
When form được submit trên Draft URL,
Then Response Mode setting không áp dụng,
And form page luôn hiển thị fixed message: "Thank you! Your response has been recorded."

**AC.04. "Form Is Submitted" — immediate response to respondent**
Given Response Mode = "Form Is Submitted",
When người điền form submit thành công trên Publish URL,
Then form page hiển thị ngay lập tức: "Thank you! Your response has been recorded.",
And không đợi workflow execution hoàn tất

**AC.05. "Workflow Finishes" — immediate progress feedback after submit**
Given Response Mode = "Workflow Finishes",
When người điền form submit thành công trên Publish URL,
Then form page hiển thị ngay lập tức message: "Your response has been submitted. Please wait while we process your request...",
And kèm animated progress indicator (spinner hoặc progress bar),
And form page đợi cho đến khi workflow execution hoàn tất hoặc timeout

**AC.06. "Workflow Finishes" — success result**
Given Response Mode = "Workflow Finishes",
And workflow execution hoàn tất thành công trong khoảng thời gian timeout,
When form page nhận kết quả,
Then hiển thị message: "All done! Your request has been processed successfully."

**AC.07. "Workflow Finishes" — failure result**
Given Response Mode = "Workflow Finishes",
And workflow execution thất bại trong khoảng thời gian timeout,
When form page nhận kết quả,
Then hiển thị message: "Something went wrong while processing your request. Please try again later or contact support."

**AC.08. "Workflow Finishes" — graceful timeout fallback**
Given Response Mode = "Workflow Finishes",
And workflow execution chưa hoàn tất sau 5 phút (default timeout),
When timeout xảy ra,
Then form page hiển thị message: "Your response has been submitted successfully. Processing is taking longer than usual. You may close this page.",
And workflow execution vẫn tiếp tục chạy ngầm phía backend,
And message này không phải error — data đã được nhận thành công

**Edge Cases:**
- Người dùng đóng tab trước khi nhận response → workflow execution vẫn tiếp tục chạy, không bị cancel.

---

### Story 7: Authentication configuration

**Story:** As an automation workflow builder, I want to configure authentication for the form, so that I can control who can access and submit the form.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Authentication is required — dropdown options**
Given user mở tab Advanced,
When user mở dropdown Authentication,
Then hệ thống hiển thị các options:

| UI Label | API value |
|---|---|
| None | none |
| Basic Auth | basic_auth |

**AC.02. Authentication default is "None"**
Given user chưa thay đổi Authentication,
Then hệ thống sử dụng None làm giá trị mặc định

**AC.03. None — form accessible without authentication**
Given Authentication = "None",
When respondent mở form URL (Draft hoặc Publish),
Then form hiển thị ngay lập tức mà không yêu cầu đăng nhập

Note: Authentication setting áp dụng cho cả Draft URL và Publish URL.

**AC.04. Basic Auth — conditional credential field appears**
Given user chọn Authentication = "Basic Auth",
When UI updates,
Then hệ thống hiển thị thêm dropdown field Basic Auth với placeholder "Select credential",
And dropdown liệt kê các credentials đã tạo (dạng: {Connection Name} — Basic Auth),
And hiển thị nút "+ Create new credential" cuối danh sách

**AC.05. Basic Auth — credential is required**
Given user chọn Authentication = "Basic Auth",
And chưa chọn credential từ dropdown Basic Auth,
Then nút Continue bị disabled,
And inline validation error hiển thị ngay dưới field: "Basic Auth is required."

**AC.06. Basic Auth — credential field hidden when switching to None**
Given user đã chọn Authentication = "Basic Auth" và chọn credential,
When user đổi Authentication sang "None",
Then dropdown Basic Auth bị ẩn,
And giá trị credential đã chọn bị clear

**AC.07. Create new credential — form within panel**
Given user click "+ Create new credential" trong dropdown Basic Auth,
When form "Connect credential" mở ra trong cùng panel,
Then hệ thống hiển thị các fields:

| Field | Type | Required | Placeholder |
|---|---|---|---|
| Connection Name | Text input | Yes | Enter Credential Name |
| Username | Text input | Yes | Enter Username |
| Password | Text input (masked) | Yes | Enter Password |

And hiển thị nút "Connect" để lưu credential

**AC.08. Create new credential — validation**
Given form "Connect credential" đang mở,
And một hoặc nhiều required fields (Connection Name, Username, Password) để trống sau khi touched,
Then nút Connect bị disabled,
And inline validation error hiển thị ngay dưới field trống: "{Field Name} is required."

**AC.09. Create new credential — success**
Given user đã nhập đầy đủ Connection Name, Username, Password,
When user click "Connect",
Then credential được lưu thành công,
And dropdown Basic Auth tự động chọn credential vừa tạo,
And form "Connect credential" đóng lại

**AC.10. Basic Auth — respondent sees custom authentication page**
Given Authentication = "Basic Auth" với credential đã cấu hình,
When respondent mở form URL (Draft hoặc Publish),
Then platform hiển thị một trang authentication riêng biệt (không phải native browser dialog),
And trang này có form với 2 fields: Username và Password, kèm nút "Sign In"

**AC.11. Basic Auth — successful authentication**
Given respondent nhập đúng Username và Password trên trang authentication,
When respondent click "Sign In",
Then platform xác thực thành công,
And redirect respondent sang trang form để điền và submit bình thường

**AC.12. Basic Auth — failed authentication**
Given respondent nhập sai Username hoặc Password trên trang authentication,
When respondent click "Sign In",
Then trang authentication hiển thị inline error: "Incorrect username or password. Please try again.",
And các fields Username và Password được giữ nguyên để respondent nhập lại,
And form không bị redirect

**AC.13. Basic Auth — respondent leaves without authenticating**
Given respondent mở trang authentication,
When respondent đóng tab hoặc navigate away mà không sign in,
Then không có action nào được thực hiện,
And không có execution nào được trigger

**Edge Cases:**
- User xoá credential đang được sử dụng → dropdown Basic Auth hiển thị trạng thái trống, nút Continue bị block cho đến khi user chọn credential khác hoặc tạo mới.
- Respondent nhập sai nhiều lần → browser tiếp tục hiển thị auth dialog cho đến khi respondent cancel hoặc nhập đúng (standard HTTP Basic Auth behavior).

---

### Story 8: Handle form and execution errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Form submission fails due to server error**
Given form được submit thành công nhưng platform gặp lỗi khi tạo execution,
When error xảy ra,
Then form page hiển thị: "Something went wrong. Please try again later."

**AC.02. Form submission rate limiting**
Given quá nhiều submissions trong thời gian ngắn,
When platform detect rate limit,
Then form page hiển thị: "Too many submissions. Please wait a moment and try again."

---

## Out of Scope

- **File upload field type** — không hỗ trợ upload file trong form.
- **Conditional logic giữa các fields** — field visibility không phụ thuộc vào giá trị field khác.
- **Multi-page / multi-step form** — form chỉ có 1 page.
- **Custom thank-you page hoặc redirect URL** — dùng message mặc định.
- **Form analytics / submission history** — không tracking số lượng submissions hoặc view history.
- **Custom CSS / theme cho form** — form dùng default platform styling.
- **CAPTCHA / bot protection** — không có anti-spam mechanism trên form.
- **Form expiration / scheduling** — form không có ngày hết hạn riêng.
- **Respond on UI action node** — ticket riêng.
- **Chat UI trigger** — ticket riêng.

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| execution_id | string | ID duy nhất của mỗi làn execute | No |
| submitted_at | string | Thời điểm form được submit theo ISO 8601 | No |
| form_url | string | URL của form đã được submit | No |
| form_title | string | Title của form tại thời điểm submit | No |
| fields | object | Key-value pairs của tất cả field values đã submit. | No |
| trigger_mode | string | Fixed: instant | No |

---

## Business Rules

1. Trigger này là internal platform feature — không yêu cầu Connected Account.
2. Form phải có 1-20 fields. Mỗi field bắt buộc có Label và Type.
3. Type = Dropdown/Checkbox/Radio bắt buộc có 1-10 options.
4. Hệ thống auto-generate 2 URLs (Draft + Publish) khi khởi tạo trigger.
5. Draft URL chỉ update khi user click Test/Retest — không tự động reflect config changes.
6. Publish URL chỉ active khi workflow Published và update khi Re-publish.
7. Draft URL submission không trigger workflow — chỉ capture data vào tab Output để test.
8. Mỗi submission trên Publish URL trigger một workflow execution riêng biệt.
9. Response Mode "Workflow Finishes" có timeout mặc định 5 phút — timeout là graceful fallback, workflow vẫn chạy ngầm.
10. Authentication setting áp dụng cho cả Draft URL và Publish URL.
11. Output field keys được generate từ field Label: bỏ dấu tiếng Việt, lowercase, thay spaces bằng underscore, loại bỏ ký tự đặc biệt. Ví dụ: "Họ và tên" → "ho_va_ten".
12. Duplicate field labels được deduplicate bằng suffix (_1, _2, ...).
13. Output value format: Dropdown/Radio → string, Checkbox → array, Number → number, Date Time → datetime.
14. Title và Description không support Expression vì trigger không có upstream step.

---

## Validation Rules

### Config-time Validation (Workflow Builder UI)

| Scenario | Message |
|---|---|
| Title empty | "Title is required." |
| Form has no fields | "At least one form field is required." |
| Field Label empty | "Label is required." |
| Field Type not selected | "Type is required." |
| Type = Dropdown, no options | "At least one option is required for Dropdown field." |
| Type = Checkbox, no options | "At least one option is required for Checkbox field." |
| Type = Radio, no options | "At least one option is required for Radio field." |
| Option value empty | "Option value cannot be empty." |
| Form fields exceed 20 | "Maximum number of fields reached (20)." |
| Options exceed 10 per field | "Maximum number of options reached (10)." |
| Authentication = Basic Auth, no credential selected | "Basic Auth is required." |
| Create credential: Connection Name empty | "Connection Name is required." |
| Create credential: Username empty | "Username is required." |
| Create credential: Password empty | "Password is required." |

### Runtime Validation (Form Respondent Side)

| Scenario | Message |
|---|---|
| Required field empty on submit | "{Field Label} is required." |
| Email format invalid | "Please enter a valid email address (e.g., user@example.com)." |
| Number format invalid | "Please enter a valid number (e.g., 42)." |
| Phone format invalid | "Please enter a valid phone number." |
| Publish URL accessed when workflow not Published | "This form is not available yet." |
| Basic Auth — wrong credentials | "Incorrect username or password. Please try again." |

---

## Error Handling

| Error Scenario | When Detected | User-facing Message | Hard / Soft Fail | Retryable |
|---|---|---|---|---|
| Publish URL khi workflow chưa Published | Form load | "This form is not available yet." | Hard Fail (form not rendered) | No |
| Server error khi tạo execution | Form submit | "Something went wrong. Please try again later." | Hard Fail | Yes |
| Rate limit exceeded | Form submit | "Too many submissions. Please wait a moment and try again." | Hard Fail | Yes |
| Workflow execution timeout (Workflow Finishes mode) | Response wait (5 phút) | "Your response has been submitted successfully. Processing is taking longer than usual. You may close this page." | Soft Fail (workflow continues in background) | N/A — not an error |
| Workflow execution failed (Workflow Finishes mode) | Response wait | "Something went wrong while processing your request. Please try again later or contact support." | Hard Fail | No |
| Basic Auth — wrong credentials | Authentication page | "Incorrect username or password. Please try again." | Soft Fail (trang auth giữ nguyên) | Yes (re-enter credentials) |
| Basic Auth — respondent leaves without authenticating | Authentication page | No action taken | N/A | No |

---

## Assumptions

- Test button timeout mặc định 5 phút — nếu user không submit trong thời gian này, button stop loading.
- Publish URL update khi workflow được Publish hoặc Re-publish — reflect config tại thời điểm Publish.
- Draft URL test data — khi user submit nhiều lần trên Draft, tab Output lưu submission mới nhất.
- Platform có khả năng host và serve web forms tại auto-generated URLs.
- Form submissions không có file upload — chỉ text-based data.
- Output field keys được generate từ field Label theo quy tắc: (1) Bỏ dấu tiếng Việt (unicode normalization), (2) chuyển toàn bộ sang lowercase, (3) thay spaces bằng underscore, (4) loại bỏ ký tự đặc biệt ngoài chữ cái latin, số, và underscore. Ví dụ: "Họ và tên" → "ho_va_ten", "Email Address" → "email_address", "Số điện thoại" → "so_dien_thoai".
- Không giới hạn số lượng submissions trên một form (ngoài rate limiting).
- Platform có khả năng lưu Draft URL submission data và hiển thị trong tab Output của trigger node.
- Preview form side panel có khả năng render form real-time theo config changes.
