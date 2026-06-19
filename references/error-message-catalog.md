# Error Message Catalog — Rule-Based Reference

## Mục đích

File này định nghĩa **bộ rule chuẩn** cho tất cả error messages trong platform. Mỗi loại lỗi có 1 pattern mặc định; service nào cần khác thì ghi rõ override kèm lý do.

---

## Cách sử dụng

| Vai trò | Cách dùng |
|---|---|
| **BA** | Áp dụng rule mặc định khi viết Error Handling. Chỉ dùng override nếu context buộc phải khác. |
| **Dev** | Implement đúng rule. Nếu thêm error mới, follow pattern mặc định — không tạo biến thể mới. |
| **QA** | Verify message khớp rule hoặc override đã ghi nhận. Flag nếu thấy message ngoài catalog. |

---

## Quy ước ký hiệu

- `{Field Name}`, `{service}`, `{resource}`, `{action}`, `{max}`, `{min}` — placeholder, thay bằng giá trị thực.
- **Override** = pattern khác rule mặc định, chỉ áp dụng cho service/context cụ thể.
- **Retryable**: ✓ = platform tự retry (exponential backoff, max 3), ✗ = không retry.

---

## Error Categories

1. [Config-time Validation Errors](#1-config-time-validation-errors)
2. [Runtime Validation Errors](#2-runtime-validation-errors)
3. [Authentication & Authorization Errors](#3-authentication--authorization-errors)
4. [Resource Errors](#4-resource-errors)
5. [Network & Provider Errors](#5-network--provider-errors)
6. [Storage & Quota Errors](#6-storage--quota-errors)
7. [Binary File Errors](#7-binary-file-errors)
8. [Workflow Execution Errors](#8-workflow-execution-errors)
9. [Content Policy & AI-Specific Errors](#9-content-policy--ai-specific-errors)

---

## 1. Config-time Validation Errors

Errors xảy ra khi user đang configure workflow, trước khi execution. Retryable: ✗ (tất cả).

### 1.1 Required Field

| Rule | Pattern |
|---|---|
| **Default** | `{Field Name} is required.` |
| Multiple fields | Hiển thị error riêng cho mỗi field |

**Override:**

| Context | Message | Lý do |
|---|---|---|
| IF/Filter node (Value 1, Value 2, Condition) | `This field could not be empty.` | Generic field, không có tên cụ thể |
| Telegram (thiếu nhiều fields cùng lúc) | `Required configuration are missing.` | Validate cả bundle, không từng field |
| Google Sheet: Delete/Clear Row | `Row input is empty. Please specify at least one row.` | Cần instruction cụ thể về format input |
| Google Sheet: Export Worksheet | `At least one export format (PDF or CSV) must be specified.` | Multi-select, không phải single field |
| Google Calendar: Specific mode | `At least one calendar must be selected when using specific calendars.` | Multi-select có context |

---

### 1.2 Length & Range Limits

| Rule | Pattern |
|---|---|
| Min length | `{Field Name} must be at least {min} characters.` |
| Max length | `{Field Name} must be {max} characters or less.` |
| Range | `{Field Name} must be between {min} and {max} characters.` |
| Exact | `{Field Name} must be exactly {n} characters.` |
| Numeric field: not positive | `{Field Name} must be a positive number.` |
| Numeric range: start > end | `Invalid {Field Name}. The start must be smaller than the end.` |
| Numeric format sai | `Invalid {Field Name}. Use numbers, commas, or ranges (e.g., 1, 2-6).` |

---

### 1.3 Format & Special Characters

| Rule | Pattern |
|---|---|
| Invalid file name characters | `{Field Name} can only contain letters, numbers, spaces, hyphens, and underscores.` |
| No special characters | `{Field Name} cannot contain special characters.` |
| Invalid email (single) | `Please enter a valid email address (e.g., user@example.com).` |
| Invalid URL | `Please enter a valid URL starting with http:// or https://.` |
| Invalid phone | `Please enter a valid phone number (e.g., +1234567890).` |
| Invalid date | `Please enter a valid date in {format} format.` |
| Invalid number | `Please enter a number between {min} and {max}.` |
| Invalid regex | `Invalid regex pattern: {error detail}.` |
| Invalid value type | `Invalid value type with type '{type}' and value '{value}'.` |
| Only letters allowed | `{Field Name} can only contain letters and spaces.` |
| Only alphanumeric | `{Field Name} can only contain letters, numbers, and underscores.` |

**Override:**

| Context | Message | Lý do |
|---|---|---|
| Gmail Send (nhiều email) | `One or more email addresses are invalid.` | Input là danh sách email, không phải single field |
| Google Sheet: Spreadsheet URL | `Invalid spreadsheet URL format.` | URL pattern đặc thù của Google Sheets |

---

### 1.4 Field Matching

| Rule | Pattern |
|---|---|
| **Default** | `{Field Name} must match {Other Field Name}.` |

---

### 1.5 Selection & Dropdown

| Rule | Pattern |
|---|---|
| No option selected | `{Field Name} is required.` (dùng chung rule 1.1) |
| Selected option no longer exists | `The selected {resource} is no longer available. Please select another.` |
| Empty dropdown list | `No {resources} available. Please check your connection or permissions.` |
| Unsupported value | `Unsupported {field}: {value}.` |
| Invalid enum value | `{Field Name} must be one of: {allowed values}.` |

**Override:**

| Context | Message | Lý do |
|---|---|---|
| Google Drive: role sai context | `Organizer and Content manager roles are only available for Shared Drive. Select Editor, Commenter, or Viewer for My Drive files.` | Business rule đặc thù Shared Drive vs My Drive |
| Google Sheet: Export format | `Export Format contains an unsupported value. Supported formats: PDF, CSV.` | Cần liệt kê rõ formats |

---

### 1.6 Configuration Structure

| Rule | Pattern |
|---|---|
| Config invalid (general) | `Invalid configuration format.` |
| Config validation failed | `{Node} configuration is invalid.` |
| Failed to encode config | `Failed to encode {node} configuration.` |
| Failed to decode config | `Failed to decode {node} configuration.` |
| Mode required | `{Node} mode is required.` |
| Mode invalid | `{Node} mode is invalid.` |
| Delimiter invalid | `Delimiter must be one of: {allowed values}.` |
| Input source required | `{Input source} is required.` |

---

### 1.7 Date & Time

| Rule | Pattern |
|---|---|
| Start after end | `Start Date must not be later than End Date.` |
| Date in past | `{Field Name} must be a future date.` |
| Invalid date range | `Please enter a valid date range.` |

---

### 1.8 Duplicate & Conflict

| Rule | Pattern |
|---|---|
| Duplicate name | `A {resource} with this name already exists. Please use a different name.` |
| Duplicate entry in config | `Duplicate {item} in {context}.` |

---

### 1.9 Node & Connection Names

| Rule | Pattern |
|---|---|
| Node name too long | `Node Name must be 256 characters or less.` |
| Connection name too long | `Connection Name must be 256 characters or less.` |
| Duplicate node name | `A node with this name already exists. Please use a different name.` |

---

## 2. Runtime Validation Errors

Errors xảy ra khi workflow execute và mapped variables được resolve. Retryable: ✗ (tất cả).

### 2.1 Empty Resolved Value

| Rule | Pattern |
|---|---|
| **Default** | `{Field Name} resolved to an empty value. Check your variable mapping.` |
| Expression resolve failed | `Failed to resolve {Field Name}.` |

---

### 2.2 Invalid Resolved Value

| Rule | Pattern |
|---|---|
| Invalid format | `{Field Name} resolved to an invalid value. Check your variable mapping.` |
| Invalid ID | `Invalid {Resource} ID. Please check and re-enter another.` |
| Invalid URL | `{Resource} URL must contain a valid {provider} {resource}.` |
| Length exceeded | `{Field Name} is too long. Please shorten the mapped value and try again.` |

---

### 2.3 Type Mismatch

| Rule | Pattern |
|---|---|
| Expected number | `{Field Name} must be a number. Check your variable mapping.` |
| Expected boolean | `{Field Name} must be true or false. Check your variable mapping.` |
| Expected array | `{Field Name} must be a list. Check your variable mapping.` |

---

### 2.4 Schema Mismatch (Google Sheets)

| Rule | Pattern |
|---|---|
| Header mismatch | `Sheet header does not match the config. Please check headers and reload.` |
| Header row empty | `Header row is empty. Please add headers and retry.` |
| Column not found | `{Column type} '{column}' not found in sheet headers.` |
| Sheet empty | `Sheet is empty, cannot validate column.` |

---

## 3. Authentication & Authorization Errors

Retryable: ✗ (tất cả).

### 3.1 Authentication

| Rule | Pattern |
|---|---|
| **OAuth/account auth failed** | `Authentication failed. Please reconnect your {service} account.` |
| **API key auth failed** | `Invalid or missing API key. Please reconfigure your {service} API key.` |
| Connection deleted | `Selected connection no longer exists. Please update your workflow configuration.` |
| Credential is required | `Credential is required.` |
| Credential not found | `Credential not found.` |
| Failed to retrieve credential | `Failed to retrieve credential.` |

> **Phân biệt OAuth vs API Key:** OAuth services (Google Drive, Gmail, Google Calendar, Facebook) dùng pattern "reconnect account". API Key services (OpenAI, Claude, Gemini) dùng pattern "reconfigure API key". Đây là override có lý do: hành động khắc phục khác nhau.

---

### 3.2 Authorization & Permission

| Rule | Pattern |
|---|---|
| Insufficient permission (general) | `The connected account does not have sufficient rights to {action}.` |
| Insufficient permission (specific) | `Cannot {action}: the connected account does not have {permission} access to this {resource}. Check the sharing settings.` |
| Read permission denied | `Cannot read {resource}: the connected account does not have read access to this {resource}. Check the sharing settings.` |
| Write permission denied | `Cannot {action} {resource}: the connected account does not have write access to this {resource}. Check the sharing settings.` |
| Delete permission denied | `Cannot delete {resource}: the connected account does not have delete access to this {resource}. Check the sharing settings.` |
| Cannot create in location | `You do not have permission to {action} in the selected location.` |
| Cannot update permission | `Cannot update permission: the connected account does not have sufficient rights. Owner or Organizer role required.` |
| Inherited permission | `Cannot update inherited permission. Update the permission at the parent folder level instead.` |
| Restricted access (batch) | `Action could not be processed in some {resources} due to restricted access. Please check your permissions.` |

**Override (Google Drive — business rules đặc thù):**

| Context | Message | Lý do |
|---|---|---|
| Trash: not owner | `Cannot move file to Trash: the connected account is not the file owner. Only the owner can trash a file.` | Google Drive ownership rule |
| Trash: Shared Drive | `Cannot trash file in Shared Drive: Organizer role is required.` | Shared Drive role hierarchy |
| Cannot change owner role | `Cannot update owner's role. Use Transfer Ownership instead.` | Google Drive-specific flow |
| Sharing blocked by org | `Sharing is blocked by the organization's policy. Contact your admin to allow external link sharing.` | Google Workspace policy |
| Public access inherited | `Public access is inherited from the Shared Drive and cannot be removed at the file level. Adjust it in Shared Drive settings.` | Shared Drive inheritance |

---

### 3.3 Account & Scope

| Rule | Pattern |
|---|---|
| Missing OAuth scope | `The connected account does not have the required permissions. Please reconnect with the necessary scopes.` |
| Workspace permission denied | `You do not have permission to perform this action in this workspace.` |
| Account not connected | `Account not connected.` |
| Account not found | `Account not found.` |
| Unable to access account | `Unable to access account.` |
| Model access denied | `Selected {service} account does not support model {model}. Please select another model or connect an account with access.` |

---

## 4. Resource Errors

Retryable: ✗ (trừ khi ghi khác).

### 4.1 Resource Not Found

| Rule | Pattern |
|---|---|
| **Default** | `{Resource} not found or access has been revoked.` |
| No matches | `No matches found.` |
| No results | `No {resource} matches your criteria.` |
| Row out of range | `Row {N} does not exist. Sheet only has data up to row {rowCount}.` |
| Selected resource deleted | `The selected {resource} no longer exists. Please select another.` |

**Override:**

| Context | Message | Lý do |
|---|---|---|
| Google Sheet: spreadsheet trashed | `Spreadsheet not found: The file does not exist, or it has been moved to the trash. Please verify the spreadsheet ID.` | Gợi ý nguyên nhân cụ thể (trash) giúp user fix nhanh hơn |
| Google Drive: Delete 404 | `File not found. It may have been deleted or the connected account may not have access.` | Phân biệt deleted vs permission, tránh user tìm sai hướng |
| Google Sheets: sheet tab | `Sheet "{sheet_name}" not found in spreadsheet.` | Cần chỉ rõ sheet name để user fix đúng chỗ |

---

### 4.2 Resource Type & State

| Rule | Pattern |
|---|---|
| Wrong resource type | `Only {expected type} can be {action}. {actual type} {action} is not supported.` |
| Unsupported type | `{Resource type} is not supported for this action.` |
| Already exists | `{Resource} already exists. Please use a different name or enable update mode.` |
| Locked | `{Resource} is currently locked by another process. Please try again later.` | ✓ retryable |
| Archived | `{Resource} is archived and cannot be modified.` |
| Deleted | `{Resource} has been deleted and cannot be accessed.` |
| In trash | `{Resource} is in trash. Please restore it first.` |
| Read-only | `{Resource} is read-only and cannot be modified.` |
| Limit exceeded | `{Resource} has reached the maximum number of {items}.` |
| Folder mismatch | `{Resource} does not belong to selected {container}.` |
| Event conflict | `Event conflicts with an existing event.` |

---

## 5. Network & Provider Errors

Retryable: ✓ (tất cả, trừ khi ghi khác).

### 5.1 Timeout

| Rule | Pattern |
|---|---|
| **Default** | `{Action} timed out. Please try again or use shorter input.` |
| Connection timeout | `Connection to {service} timed out. Please try again.` |
| Read timeout | `Reading data from {service} timed out. Please try again.` |

**Override:**

| Context | Message | Lý do |
|---|---|---|
| Extract From File | `The file is taking too long to process. Please try a different file or reduce its size.` | Instruction khác: giảm file size, không phải "shorter input" |

---

### 5.2 Rate Limit

| Rule | Pattern |
|---|---|
| **Provider rate limit (auto-retry)** | `{Service} rate limit reached. Retrying with exponential backoff (max 3 attempts).` |
| **Provider rate limit (manual-retry)** | `Rate limit exceeded. Please try again later.` |
| **Platform rate limit** | `You have exceeded the rate limit for this action. Please try again in {time}.` |

**Override:**

| Context | Message | Lý do |
|---|---|---|
| Google 429 (batch actions) | `Exceeding Request Rate Limits: Only items within the limit were processed.` | Partial success: một số items đã xử lý, chỉ phần còn lại bị rate limit |

---

### 5.3 Provider Unavailable

| Rule | Pattern |
|---|---|
| **Default** | `{Service} is currently unavailable. Please try again later.` |
| Service maintenance | `{Service} is undergoing maintenance. Please try again later.` |
| API unexpected error | `{Service} API returned an unexpected error. Please try again later.` |
| External service error | `External service error.` |

---

### 5.4 Network & Connection

| Rule | Pattern |
|---|---|
| Connection failed | `Failed to connect to {service}. Please check your internet connection.` |
| DNS failed | `Failed to resolve {service} address. Please check your network settings.` |
| SSL/TLS failed | `Secure connection to {service} failed. Please try again.` |
| HTTP request failed | `Failed to send HTTP request.` |
| HTTP create request failed | `Failed to create HTTP request.` |
| API non-success | `API returned non-success status.` |

---

## 6. Storage & Quota Errors

Retryable: ✗ (tất cả).

### 6.1 Storage Quota

| Rule | Pattern |
|---|---|
| Provider storage | `{Action} failed: storage quota has been exceeded.` |
| Platform storage | `Platform storage quota has been exceeded. Please contact your administrator.` |
| Workspace storage | `Workspace storage quota has been exceeded. Please upgrade your plan or free up space.` |

---

### 6.2 API Quota

| Rule | Pattern |
|---|---|
| Model quota | `Quota for model {model_name} has been exceeded. Please select another model.` |
| Daily quota | `Daily quota for {service} has been exceeded. Please try again tomorrow.` |
| Monthly quota | `Monthly quota for {service} has been exceeded. Please upgrade your plan.` |

**Override:**

| Context | Message | Lý do |
|---|---|---|
| AI Chat/Conversation/Summarize | `AI quota limit reached. Please contact system administrator and try again.` | Internal AI service, user không thể tự fix quota |
| Gemini API | `API quota exceeded. Please check your billing or wait for quota reset.` | User tự quản lý Gemini billing |

---

### 6.3 Plan & Feature Limits

| Rule | Pattern |
|---|---|
| Feature not in plan | `This feature is not available in your current plan. Please upgrade to access it.` |
| Workflow limit | `You have reached the maximum number of workflows for your plan. Please upgrade or delete unused workflows.` |
| Execution limit | `You have reached the monthly execution limit for your plan. Please upgrade or wait until next month.` |

---

## 7. Binary File Errors

Retryable: ✗ (tất cả, trừ khi ghi khác).

### 7.1 File Size

| Rule | Pattern |
|---|---|
| Too large | `{Field Name} cannot exceed {max} MB.` |
| Empty | `{Field Name} cannot be empty.` |
| Total too large | `Total file size cannot exceed {max} MB.` |

---

### 7.2 File Type & MIME

| Rule | Pattern |
|---|---|
| Unsupported type | `{Field Name} must be a valid {format}.` |
| Invalid MIME | `{Field Name} has an invalid file type. Supported types: {types}.` |
| Executable blocked | `Executable files are not allowed for security reasons.` |
| Unknown type | `File type is unknown.` |
| No Content-Type | `Cannot determine file type from URL because response Content-Type header is missing.` |

---

### 7.3 File Content & Reference

| Rule | Pattern |
|---|---|
| Corrupted | `{Field Name} appears to be corrupted. Please upload a valid file.` |
| Invalid structure | `{Field Name} has an invalid structure. Please check the file format.` |
| Virus detected | `{Field Name} failed security scan. Please upload a different file.` |
| File reference invalid | `File reference is invalid.` |
| Failed to read | `Failed to read file content.` |
| Failed to download | `Failed to download file content.` |

---

## 8. Workflow Execution Errors

### 8.1 Execution State

| Rule | Pattern | Retryable |
|---|---|---|
| Workflow not found | `Workflow not found. It may have been deleted.` | ✗ |
| Workflow disabled | `This workflow is currently disabled. Please enable it to run.` | ✗ |
| Workflow archived | `This workflow is archived and cannot be executed.` | ✗ |
| Already running | `This workflow is already running. Please wait for it to complete.` | ✓ |
| Max retries exhausted | `Operation failed after maximum retries.` | ✗ |
| All items failed | `Execution fail: {error detail}.` | ✗ |

---

### 8.2 Step & Dependency

| Rule | Pattern | Retryable |
|---|---|---|
| Missing upstream output | `Required data from step "{step_name}" is not available.` | ✗ |
| Invalid variable reference | `Variable "{variable}" does not exist. Please check your mapping.` | ✗ |
| Circular dependency | `Circular dependency detected between steps. Please review your workflow.` | ✗ |

---

### 8.3 Execution Timeout

| Rule | Pattern | Retryable |
|---|---|---|
| Step timeout | `Step "{step_name}" timed out after {time} seconds.` | ✓ |
| Workflow timeout | `Workflow execution timed out after {time} minutes.` | ✓ |

---

### 8.4 Parsing & Encoding

| Rule | Pattern | Retryable |
|---|---|---|
| Parse input failed | `Failed to parse input.` | ✗ |
| Encode payload failed | `Failed to encode payload.` | ✗ |
| Decode response failed | `Failed to decode response.` | ✗ |
| Read response failed | `Failed to read response.` | ✗ |
| Marshal payload failed | `Failed to marshal payload.` | ✗ |
| JSON parse failed | `JSON cannot be parsed.` | ✗ |
| Read request body failed | `Failed to read request body.` | ✗ |
| Unmarshal config failed | `Failed to unmarshal {node} configuration.` | ✗ |

---

### 8.5 Service Operation Failed

Pattern chung: `Failed to {action}.` — áp dụng cho mọi service khi API call thất bại ở bước cuối.

| Rule | Pattern | Retryable |
|---|---|---|
| Failed to initialize service | `Failed to initialize {service} service.` | ✗ |
| Failed to retrieve resource | `Failed to retrieve {resource}.` | ✗ |
| Failed to read data | `Failed to read {data description}.` | ✗ |
| Failed to create resource | `Failed to create {resource}.` | ✗ |
| Failed to update resource | `Failed to update {resource}.` | ✗ |
| Failed to delete resource | `Failed to {delete action} {resource}.` | ✗ |
| Failed to search | `Failed to search {resources}.` | ✗ |
| Failed to export | `Failed to export {resource}.` | ✗ |
| Failed to upload | `Failed to upload {resource} to {service}.` | ✗ |
| Failed to send message | `Failed to send message.` | ✗ |
| Unsupported operation | `Unsupported operation.` | ✗ |
| Invalid operation | `Invalid {service} operation.` | ✗ |

---

## 9. Content Policy & AI-Specific Errors

| Rule | Pattern | Retryable |
|---|---|---|
| Content policy violation | `Content violates {service} usage policies. Please modify your input.` | ✗ |
| Safety filter triggered | `Content was blocked by safety filters. Please modify your input.` | ✗ |
| Model not available | `Model "{model_name}" is not available. Please select another model.` | ✗ |
| Token limit exceeded | `Input exceeds token limit for model "{model_name}". Please shorten your input.` | ✗ |
| No output returned | `No output returned from {service}.` | ✗ |
| Empty output returned | `{Service} returned empty output.` | ✗ |
| Model overloaded | `Model is currently overloaded. Please try again later.` | ✓ |
| Model not support input type | `Configured model does not support {input type} input.` | ✗ |

---

## Retry Strategy

| Error Type | Retry | Max Attempts | Backoff |
|---|---|---|---|
| Rate limit | ✓ | 3 | Exponential (2s → 4s → 8s) |
| Timeout | ✓ | 3 | Exponential (2s → 4s → 8s) |
| Network | ✓ | 3 | Exponential (2s → 4s → 8s) |
| Provider unavailable | ✓ | 3 | Exponential (2s → 4s → 8s) |
| Resource locked | ✓ | 3 | Exponential (2s → 4s → 8s) |
| Authentication | ✗ | 0 | N/A |
| Authorization | ✗ | 0 | N/A |
| Validation | ✗ | 0 | N/A |
| Not found | ✗ | 0 | N/A |
| Quota exceeded | ✗ | 0 | N/A |

---

## Error Message Principles

1. **User-first language** — Không expose technical jargon (error codes, stack traces, internal field names).
2. **Actionable** — Luôn nói user phải làm gì tiếp theo.
3. **Specific** — Nói rõ field nào, resource nào, service nào bị lỗi.
4. **Consistent** — Follow pattern mặc định. Override chỉ khi context buộc phải khác, và phải ghi vào catalog.
5. **No new variants** — Khi thêm error mới, dùng pattern có sẵn. Không tạo wording mới cho cùng loại lỗi.

---

## Error Logging

Mọi error phải log với: Error code (internal), Error message (user-facing), Timestamp, User ID, Workflow ID, Step ID, Provider (nếu có), Request ID (nếu có), Stack trace (internal only).

---

## Quick Reference

| Scenario | Message Pattern | Retryable |
|---|---|---|
| Required field empty | `{Field} is required.` | ✗ |
| Length exceeded | `{Field} must be {max} characters or less.` | ✗ |
| Invalid format | `Please enter a valid {format}.` | ✗ |
| Variable resolves empty | `{Field} resolved to an empty value. Check your variable mapping.` | ✗ |
| Variable resolves invalid | `{Field} resolved to an invalid value. Check your variable mapping.` | ✗ |
| Auth failed (OAuth) | `Authentication failed. Please reconnect your {service} account.` | ✗ |
| Auth failed (API key) | `Invalid or missing API key. Please reconfigure your {service} API key.` | ✗ |
| Permission denied | `Cannot {action}: the connected account does not have {permission} access.` | ✗ |
| Resource not found | `{Resource} not found or access has been revoked.` | ✗ |
| Rate limit | `{Service} rate limit reached. Retrying with exponential backoff (max 3 attempts).` | ✓ |
| Timeout | `{Action} timed out. Please try again or use shorter input.` | ✓ |
| Provider unavailable | `{Service} is currently unavailable. Please try again later.` | ✓ |
| Quota exceeded | `Quota for {resource} has been exceeded.` | ✗ |
| File too large | `{Field} cannot exceed {max} MB.` | ✗ |
| Unsupported file type | `{Field} must be a valid {format}.` | ✗ |

---

## Changelog

| Date | Change | Author |
|---|---|---|
| 2026-05-28 | Initial version (error-message-existed.md) | AI BA |
| 2026-06-15 | Merged with PDF (436 entries). Full detailed catalog. | AI BA |
| 2026-06-15 | Refactored to rule-based format. Collapsed specific cases into patterns. Added override tables. Added section 9 (Content Policy & AI-Specific). | AI BA |
| 2026-06-16 | Merged error-message-existed.md into this file. Added: 1.3 special chars, 1.9 Node/Connection Names, 3.2 specific permission sub-patterns, 3.3 workspace permission, 5.1 read timeout, 5.3 service maintenance, 4.1 Google Sheets sheet tab override, 4.2 event conflict, Quick Reference table. | AI BA |
