# Ticket Template — Lean Style

## Mục đích

File này cung cấp skeleton chuẩn cho Lean Ticket. AI phải tuân theo structure này khi viết ticket cho action/trigger.

> **Scope:** Đây là default template cho action/trigger tickets. Cho non-integration feature tickets (platform features, UI features), refer to `ba-behavier.md` Section 6.

---


## Epic Summary Block

```
## Epic Summary

**Epic Name:** [Platform] – [Action Name]

**Goal:**
[1–2 câu. WHO can do WHAT and WHY. Business language, không technical detail.]

**Context:**
[Business context — tiếp nối ticket nào, thuộc sprint/feature nào, ticket này tập trung giải quyết vấn đề gì. Max 3 câu. Business language, không technical detail.]

**Expected UI Fields:**
- Field 1 *
- Field 2 *
- Field 3 (conditional)
- Field 4
```

### Rules cho Epic Summary
- Goal = business language; Context = business language.
- Context mô tả: tiếp nối ticket nào, nền tảng đã có gì, ticket này tập trung vào điểm gì mới. Không mô tả API endpoint, call sequence, hoặc implementation detail.
- Ví dụ Context tốt: "Tiếp nối ticket MART-1866. Action Human Input web form đã có nền tảng từ sprint 12. Ticket này tập trung vào bảo mật truy cập form thông qua Basic Auth."
- Nếu action tương tự action khác, explicitly state difference ở góc độ business (behavior/scope), không technical.
- Goal max 2 câu; Context max 5 câu.

---

## User Stories Structure

### Story Order chuẩn

| # | Story Name | Required | Notes |
|---|---|---|---|
| 1 | Connect account and select action | ✓ | Always first. Fixed template. Không viết lại. |
| 2 | [Core action story] | ✓ | Main happy path — primary thing node does. |
| 3 | [Identify target resource] | ✓ | How user identifies file, folder, sheet, etc. |
| 4 | [Configure output or destination] | If applicable | Name, destination folder, output format, etc. |
| N | Handle authentication and permission errors | ✓ | Always last. Fixed template. |

### Per-Story Structure

```
### Story N: [Title — English verb phrase]

**Story:** As an automation workflow builder, I want [action], so that [outcome].

**Priority:** Must-have | Should-have | Nice-to-have

**Acceptance Criteria:**

**AC.01. [One-line summary in English]**
Given [pre-condition],
And [additional pre-condition if needed],
When [trigger / action],
Then [expected outcome],
And [additional outcome]

**AC.02. [Title]**
Given ...,
When ...,
Then ...

**Edge Cases:**
- [Tình huống bất thường — mô tả từ góc độ user/business, không technical]
- [VD: "User chọn file đã bị xóa", "Folder đích không còn tồn tại"]

```

---

## AC Format Rules

- AC ID: `AC.01`, `AC.02`, … sequential per story.
- AC ID + one-line title trên cùng dòng (English). **Title phải bold.**
- Given / When / Then / And — mỗi keyword trên dòng riêng, kết thúc bằng dấu phẩy (`,`).
- Dòng cuối cùng của AC không có dấu phẩy.
- One action per When — nếu 2 actions, split thành 2 ACs.
- Mỗi AC mô tả one observable behavior.
- Khi AC cover multi-option fields (dropdown), embed table bên trong AC body.

### AC with Table Example

```
**AC.NN. [Title]**
Given [pre-condition],
When user opens the dropdown,
Then system displays the following options:

| UI Label | API value | Description |
|---|---|---|
| Option A | value_a | What it does |
| Option B | value_b | What it does |
```

---

## Story 1: Fixed Template (Copy-paste)

```
### Story 1: Connect account and select action

**Story:** As an automation workflow builder, I want to connect an account and confirm the action type, so that the [Action Name] action runs under the correct authenticated identity.

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

**AC.03. Action * displays "[Action Name]" as pre-selected value**
Given the user is configuring this node,
When the user views the "Action *" field,
Then the dropdown shows "[Action Name]" already pre-selected,
And the user can change to other available actions if needed
```

---

## Error Handling Story: Fixed Template (Last story)

```
### Story [N]: Handle authentication and permission errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. OAuth token expired or revoked**
Given the connected account's OAuth token has expired or been revoked,
When the action attempts to execute,
Then the action fails with: "Authentication failed. Please reconnect your [service] account.",
And the workflow execution is marked as failed

**AC.02. Insufficient permission on resource**
Given the connected account does not have required access to the resource,
When the action runs,
Then the action surfaces: "Cannot [action]: the connected account does not have [permission level] access to [resource]. Check the sharing settings.",
And the resource ID and account email are included in the error detail

**AC.03. Resource not found**
Given the resource ID does not exist or has been deleted,
When the action runs,
Then the action surfaces: "[Resource] not found. It may have been deleted or moved.",
And the workflow execution is marked as failed

```

---

## Out of Scope Section

```
## Out of Scope

- **[Feature / case]** — [reason it is out of scope / which ticket covers it].
- **[Feature / case]** — [reason it is out of scope / which ticket covers it].
```

### Rules cho Scope
- **Không viết "In Scope"** — Business Rules đã mô tả đầy đủ những gì ticket cover. Viết In Scope sẽ bị trùng lặp.
- **Chỉ viết "Out of Scope"** — liệt kê rõ những gì KHÔNG nằm trong ticket này.

### Out of Scope Checklist
Luôn xem xét các items sau:
- Tính năng liên quan nhưng thuộc node/ticket khác
- Batch / bulk operations
- Reverse / undo operations
- Tính năng defer sang phase sau

---

## Lean Ticket Rules (Reminder)

- Không include `In Scope` — Business Rules đã cover. Chỉ viết `Out of Scope`.
- Không include `Implementation Notes` mặc định.
- Không include `Technical Notes` mặc định.
- Không include `External Dependencies` section riêng nếu đã có trong Dependencies.
- Không include `Success Criteria Summary` mặc định.
- Không mô tả lại Connected Account, authentication, permission, retry, scheduler/polling behavior nếu hệ thống đã có logic sẵn.
- Dùng standard reference sentences thay vì mô tả lại logic đã có.
- Ưu tiên ít story hơn, mỗi story chứa AC rõ và testable.
- **Giới hạn AC:** Chỉ viết AC cho behavior quan trọng, xảy ra thường xuyên, và QC có thể verify rõ ràng. Bỏ qua AC cho case hiếm gặp, khó test, hoặc behavior nhỏ không ảnh hưởng đến giá trị core.
- **Edge Cases viết từ góc business:** Mô tả tình huống bất thường mà user có thể gặp (VD: "File đã bị xóa", "Folder đích không còn tồn tại"). Không mô tả technical root cause (HTTP status code, API error, timeout mechanism). Chỉ giữ edge case có business impact thực sự.

---

## Standard Reference Sentences

Dùng các câu dưới đây trong ticket thay vì mô tả lại toàn bộ logic đã có sẵn:

### Connection & Account
- `Connection dùng lại logic Connected Account hiện có của hệ thống.`
- `Account field follows standard Connect step behavior (§ 12 of Product Standards).`

### Selectors & Pickers
- `Drive/Folder/File selector dùng lại picker và resource lookup hiện có của Google Drive integration.`
- `Spreadsheet/Worksheet selector dùng lại picker và resource lookup hiện có của Google Sheets integration.`
- `Calendar selector dùng lại picker và calendar access logic hiện có của Google Calendar integration.`
- `Model/voice options dùng lại provider option loading hiện có của {Provider} integration.`

### Retry & Error
- `Dùng lại logic retry của hệ thống.`
- `Error handling theo § 3 và § 4 của Product Standards.`

### Permission
- `Workflow access must follow the existing Workflow Permission Matrix.`

### File Storage
- `URL expiration dùng lại rule expired file của hệ thống.`
- `File size limit theo system limit (25 MB).`

---

## Business Rules Writing Guidelines

Business Rules section phải ngắn gọn, tập trung vào happy path, và tránh lặp lại nội dung đã có trong User Stories.

### Rules:
- **Không mô tả lại chi tiết** — nếu ý đã được mô tả đầy đủ trong User Stories/ACs, chỉ cần viết ý chính ngắn gọn. Ai muốn xem chi tiết thì đọc User Stories.
- **Chỉ mô tả happy case** — không mô tả lại edge cases (đã có trong Edge Cases của mỗi Story).
- **Không mô tả technical implementation** — tránh các ý chuyên sâu về kỹ thuật (VD: API call sequence, retry mechanism details, error handling flow). Chỉ mô tả business constraints và behavior.
- **Format ngắn gọn** — mỗi rule là 1 câu hoặc 1 bullet point, dễ scan.

### Good Examples:
✅ "Model là required, không có default — user phải chọn trước khi Continue."
✅ "Quality và Resolution options load động theo model đã chọn."
✅ "Khi user thay đổi model, Quality và Resolution được reset về default của model mới."

### Bad Examples:
❌ "Given user chọn model GPT Image 2, When Quality dropdown được load, Then hệ thống hiển thị Low/Medium/High..." (quá chi tiết, đã có trong AC)
❌ "Action request `response_format: 'url'` từ OpenAI, platform download image từ URL và lưu dạng binary PNG." (quá technical)
❌ "Nếu download fail thì retry 3 lần với exponential backoff..." (edge case + technical detail)

---

## Language Rules

- Ticket Header, Story titles, AC titles: English.
- AC body (Given/When/Then): Vietnamese-English mix accepted.
- Error messages: English only — follow error-message-catalog.md.
- Business rules, validation rules, mô tả: Vietnamese.
- UI field labels: Title Case (theo UI Label Naming Standard).

---

## Validation Behavior Rules

### Real-time Validation (Config-time)
- Validation error hiển thị **ngay** khi field invalid (real-time) và block Continue button. KHÔNG phải click Continue rồi mới báo error.
- Khi user nhập giá trị không hợp lệ hoặc field required đang trống sau khi touched, error hiển thị ngay và Continue bị disabled.

### AC Writing Rule for Real-time Validation
- Phải dùng từ "ngay" để làm rõ real-time behavior.
- Format chuẩn: `"Then the 'Continue' button is disabled, And inline validation error hiển thị ngay dưới field: '[Error message]'."`
- **KHÔNG** viết: `"And if the user clicks 'Continue', a validation error is shown"` — gây nhầm lẫn rằng error chỉ hiển thị khi click Continue.

### Dropdown with Default Value
- Nếu field là dropdown và đã có default value → KHÔNG cần AC validation `"{Field} is required."` vì field luôn có giá trị default, không thể trống.

## Dropdown Options — Single Source of Truth

- Danh sách options của dropdown chỉ mô tả **MỘT LẦN** duy nhất trong User Story (AC có embed table với UI Label + API value).
- **KHÔNG liệt kê lại options** ở Inputs section, Business Rules, hoặc bất kỳ section nào khác.
- Inputs section chỉ ghi tên field + note ngắn (VD: `Options: xem AC.02 Story 2`).
- Business Rules chỉ nói constraint (VD: "Model là required, không có default") mà không list lại từng option.
- Lý do: tránh phải sửa nhiều chỗ khi options thay đổi — AC là single source of truth.

---

## Error Handling Section Format

- Chỉ có **một mục Error Handling** duy nhất trong ticket, không tạo mục Validation Rules riêng.
- Table gồm **3 cột**: `Error Scenario | When Detected | User-facing Message`
- Không có cột Hard/Soft Fail, không có cột Retryable (technical detail — Dev tự quyết).
- Chỉ liệt kê lỗi **user-facing có business impact**.

---

## Output Field Rules

- Output fields chỉ mô tả **một lần** tại mục **Outputs** (bảng). AC không liệt kê lại danh sách fields — chỉ tham chiếu: `"Then action trả Output Payload gồm các fields tại mục Output."`

---

## AI Node Exceptions

Áp dụng cho OpenAI, Google Gemini, Anthropic Claude, AI Generic:
- Error Handling Story: **bỏ AC "Rate limit exceeded — automatic retry"**. Ghi: "Không retry. Chỉ dùng cơ chế retry node của hệ thống."
- Session ID: nếu node có Session ID, chỉ cần mô tả "Follow theo standard Ticket MART-1885: Session ID Field Spec - (AI Nodes)".
