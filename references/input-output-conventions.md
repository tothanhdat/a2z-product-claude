# Input & Output Conventions

## Mục đích

File này định nghĩa conventions chuẩn cho Input fields và Output payload khi viết ticket. AI phải tuân theo các rules này để đảm bảo consistency xuyên suốt platform.

---

## PART 1: INPUT CONVENTIONS

---

### 1.1 Node Structure (4-step)

Mọi node trên platform phải follow fixed 4-step structure:

| Step | Purpose | Required |
|---|---|---|
| Connect (or Credential) | Authenticate account và select action/trigger type | ✓ |
| Configure | Enter input fields, map dynamic variables | ✓ |
| Advanced | Customise retry policy và error handling behaviour | Optional |
| Test | Run test với sample data trước khi publish | Optional |

---

### 1.2 Supported Input Field Types

| Type | Description | Dynamic Variable |
|---|---|---|
| Text input | Free-text or mapped variable | ✓ Yes |
| Dropdown | Single-select from static or API-loaded list | ✗ No |
| Toggle | On/Off; may trigger conditional fields | ✗ No |
| Type selector + Dropdown | Changing type reloads dropdown data source | ✓ Yes |
| Type selector + Text | Changing type switches to ID/value text entry | ✓ Yes |
| Textarea | Multi-line free text | ✓ Yes |
| File (Binary) | Drag binary file from upstream | ✓ Yes (drag expression) |
| Multi-select | Select one or more items from list | ✗ No |

---

### 1.3 Field Labelling Rules

| State | Indicator | Example |
|---|---|---|
| Required | Asterisk (*) after label | File Name * |
| Optional | No indicator | Drive |
| Has tooltip | ℹ icon next to label | — |

**UI Label Naming:** Always Title Case for multi-word labels.
- ✓ `File Name`, `Phone Number`, `Calendar Selection Mode`
- ✗ `file name`, `phone number`, `calendar selection mode`

---

### 1.4 Conditional Field Rules

| Situation | Behaviour |
|---|---|
| Field is hidden | Do not validate; do not include in request payload |
| Field becomes visible | Apply normal validation according to its field type |
| Trigger/Action field value changes | Reset conditional field to empty |
| Conditional field is reset and is required | Display validation error immediately |

**Rule:** Hidden fields are never validated and never included in the API request payload, regardless of any previously entered value.

---

### 1.5 Dynamic Variable Rules

| Situation | Behaviour |
|---|---|
| Field contains placeholder `{{step.field}}` | Passes required validation at FE level |
| Placeholder resolves to empty at runtime | Runtime error — not FE validation error |
| Placeholder contains invalid characters | Special-character rules do not apply to placeholder itself; runtime error if resolved value invalid |
| Entire field is placeholder that resolves to empty | Block execution; surface runtime error message |

---

### 1.6 Input Table Format (for tickets)

Khi viết Inputs section trong ticket, dùng format sau:

**For Actions:**

| Field | Type | Required | Default | Expression | Description |
|---|---|---|---|---|---|
| Connection | Connected Account | Yes | None | No | Dùng lại logic Connected Account hiện có. |
| File ID | Text input | Conditional | None | Yes | Required khi Input Type = File ID. |
| Speed | Dropdown | No | Normal | No | Options: Slow, Normal, Fast. |

**For Triggers:**

| Field | Type | Required | Default | Mapped Variable | Notes |
|---|---|---|---|---|---|
| Connection | Connected Account | Yes | None | No | — |
| Calendar Selection Mode | Dropdown | Yes | All my Calendars | No | — |

---

### 1.7 Input Validation Rules (per type)

#### Text Input
- Max length: define per field
- Special characters: define whitelist/blacklist per field
- Illegal characters found: replace with `_` and warn, or block — define per field

#### Email
- Standard: RFC 5322
- Must have characters before `@` and valid domain after `@`
- No spaces allowed

#### URL
- Must start with `http://` or `https://`
- Must have valid domain
- Localhost not accepted in production

#### Number
- Define: min value, max value, decimal precision, negative numbers allowed, integer only

#### Date & Time
- Storage: ISO 8601, UTC
- Display: convert to local timezone
- Date range: start date must not be later than end date

#### File Name
- Max length: 100 characters (FE and BE validate)
- Allowed characters: letters, numbers, spaces, hyphens, underscores
- Auto-sanitise: not supported
- Empty after variable resolve: block execution

#### Dropdown
- No item selected: block; show `"{Field Name} is required."`
- Selected item deleted from source after config: runtime error
- API returns empty list: show empty state message — do not block UI

---

## PART 2: OUTPUT CONVENTIONS

---

### 2.1 Required Fields (All Nodes)

Mọi action output payload phải bao gồm:

| Field | Type | Description |
|---|---|---|
| success | Boolean | `true` if action completed successfully |
| action | String | Name of action executed (e.g., `"create_file"`, `"delete_file"`) |
| timestamp | ISO 8601 | Timestamp when action completed |

---

### 2.2 Field Naming Convention

| Rule | Standard | Example |
|---|---|---|
| Case | snake_case | `file_name`, `folder_id` |
| Boolean fields | Prefix: `is_`, `has_`, `can_` | `is_public`, `has_error` |
| Timestamp fields | Suffix: `_at` or `_time` | `created_at`, `modified_time` |
| ID fields | Suffix: `_id` | `file_id`, `folder_id` |
| Count fields | Suffix: `_count` | `row_count`, `item_count` |
| URL fields | Suffix: `_url` or `_link` | `web_view_url`, `audio_url` |

---

### 2.3 Skip / No-op Output

Khi action không thực hiện gì (resource already exists, duplicate-skip mode):

| Field | Value |
|---|---|
| success | `true` |
| skipped | `true` |
| reason | Descriptive message (e.g., `"File already exists."`) |

---

### 2.4 Output Table Format (for tickets)

Khi viết Outputs section trong ticket:

| Field | Type | Description | Nullable |
|---|---|---|---|
| success | boolean | Whether action succeeded | No |
| file_id | string | ID of created/modified file | No |
| file_name | string | Name of file | No |
| file_url | string | Web view URL of file | No |
| file_size | number | File size in bytes | No |
| description | string | File description if provided | Yes |
| created_at | string | Creation timestamp ISO 8601 | No |
| timestamp | string | Execution timestamp ISO 8601 | No |

---

### 2.5 Output Design Rules

1. Define outputs intentionally — không simply "return provider response".
2. Include fields mà downstream steps likely cần sử dụng.
3. Stable field names — không đổi tên output field mà không có backward compatibility note.
4. Nullable fields phải explicitly mark `Nullable: Yes`.
5. Resolved values (model, voice, speed) phải trả về actual values used: `model_used`, `voice_used`, `speed_used`.
6. File-related outputs luôn include: `file_id`, `file_name`, `file_url` (nếu applicable).
7. Trigger outputs luôn include: `detected_at`, `trigger_mode` (instant/polling).
8. URL outputs phải có expiration policy rõ ràng (reference platform rule).

---

### 2.6 Trigger Output Conventions

Trigger outputs có thêm các standard fields:

| Field | Type | Description |
|---|---|---|
| detected_at | string | Thời điểm trigger nhận event (ISO 8601) |
| trigger_mode | string | `"instant"` hoặc `"polling"` |

---

## PART 3: BINARY FILE CONVENTIONS

### Binary File Metadata Structure

| Field | Type | Required | Description |
|---|---|---|---|
| binary[i].data | Object | ✓ | Binary data path |
| binary[i].fileName | String | ✓ | Original file name |
| binary[i].id | String (UUID) | ✓ | Unique file identifier |
| binary[i].mimeType | String | ✓ | MIME type of file |
| binary[i].size | Number | ✓ | File size in bytes |
| binary[i].success | Boolean | ✓ | Upload success status |

### Validation Criteria

| Criteria | Valid | Invalid |
|---|---|---|
| File Size | > 0 bytes, < 25 MB | 0 bytes (empty), > 25 MB |
| File Name | Not empty, valid characters | Empty, special characters only |
| Status | success = true | success = false |
| MIME Type | Listed in supported types | Unsupported (.exe, .bat) |
| File ID | Valid UUID format (36 chars) | Invalid UUID, empty |

### Supported File Categories

| Category | Extensions |
|---|---|
| Image | JPG, PNG, GIF, WebP, BMP, SVG, TIFF |
| Document | PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX, TXT |
| Video | MP4, AVI, MOV, MKV, WMV, FLV, WebM |
| Audio | MP3, WAV, FLAC, AAC, OGG, M4A |
| Archive | ZIP, RAR, 7Z, TAR, GZ |
| Code | JSON, XML, CSV, SQL, HTML, CSS, JavaScript, Python |
