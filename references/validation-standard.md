---
inclusion: manual
---

# Validation Standard

## Mục đích

File này định nghĩa chuẩn Node Structure và Validation Rules cho platform. AI phải tuân theo các rules này khi viết ticket liên quan đến input fields, validation behavior, và conditional logic.

> **Source:** Product Standards & Ticket Writing Guideline v2.0 — § 1 Node Standards, § 2 Validation Standards.

---

## PART 1: NODE STANDARDS

---

### 1.1 Node Structure (4-step)

Mọi node trên platform phải follow fixed 4-step structure theo thứ tự:

| Step | Purpose | Required |
|---|---|---|
| Connect (or Credential) | Authenticate account và select action/trigger type. Bao gồm AI Model selection trong một số AI Node. | ✓ |
| Configure | Enter input fields, map dynamic variables | ✓ |
| Advanced | Customise retry policy và error handling behaviour | Optional |
| Test (Test Node only / Test Workflow) | Run test với sample data trước khi publish. Cho phép user biết output field nào support Expression (Dynamic Value) cho upstream Node. | Optional |

---

### 1.2 Connect Step

Hai fields bắt buộc trên mọi Connect step:

| Field | Type | Required | Behaviour |
|---|---|---|---|
| Account * | Dropdown + OAuth | ✓ | "Continue" is disabled until an account is selected |
| Action * | Dropdown | ✓ | Pre-selected per node context; user can change to other available actions |
| AI Model | Dropdown | For AI Node | System usually uses default Model which is the most optimal for selected Action |

**Additional rules:**
- Account field displays `Name (email)` after successful selection — e.g., `Nguyen (nguyennnp@company.io)`
- If no account exists, system redirects to OAuth flow to connect a new one
- User can switch accounts by clicking "Select" again

---

### 1.3 Configure Step

Configure step là main input quyết định cách Node hoạt động.

**Supported input field types:**

| Type | Description | Dynamic Variable |
|---|---|---|
| Text input | Free-text or mapped variable | ✓ Yes |
| Dropdown | Single-select from static or API-loaded list | ✗ No |
| Toggle | On/Off; may trigger conditional fields | ✗ No |
| Type selector + Dropdown | Changing type reloads dropdown data source | ✓ Yes |
| Type selector + Text | Changing type switches to ID/value text entry | ✓ Yes |
| Textarea | Multi-line free text | ✓ Yes |

**Field labelling rules:**

| State | Indicator | Example |
|---|---|---|
| Required | Asterisk (*) after the label | File Name * |
| Optional | No indicator | Drive |
| Has tooltip | ℹ icon next to the label | — |

---

### 1.4 Conditional Field Standard

Field xuất hiện hoặc ẩn dựa trên giá trị của trigger/action field phải tuân theo:

| Situation | Behaviour |
|---|---|
| Field is hidden | Do not validate; do not include in request payload |
| Field becomes visible | Apply normal validation according to its field type |
| Trigger/Action field value changes | Reset the conditional field to empty |
| Conditional field is reset and is required | Display validation error immediately |

**Critical Rule:** All fields of the same type — regardless of whether they belong to the same or different Nodes, Actions, or Triggers — must share the exact same data type and format to support dynamic variables (Expression).

**Note:** Hidden fields are never validated and never included in the API request payload, regardless of any previously entered value.

---

### 1.5 Dynamic Variable Standard

| Situation | Behaviour |
|---|---|
| Field contains a placeholder, e.g. `{{step.field}}` | Passes required validation at the FE level |
| Placeholder resolves to empty at runtime | Runtime error — not a FE validation error |
| Placeholder contains invalid or unsupported characters | Special-character rules do not apply to the placeholder itself. Or: Runtime error — not a FE validation error |
| Entire field is a placeholder that resolves to empty | Block execution; surface a runtime error message |

---

## PART 2: VALIDATION STANDARDS

---

### 2.1 General Rules

- Validation runs client-side before any API call is made (Required field, Value Format — in manual input, Length of value).
- **Pristine state:** no error is shown on a field the user has never touched (avoid user's pressure).
- **Trigger of Error:** when the user clicks "Continue" or blurs a field after touching it.
- **Multiple errors:** all field errors are shown simultaneously, not one at a time.
- **Auto-clear:** once the user corrects a field, the error clears and the border resets.

**Visual behaviour:**

| Element | Error State Behaviour |
|---|---|
| Field border | Turns red |
| Error message | Inline, immediately below the field (no toast, modal, or popup) |
| "Continue" button | Disabled while any required field is invalid |

---

### 2.2 Required Field Validation

**Error message format:**
```
"{Field Name} is required."
```

**Validation behaviour by input case:**

| Input Case | Behaviour |
|---|---|
| Empty string | Block; show required error |
| Whitespace only | Treat as empty; block |
| Dynamic variable placeholder | Pass — not counted as empty |
| Null / undefined | Block; show required error |

---

### 2.3 Text Input Validation

| Rule | Behaviour |
|---|---|
| Max length | Truncate or block — define per field |
| Special characters | Define a whitelist or blacklist per field type |
| Illegal characters found | Replace with `_` and warn, or block — define per field |
| Case sensitivity | Define per field (e.g., email = case-insensitive) |

---

### 2.4 Email Validation

- Standard: RFC 5322
- Must have characters before `@` and a valid domain after `@`
- No spaces allowed

**Error message:**
```
"Please enter a valid email address (e.g., user@example.com)."
```

---

### 2.5 URL Validation

- Must start with `http://` or `https://`
- Must have a valid domain
- Localhost is not accepted in production environments

**Error message:**
```
"Please enter a valid URL starting with http:// or https://."
```

---

### 2.6 Number Validation

Define each parameter explicitly when a number field is used:

| Parameter | Definition |
|---|---|
| Min value | Minimum accepted value |
| Max value | Maximum accepted value |
| Decimal precision | Number of decimal places allowed |
| Negative numbers | Whether negative values are permitted |
| Integer only | Whether the value must be a whole number |

**Error message format:**
```
"Please enter a number between {min} and {max}."
```

---

### 2.7 Date & Time Validation

| Rule | Standard | Notes |
|---|---|---|
| Date format | ISO 8601 for storage | Display per locale |
| Time format | 24-hour (HH:mm:ss, hh:mm) or 12-hour (hh:mm:ss AM/PM, hh:mm AM/PM) for storage | — |
| Timezone | UTC for storage | Convert to local timezone for display |
| Date range | Start date must not be later than End date | — |
| Past dates | Define explicitly per field | Whether past dates are permitted |

**Supported date display formats (24-hour):**
- `MM/DD/YYYY hh:mm:ss`
- `MM/DD/YYYY hh:mm` (seconds = 0)
- `MM/DD/YYYY` (default Start time = 00:00:00, End time = 23:59:59)

**Supported date display formats (AM/PM):**
- `MM/DD/YYYY hh:mm:ss AM / PM`
- `MM/DD/YYYY hh:mm AM / PM`

**Default:** Time = 00:00:00 (khi không có End date)

**Error message format:**
```
"Please enter a valid date in {format} format."
```

---

### 2.8 File Name Validation

| Rule | Behaviour |
|---|---|
| Max length | File Name must be 100 characters or less. FE and BE must validate this rule. |
| Illegal characters | File Name can only contain letters, numbers, spaces, hyphens, and underscores. FE & BE must validate this rule. |
| Empty after variable resolve | Block execution, BE return: "File Name cannot be empty or contain only spaces." |
| Auto-sanitise | Not supported |

---

### 2.9 Dropdown Validation

| Case | Behaviour |
|---|---|
| No item selected | Block; FE validation show: `"{Field Name} is required."` |
| Selected item deleted from source after config | **Runtime error** when resolving the value |
| API returns empty list | Show empty state message in dropdown — do not block the UI |

---

### 2.10 Range / Limit Validation

Define each parameter explicitly per field:

| Parameter | Definition |
|---|---|
| Minimum value / length | Minimum accepted value or character count |
| Maximum value / length | Maximum accepted value or character count |
| Out-of-range behaviour | Block or truncate, with the accompanying message |

**Error message format:**
```
"{Field Name} must be between {min} and {max}."
```

---

### 2.11 UI Label Naming Standard

**Rule 1 — Always Title Case for Multi-Word Labels**

Each word in a multi-word label starts with a capital letter. Applies to UI display and error messages.

| ✗ Inconsistent (do not use) | ✓ Canonical (always use) |
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

| ✗ Incorrect | ✓ Canonical | Reason |
|---|---|---|
| User Name / user name | Username | Single compound word — standard UI convention |
| Pass word / pass word | Password | Single compound word |
| E mail / e-mail | Email | Modern standard; no hyphen |

**Note:** When in doubt, check how the label appears in the product's own design system / Figma. The UI is the source of truth for the canonical label string.

**Rule 3 — Use {label} Token in All Error Message Templates**

Error messages must always reference the field by its canonical label. In ticket ACs and validation specs, write the label as a `{label}` placeholder and substitute the exact canonical label name when writing the final message string.

**Standard error message patterns using {label}:**

| Pattern | Example (label = "Full Name") |
|---|---|
| {label} is required. | "Full Name is required." |
| {label} must be at least {min} characters. | "Full Name must be at least 2 characters." |
| {label} must be between {min} and {max} characters. | "Full Name must be between 2 and 100 characters." |
| {label} must be {max} characters or less. | "Full Name must be 100 characters or less." |
| {label} can only contain letters and spaces. | "Full Name can only contain letters and spaces." |
| {label} can only contain letters, numbers, and underscores. | "Username can only contain letters, numbers, and underscores." |
| {label} cannot contain special characters. | "Full Name cannot contain special characters." |
| {label} cannot be empty after resolving variables. | "Full Name cannot be empty after resolving variables." |
| {label} must be a valid {format}. | "Email Address must be a valid email address." |
| {label} must match {other label}. | "Confirm Password must match New Password." |

**Rule 4 — Consistency in Tickets and Error Strings**

- The label in the error message must exactly match the label shown in the UI — same capitalisation, same spelling.
- Never use a lowercase or abbreviated version of the label inside an error string.
- If the UI uses "Full Name", the error must say "Full Name is required." — not "full name is required." or "Fullname is required."
- When writing ACs, always spell out the canonical label inside the message string — do not leave it as `{label}`.

**Note:** Treat the canonical label as a constant. Every place it appears — UI, error message, AC, documentation — must use the same string. This makes global find-and-replace safe and keeps all artefacts consistent.

---

### 2.12 Standard for Binary File

#### 1. Overview

Binary File is obtained from upstream sources (Drive, Telegram, Messenger, etc.) with accompanying metadata.

**Purpose:**
- Store Files (backup)
- Share Files (gửi qua channel)
- Process Files (extract, convert)
- Validate Files (kiểm tra properties)
- Audit & Logging (tracking)
- Conditional Processing (lọc files)
- Batch Operations (xử lý hàng loạt)

#### 2. Binary File Data Structure

**2.1 Metadata of Binary File**

Binary File from upstream provides objects containing the following information:

*i: start from 0

| Field Name | Data Type | Required | Description | Example |
|---|---|---|---|---|
| binary | Array | ✓ | Path of binary file | - |
| binary[i].data | Object | ✓ | binary data of the file | webhooks/2026/05/c719cf2902... |
| binary[i].fileName | String | ✓ | Original file name | Sample-pdf-Files-for-Testing-1-1280x720.webp |
| binary[i].id | String (UUID) | ✓ | Unique file identifier | 414c6844-b537-4a56-96f5-907272fc95dc |
| binary[i].mimeType | String | ✓ | MIME type of the file | image/webp, application/pdf |
| binary[i].size | Number | ✓ | File size (bytes) | 28958 |
| binary[i].success | Boolean | ✓ | Upload success status | true / false |

**2.2 Structure when Dragging Expression for Current Node**

```json
{
  "binary": {
    "file0": {
      "data": "webhooks/2026/05/file0_path...",
      "fileName": "document.pdf",
      "id": "uuid-000-001",
      "mimeType": "application/pdf",
      "size": 45678,
      "success": true
    },
    "file1": {
      "data": "webhooks/2026/05/file1_path...",
      "fileName": "image_cover.png",
      "id": "uuid-000-002",
      "mimeType": "image/png",
      "size": 12345,
      "success": true
    }
  }
}
```

#### 3. Field Type for Binary File and Metadata

**3.1 Field type: File - Drag Binary file**

**Label:** File - Object

**How to use:**
- Drag binary file from upstream
- Use when you need to upload, send, or process actual file

**Applications:**
- Node: Upload file to Drive
- Node: Send file via Telegram
- Node: Send attachment via Email

**Error Message - Validation:**

| Error | Message |
|---|---|
| File not uploaded | {fieldName} is required. |
| File too large | {fieldName} cannot exceed {max} MB. |
| Unsupported format | {fieldName} must be a valid {format}. |

**3.2 Field Type: Text (Drag "Metadata" of Binary file)**

**Label:**
- Binary[i].data - String (URL)
- binary[i].fileName - String
- Binary[i].id - String (UUID)
- binary[i].mimeType - String
- Binary[i].size - Number
- Binary[i].success - Boolean

**How to use:**
- Drag each metadata field from upstream
- Use in condition, mapping, log, transform
- Do not drag entire object, only drag the metadata field you need

**Applications:**
- Node: Condition
- Node: Log (write file information)
- Node: Transform (create object data from metadata)

**Error Message - Validation:**

| Error | Message |
|---|---|
| File Name empty | {fieldName} is required. |
| File Type invalid | {fieldName} must be a valid {format}. |
| File Size exceeds limit | {fieldName} cannot exceed {max} bytes. |
| File ID invalid | {fieldName} must be a valid {format}. |

#### 4. List of Nodes Supporting Binary File

**Record:** count by number of binary
**In binary contain a lot of binary[i]**

**4.1 Node Condition (If, Filter)**

**Lưu ý:**
- Binary[i] nằm trong binary
- Khi 1 binary[i] thoả điều kiện, tất cả file trong binary đều được trả qua Node tiếp theo
- **Disadvantage:** Tồn tại những file binary[i] không thoả điều kiện

| Use case | Field Supported | Data Type | Example |
|---|---|---|---|
| File Size Check | binary[0].size | Text | binary[0].fileSize > 5000000 |
| File Type Check | binary[1].mimeType | Text | binary[0].fileType === "application/pdf" |
| File Name Check | binary[2].fileName | Text | binary[0].fileName contains "report" |

**4.2 Node Action: File Storage (GG Drive)**

**Lưu ý:**
- Tuỳ chọn: Kéo binary: thì tất cả các file trong binary đều được xử lý
- Hoặc: Kéo binary[i]: thì chỉ xử lý mỗi file thuộc binary[i] được kéo qua
- Ví dụ: Kéo binary[1] => thì chỉ upload mổi binary[2]

| Use case | Field Supported | Data Type | Purpose |
|---|---|---|---|
| Upload to Drive | binary[0] | File | Upload file to Drive |

**4.3 Node Action: Communication Channel (Telegram, Messenger, Gmail)**

**Lưu ý:**
- Binary[i] nằm trong binary
- Kéo binary: thì tất cả các file binary[i] đều được xử lý
- Kéo binary[i]: thì chỉ xử lý mỗi file thuộc binary[i] được kéo qua
- Ví dụ: Kéo binary[1] => thì chỉ upload mỗi binary[2]

| Use case | Field Supported | Data Type | Purpose |
|---|---|---|---|
| Send Photo | binary[0] | File | Send image |
| Send Document | binary[1] | File | Send document |
| Send Attachment | binary[0] | File | Send attachment |
| Send Document | binary[2].fileName | Text | File display name |

**4.4 Node Action: File Convert (Extract from File)**

**Lưu ý:**
- Binary[i] nằm trong binary
- Kéo binary[i]: thì chỉ xử lý mỗi file thuộc binary [i] được kéo qua
- Ví dụ: Kéo binary[1] => thì chỉ upload mỗi binary[1]

| Use case | Field Supported | Data Type | Purpose |
|---|---|---|---|
| Convert Image to Text | binary[i] Or binary[1] | File | Process image |
| Convert PDF to Text | binary[i] Or binary[1] | File | Process PDF file |

#### 5. Processing Multiple Binary Files

**Scenario:** Processing multiple binary files via loop

- Loop: Process each file one by one
- Condition: Check if upload succeeded or failed
- Action: Send result (success) for each result
- Continue/Exit: If more files exist, proceed to next; otherwise stop

#### 6. Processing Multiple File Information (Metadata)

**Scenario:** Processing metadata of multiple binary files via loop

- Loop: Process each record one by one
- Extract: Get file information (fileName, fileType, fileSize)
- Validate: Ensure metadata is valid
- Filter: Apply user-defined rules to select files for processing
- Continue/Exit: If more files exist, proceed to next; otherwise stop

#### 7. Valid Binary File Types

**7.1 Supported File Categories**

| Category | File Types | MIME Types | Examples |
|---|---|---|---|
| Image | JPG, PNG, GIF, WebP, BMP, SVG, TIFF | image/jpeg, image/png, image/gif, image/webp, image/bmp, image/svg+xml, image/tiff | Photo.jpg, Logo.png, Icon.gif |
| Document | PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX, TXT | application/pdf, application/msword, application/vnd.openxmlformats-officedocument.wordprocessingml.document, application/vnd.ms-excel, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-powerpoint, application/vnd.openxmlformats-officedocument.presentationml.presentation, text/plain | Report.pdf, Document.docx, Data.xlsx |
| Video | MP4, AVI, MOV, MKV, WMV, FLV, WebM | video/mp4, video/x-msvideo, video/quicktime, video/x-matroska, video/x-ms-wmv, video/x-flv, video/webm | Video.mp4, Movie.mov, Recording.mkv |
| Audio | MP3, WAV, FLAC, AAC, OGG, M4A | audio/mpeg, audio/wav, audio/flac, audio/aac, audio/ogg, audio/mp4 | Music.mp3, Podcast.wav, Song.m4a |
| Archive | ZIP, RAR, 7Z, TAR, GZ | application/zip, application/x-rar-compressed, application/x-7z-compressed, application/x-tar, application/gzip | Files.zip, Backup.rar, Data.7z |
| Code | JSON, XML, CSV, SQL, HTML, CSS, JavaScript, Python | application/json, application/xml, text/csv, application/sql, text/html, text/css, text/javascript, text/x-python | Config.json, Data.xml, Script.py |

**7.2 Validation Criteria for Binary Files**

| Criteria | Valid | Invalid |
|---|---|---|
| File Size | > 0 bytes, < 25 MB (system limit) | 0 bytes (empty), > 25 MB (too large) |
| File Name | Not empty, contains valid characters (Not unique required) | Empty, special characters only |
| Status | success = true | success = false |
| MIME Type | Listed in supported types above | Unsupported type (e.g., .exe, .bat) |
| File ID | Valid UUID format (36 chars) | Invalid UUID, empty, malformed |

---

### 2.13 Node Name & Connection Name Validation

Max length rule cho Node Name và Connection Name để prevent oversized names blocking Workflow Save, Publish, or Execute Test.

| Rule | Behaviour |
|---|---|
| Max length | {Node Name}/{Connection Name} must be 256 characters or less. FE and BE must validate this rule. |
| Applies to | Create and Update flow for both Node Name and Connection Name. |
| Empty value | Follow existing required-field validation if the field is required in that flow. |
| Auto-truncate | Not supported. User must manually shorten the value. |
| Save / Publish / Execute with invalid value | Block action until the value is valid. |

**Frontend Validation:**

| Case | Behaviour |
|---|---|
| User enters more than 256 characters | Show inline validation error: `"Node Name/Connection Name must be 256 characters or less."` |
| User blurs the field with invalid value | Show validation error. |
| User clicks Save / Publish while value is invalid | Block submit and keep the validation error visible. |
| Auto-truncate | Not supported. User must manually shorten the value. |
| User shortens value to 256 characters or less | Clear validation error and allow Save / Publish. |

**Backend Validation:**

| Case | Behaviour |
|---|---|
| API receives Node Name or Connection Name longer than 256 characters | Reject request and do not save data. |
| Validation fails | Return `"{Node Name}/{Connection Name} must be 256 characters or less."` |
| Validation passes | Allow create/update and workflow Save / Publish / Execute Test to continue normally. |
