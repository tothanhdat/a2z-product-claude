---
name: Write User Story
description: Viết ticket scope nhỏ — thêm/điều chỉnh tính năng cho node/trigger/action đã có. Tập trung vào delta (phần mới/thay đổi), không mô tả lại phần đã tồn tại.

---

# Skill: Write User Story (Delta Ticket)

## Khi nào kích hoạt

Khi user yêu cầu viết ticket cho **một thay đổi nhỏ** trên node/trigger/action **đã tồn tại**:
- Bổ sung tính năng (thêm field, thêm option, thêm behavior).
- Điều chỉnh tính năng (thay đổi default, đổi validation rule, đổi flow nhỏ).
- Thêm tab/section mới (e.g., bổ sung tab Advanced).
- Tăng cường UX (thêm error message, thêm confirmation, thêm progress indicator).

> **Phân biệt rõ với `write-ticket`:**
> - `write-ticket` = viết **full ticket** cho **node/trigger/action MỚI hoàn toàn** (có Connect Story, full Inputs/Outputs, full Business Rules).
> - `write-user-story` = viết **delta ticket** chỉ cho **phần thêm mới hoặc thay đổi** trên node đã có. KHÔNG lặp lại phần đã tồn tại.

Nếu user yêu cầu viết ticket cho một node/action/trigger HOÀN TOÀN MỚI, dừng lại và route sang skill `write-ticket`.

---

## Nguyên tắc cốt lõi

1. **Delta-only** — chỉ mô tả phần mới hoặc phần thay đổi. KHÔNG mô tả lại Connect step, Account/Action fields, các input fields cũ, business rules cũ, error messages chuẩn của hệ thống.
2. **Tieback rõ ràng** — luôn reference ticket cha hoặc sprint/feature gốc (e.g., "Tiếp nối ticket MART-1866", "Bổ sung cho Sprint 12").
3. **Adaptive depth** — độ chi tiết khớp với scope:
   - **Minimum (cực đơn giản):** Epic Summary + User Stories. Hết.
   - **Standard (đa số case):** Epic Summary + User Stories + Out of Scope + (Inputs/Outputs/Business Rules/Validation/Error Handling chỉ những gì NEW).
4. **Không có Story 1 fixed template** — Connect step đã tồn tại, không cần viết lại.
5. **Không có Error Handling Story fixed template** — chỉ thêm story này khi có error scenarios MỚI thực sự đặc thù cho thay đổi này.

---

## Quy trình bắt buộc

### Bước 1: Xác nhận scope là delta

Đọc nhanh request để confirm đây là thay đổi/bổ sung trên node đã có. Nếu mơ hồ giữa "node mới" vs "thêm tính năng cho node cũ", **hỏi 1 câu** để confirm trước khi tiếp tục.

Ví dụ: *"Đây là thêm tính năng cho [node hiện có] đúng không, hay viết lại full ticket cho node mới? Nếu là thay đổi/bổ sung, mình sẽ viết theo style delta — chỉ mô tả phần mới."*

### Bước 2: Thu thập thông tin

Hỏi user **tối đa 3 câu hỏi** để làm rõ scope của delta. Không hỏi nhiều hơn 3. Bỏ qua nếu request đã đủ rõ.

**Bắt buộc:** Mỗi câu hỏi phải kèm **suggested answer**. User chỉ cần reply "OK hết" hoặc chỉnh từng câu.

**Các câu hỏi nên hỏi:**
1. **Parent ticket / context gốc** — Thay đổi này tiếp nối ticket nào? Hoặc thuộc sprint/feature nào? (Nếu user chưa nói)
2. **Phạm vi delta** — Cụ thể những gì được thêm/đổi? Có ảnh hưởng đến field/behavior nào đã có không?
3. **Edge case đặc thù** — Có scenario nào khác standard mà delta này phải handle không?

**KHÔNG hỏi:**
- Connect step / Account / Action field — đã chuẩn hoá.
- Retry, auth, permission — dùng logic hệ thống.
- Input fields cũ — không thuộc scope delta.

**Format câu hỏi:**

```
N. [Câu hỏi]
   **Đề xuất:** [Câu trả lời gợi ý cụ thể]
   *(Lý do: [1 câu giải thích])*
```

### Bước 3: Tham chiếu existing nodes

Đọc `references/existing-nodes.md` để biết:
- Node gốc đã có logic gì để reference.
- Pattern tương tự đã dùng ở node nào để tái sử dụng wording.

### Bước 4: Chọn depth phù hợp

Quyết định viết theo **Minimum** hay **Standard** dựa trên scope:

| Tín hiệu | Depth |
|---|---|
| 1–2 field thay đổi nhỏ, không có business rule mới, không có error message mới | **Minimum** |
| Có ≥ 1 field mới, có business rule mới, hoặc có error scenario mới | **Standard** |
| Thay đổi nhiều behavior, ảnh hưởng UX lớn | **Standard** (đầy đủ section) |

### Bước 5: Viết ticket theo template

Dùng template trong section "Output Template" bên dưới. Tuân theo format AC trong `references/ticket-template-lean.md` (Given/When/Then, AC.NN với title bold, dropdown embed table).

Tuân theo error message catalog `references/error-message-catalog.md` cho các error mới.

### Bước 6: Lưu file

Lưu ticket vào:
```
specs/[tên-node-hoặc-feature]-[delta-suffix]/final-ticket.md
```

**Quy tắc đặt tên thư mục:**
- Lowercase, dấu gạch ngang.
- Suffix mô tả delta: `-response-mode-auth`, `-add-bcc-field`, `-quality-option`, etc.
- Ví dụ:
  - `specs/human-input-web-form-response-mode-auth/final-ticket.md`
  - `specs/gmail-send-email-add-bcc/final-ticket.md`
  - `specs/openai-generate-image-quality-option/final-ticket.md`

### Bước 7: Tạo file error-message.md (chỉ khi có error mới)

**Bỏ qua bước này nếu delta KHÔNG introduce error message mới.**

Nếu có error message mới, tạo `error-message.md` cùng thư mục với format bảng sau — chỉ liệt kê các error MỚI, không list lại error đã có ở ticket cha:

```markdown
# Error Messages: [Platform] – [Action/Trigger Name]

| ID | Services | Action/Trigger | Category | Test Existence in App (QC) | Error Context | User Error Message (BA) | User Instruction (BA) |
|---|---|---|---|---|---|---|---|
| 1 | [Service] | [Action/Trigger: Name] | [Category] | FALSE | [Mô tả ngắn context] | [Error message chính xác] | [Hướng dẫn user — nếu có] |
```

**Category:** `Validation`, `Execution`, `Authentication`, `Authorization`, `Not Found`, `Rate Limit`, `Provider Error`.

### Bước 8: Hỏi user có muốn review không

Sau khi lưu file, hỏi đúng câu:

> "Ticket đã được lưu. Bạn có muốn tôi review và chỉnh lại không?"

Nếu user đồng ý → chạy Self-Review Checklist bên dưới và update file. Nếu từ chối → dừng.

---

## Output Template

### Minimum Depth (cực đơn giản)

```markdown
# [Type] - [Node Name] - [Delta Description]

## Epic Summary

**Epic Name:** [Node Name] – [Delta Description]

**Goal:**
[1–2 câu. Bổ sung/điều chỉnh gì, cho ai, để làm gì. Business language.]

**Context:**
[Tiếp nối ticket nào / thuộc feature nào. Phạm vi delta là gì. Max 3 câu.]

**Expected UI Fields:**
- [Chỉ liệt kê field MỚI hoặc field bị thay đổi]

---

## User Stories

### Story 1: [Title — verb phrase mô tả delta]

**Story:** As an automation workflow builder, I want [delta capability], so that [outcome].

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. [Title]**
Given ...,
When ...,
Then ...

**AC.02. [Title]**
Given ...,
When ...,
Then ...
```

### Standard Depth

```markdown
# [Type] - [Node Name] - [Delta Description]

## Epic Summary

**Epic Name:** [Node Name] – [Delta Description]

**Goal:**
[1–2 câu business language.]

**Context:**
[Tiếp nối ticket nào, scope của delta, key technical notes nếu cần. Max 5 câu.]

**Expected UI Fields:**
- [Chỉ field MỚI hoặc thay đổi — không list lại field cũ]

---

## User Stories

### Story 1: [Title]

**Story:** ...
**Priority:** ...
**Acceptance Criteria:**

**AC.01. [Title]**
Given ...,
When ...,
Then ...

[... thêm story khác nếu delta cover nhiều behavior độc lập]

---

## Out of Scope

- [Liệt kê những gì user/stakeholder có thể tưởng nằm trong delta nhưng KHÔNG]
- [Tính năng liên quan nhưng defer sang ticket khác]

---

## Inputs (chỉ field MỚI hoặc thay đổi)

| Field | Type | Required | Default | Manual Input | Expression | Description |
|---|---|---|---|---|---|---|
| [Field mới] | ... | ... | ... | ... | ... | [Note ngắn] |

---

## Outputs (chỉ field MỚI nếu có)

| Field | Type | Description | Nullable |
|---|---|---|---|
| [Field mới] | ... | ... | ... |

> Nếu delta không thay đổi output → bỏ section này hoàn toàn.

---

## Business Rules (chỉ rule MỚI cho delta)

1. [Rule mới do delta tạo ra]
2. ...

> Không list lại rules đã có ở ticket cha.

---

## Validation Rules (chỉ rule MỚI)

### Config-time Validation
| Scenario | Message |
|---|---|
| [New scenario] | "[Message]" |

### Runtime Validation (nếu có)
| Scenario | Message |
|---|---|
| [New scenario] | "[Message]" |

---

## Error Handling (chỉ error MỚI)

| Scenario | Message |
|---|---|
| [New error scenario] | "[Message]" |

> Không list lại error đã có sẵn (auth, permission, rate limit) trừ khi delta thay đổi message.

---

## Assumptions

1. [Assumption về platform/parent ticket cần thiết để delta hoạt động]
2. ...
```

---

## Self-Review Checklist (chỉ dùng khi user đồng ý ở Bước 8)

Review theo thứ tự. Sửa trực tiếp vào file — không liệt kê ra chat.

**Scope & Delta-Focus**
- [ ] Ticket có tieback rõ ràng tới parent ticket / sprint gốc trong Context
- [ ] Không mô tả lại Connect step, Account field, Action field
- [ ] Không lặp lại input fields đã có ở ticket cha
- [ ] Không lặp lại business rules đã có ở ticket cha
- [ ] Không lặp lại error messages standard (auth, rate limit, permission) trừ khi delta thay đổi

**Depth Phù Hợp**
- [ ] Nếu delta đơn giản (1–2 fields, không có rule mới) → dùng Minimum, không thêm section thừa
- [ ] Nếu delta phức tạp → có đủ section cần thiết, không thiếu

**AC Quality**
- [ ] AC dùng Given/When/Then format chuẩn, mỗi keyword trên dòng riêng
- [ ] AC ID sequential với one-line title English bold
- [ ] Mỗi AC một observable behavior
- [ ] Dropdown fields có embed table với UI Label + API Value
- [ ] Không có ngôn ngữ vague ("works correctly", "user-friendly")

**Error Messages**
- [ ] Error message mới đúng format catalog (capitalise, end với `.`, có "what + what to do")
- [ ] UI labels dùng Title Case canonical

**Out of Scope**
- [ ] Liệt kê được những gì stakeholder có thể tưởng là in-scope nhưng không
- [ ] Có reference ticket khác / phase khác nếu defer

**Lean Rules**
- [ ] Dùng standard reference sentences cho logic đã có
- [ ] Không có section trống hoặc chỉ chứa "—"
- [ ] Không có Implementation/Technical Notes thừa

Sau khi review xong, update file và thông báo ngắn gọn các sửa đổi.

---

## Ví dụ interaction

**User:** Viết ticket bổ sung Response Mode và Authentication cho Web Form Trigger (đã có từ Sprint 12, ticket gốc MART-1866).

**AI (hỏi tối đa 3 câu, kèm đề xuất):**

1. Response Mode bao gồm những options nào và default là gì?
   **Đề xuất:** 2 options — "Form Is Submitted" (default, phản hồi ngay) và "Workflow Finishes" (chờ workflow chạy xong, timeout 5 phút).
   *(Lý do: Pattern phổ biến cho web form triggers — cho user chọn giữa instant feedback và end-to-end status.)*

2. Authentication có những phương thức nào và áp dụng cho Draft hay Publish URL?
   **Đề xuất:** 2 options — "None" (default) và "Basic Auth" (yêu cầu credential). Áp dụng cho cả Draft và Publish URL.
   *(Lý do: Basic Auth là minimum viable cho form bảo mật, OAuth có thể defer sang phase sau.)*

3. Khi Response Mode = "Workflow Finishes" và workflow timeout, behavior mong muốn?
   **Đề xuất:** Form page hiển thị message graceful timeout (e.g., "Processing is taking longer than usual..."), workflow vẫn chạy ngầm.
   *(Lý do: Tránh user bị stuck ở form page, đồng thời không drop workflow execution.)*

**User:** OK hết.

**AI:** (viết ticket Standard depth — có Epic Summary, 3 stories, Out of Scope, Inputs, Business Rules, Validation, Error Handling, Assumptions; lưu file; hỏi review)

---

## Ví dụ scope siêu đơn giản (Minimum depth)

**User:** Trong action Gmail – Send Email, thêm option "Cc" và "Bcc". Cả hai đều optional, support multiple emails phân tách bằng dấu phẩy.

**AI (hỏi 1 câu confirm):**

1. Validation cho Cc/Bcc: format email check ở config-time với manual input đúng không? Mapped variable thì check runtime?
   **Đề xuất:** Đúng — manual input check email format ngay; mapped variable chỉ check runtime nếu resolved value invalid.
   *(Lý do: Pattern chuẩn của các email field trên platform.)*

**User:** OK.

**AI:** (viết ticket Minimum depth — chỉ Epic Summary + 1 user story với 4–5 ACs; không có Out of Scope/Business Rules section vì không có gì mới ngoài AC; lưu file)
