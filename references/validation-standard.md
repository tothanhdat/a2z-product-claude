---
inclusion: manual
---

# Validation Standard

## Mục đích

File này định nghĩa các rules **không có trong file reference nào khác**: Connect Step behavior, validation trigger & visual behavior, UI Label Naming Standard, và Node/Connection Name validation.

> Để tra cứu: input field types + conditional fields + dynamic variables → `input-output-conventions.md`. Error message patterns → `error-message-catalog.md`.

---

## PART 1: CONNECT STEP

---

### 1.1 Connect Step — Fields & Behavior

Hai fields bắt buộc trên mọi Connect step:

| Field | Type | Required | Behaviour |
|---|---|---|---|
| Account * | Dropdown + OAuth | ✓ | "Continue" is disabled until an account is selected |
| Action * | Dropdown | ✓ | Pre-selected per node context; user can change to other available actions |
| AI Model | Dropdown | For AI Node only | System uses default Model which is the most optimal for selected Action |

**Additional rules:**
- Account field displays `Name (email)` after successful selection — e.g., `Nguyen (nguyennnp@company.io)`
- If no account exists, system redirects to OAuth flow to connect a new one
- User can switch accounts by clicking "Select" again

---

## PART 2: VALIDATION BEHAVIOR

---

### 2.1 General Validation Rules

- Validation runs client-side before any API call is made.
- **Pristine state:** no error is shown on a field the user has never touched — tránh gây áp lực ngay khi mở form.
- **Trigger of error:** khi user click "Continue" hoặc blur khỏi field sau khi đã touch.
- **Multiple errors:** tất cả field errors hiển thị đồng thời, không one-by-one.
- **Auto-clear:** khi user sửa đúng, error tự clear và border reset.

**Visual behavior:**

| Element | Error State |
|---|---|
| Field border | Turns red |
| Error message | Inline, immediately below the field (no toast, modal, or popup) |
| "Continue" button | Disabled while any required field is invalid |

---

### 2.2 UI Label Naming Standard

**Rule 1 — Always Title Case for Multi-Word Labels**

Mỗi word trong multi-word label viết hoa chữ cái đầu. Áp dụng cho UI display và error messages.

| ✗ Không dùng | ✓ Canonical |
|---|---|
| Company name / company name | Company Name |
| Full name / full name | Full Name |
| Phone number / phone number | Phone Number |
| New password / new password | New Password |
| Confirm password | Confirm Password |
| First name / Last name | First Name / Last Name |
| Date of birth | Date of Birth |
| Email address | Email Address |
| Account name | Account Name |

**Rule 2 — Single-Concept Words Stay as One Word**

| ✗ Incorrect | ✓ Canonical | Lý do |
|---|---|---|
| User Name / user name | Username | Single compound word — standard UI convention |
| Pass word / pass word | Password | Single compound word |
| E mail / e-mail | Email | Modern standard; no hyphen |

> Khi nghi ngờ, kiểm tra label trong Figma/design system. UI là source of truth.

**Rule 3 — Dùng canonical label trong error messages**

Error message phải reference field bằng đúng canonical label — cùng capitalisation, cùng spelling.

| Pattern | Ví dụ (label = "Full Name") |
|---|---|
| {label} is required. | "Full Name is required." |
| {label} must be at least {min} characters. | "Full Name must be at least 2 characters." |
| {label} must be {max} characters or less. | "Full Name must be 100 characters or less." |
| {label} can only contain letters and spaces. | "Full Name can only contain letters and spaces." |
| {label} must match {other label}. | "Confirm Password must match New Password." |

**Rule 4 — Consistency across all artefacts**

Label phải giống nhau ở mọi nơi: UI, error message, AC, documentation. Không dùng lowercase hoặc abbreviated version trong error string.

---

### 2.3 Node & Connection Name Validation

Max length rule để prevent oversized names blocking Workflow Save, Publish, hoặc Execute Test.

| Rule | Behaviour |
|---|---|
| Max length | 256 characters or less. FE và BE đều phải validate. |
| Applies to | Create và Update flow cho cả Node Name và Connection Name. |
| Auto-truncate | Không hỗ trợ. User phải tự rút ngắn. |
| Save / Publish / Execute với giá trị invalid | Block action cho đến khi giá trị hợp lệ. |

**Frontend Validation:**

| Case | Behaviour |
|---|---|
| User nhập quá 256 ký tự | Show inline error: `"Node Name must be 256 characters or less."` hoặc `"Connection Name must be 256 characters or less."` |
| User blur khỏi field với giá trị invalid | Show validation error |
| User click Save / Publish khi invalid | Block submit, giữ nguyên validation error |
| User rút ngắn về ≤ 256 ký tự | Clear validation error, cho phép Save / Publish |

**Backend Validation:**

| Case | Behaviour |
|---|---|
| API nhận Name dài hơn 256 ký tự | Reject request, không lưu data |
| Validation fails | Return `"Node Name/Connection Name must be 256 characters or less."` |
| Validation passes | Cho phép create/update và workflow Save / Publish / Execute Test |
