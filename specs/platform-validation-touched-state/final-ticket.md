# Bug Fix - Platform Validation - Prevent Premature Validation on Untouched Fields

## Epic Summary

**Epic Name:** Platform Validation – Prevent Premature Validation on Untouched Fields

**Goal:**
Sửa lỗi validation error hiển thị ngay khi field mới được render mặc dù user chưa tương tác, đảm bảo trải nghiệm cấu hình workflow mượt mà và không gây nhầm lẫn cho user.

**Context:**
Hiện tại, khi một field mới được tạo hoặc render (ví dụ: thêm form field trong Human Input – Web Form, hoặc conditional field xuất hiện sau khi thay đổi dropdown), hệ thống chạy validation ngay lập tức và hiển thị error như "Label is required." dù user chưa tương tác với field. Bug này xảy ra cross-feature trên toàn platform, không riêng một node nào. Ticket này định nghĩa quy tắc "touched state" chung cho validation behavior của platform.

---

## User Stories

### Story 1: Prevent validation on newly rendered fields

**Story:** As an automation workflow builder, I want validation errors to only appear after I interact with a field, so that I am not confused by error messages on fields I haven't touched yet.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. No validation error on initial field render**
Given một field required vừa được render lần đầu (tạo mới, conditional field xuất hiện, hoặc page load),
When field đó chưa được user tương tác (chưa focus, chưa nhập, chưa blur),
Then hệ thống KHÔNG hiển thị validation error,
And field border ở trạng thái default (không phải error state),
And "Continue" button vẫn disabled nếu field required còn trống nhưng không hiển thị inline error

**AC.02. Validation fires after user touches then leaves field empty**
Given một required field đã được user focus (click vào hoặc tab vào),
When user blur ra khỏi field mà không nhập giá trị (hoặc nhập rồi xoá hết),
Then hệ thống hiển thị validation error ngay dưới field theo message catalog hiện có (ví dụ: `"{Field Name} is required."`),
And field border chuyển sang error state

**AC.03. Validation fires when user clears an existing value**
Given một required field đã có giá trị (do user nhập trước đó hoặc do default),
When user xoá hết giá trị và blur ra khỏi field,
Then hệ thống hiển thị validation error ngay dưới field,
And field border chuyển sang error state

**AC.04. Continue/Save button triggers validation on all untouched required fields**
Given có required fields chưa được touched và còn trống,
When user click "Continue" hoặc "Save",
Then hệ thống hiển thị validation error cho tất cả required fields còn trống (bao gồm cả fields chưa touched),
And "Continue"/"Save" bị block

**AC.05. Conditional field appears without premature validation**
Given user thay đổi giá trị một field (ví dụ: dropdown) làm xuất hiện conditional field mới,
When conditional field mới render,
Then conditional field KHÔNG hiển thị validation error dù là required,
And validation chỉ trigger sau khi user tương tác với conditional field đó hoặc click "Continue"/"Save"

**AC.06. Dynamically added fields (Add button) without premature validation**
Given user click button "Add" để tạo field mới (ví dụ: Add Form Field trong Human Input – Web Form),
When field mới được thêm vào form,
Then field mới KHÔNG hiển thị validation error,
And validation chỉ trigger sau khi user tương tác với field đó hoặc click "Continue"/"Save"

**Edge Cases:**
- User focus vào field rồi blur ngay mà không nhập gì → validation fires vì field đã được touched.
- Field có default value khi render → không hiển thị error vì field không trống.
- Multiple required fields trống khi click Continue → tất cả đều hiển thị error cùng lúc.

---

## Out of Scope

- Thay đổi nội dung validation messages — ticket này chỉ sửa **timing** hiển thị, không thay đổi message text.
- Thay đổi validation rules (required, format, length) — logic validation giữ nguyên, chỉ thay đổi thời điểm trigger.
- Validation behavior cho runtime (workflow execution) — ticket này chỉ cover config-time validation trên UI.
- Redesign validation UX (ví dụ: thay đổi vị trí hiển thị error, thay đổi màu sắc error state) — giữ nguyên visual hiện tại.

---

## Business Rules (chỉ rule MỚI cho delta)

1. Một field được coi là "touched" khi user đã focus vào field đó ít nhất một lần.
2. Validation error chỉ hiển thị inline cho field đã touched và có giá trị invalid/trống.
3. Khi user click "Continue" hoặc "Save", tất cả required fields được coi là touched — validation fires cho mọi field trống bất kể trạng thái touched trước đó.
4. Field mới được render (initial load, conditional appear, dynamic add) luôn ở trạng thái "untouched" cho đến khi user tương tác.
5. Quy tắc này áp dụng cho tất cả field types trên toàn platform: text input, dropdown, textarea, multi-select, toggle, file input.

---

## Assumptions

1. Platform có cơ chế quản lý field state (hoặc có thể bổ sung) để track trạng thái "touched" của mỗi field.
2. Tất cả các feature hiện tại đang dùng chung validation framework — fix ở tầng framework sẽ áp dụng cross-feature.
3. "Continue" button disabled logic hiện tại (disabled khi có field required trống) giữ nguyên — chỉ thay đổi việc hiển thị inline error message.
