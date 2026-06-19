---
name: write-ticket
description: "Viết ticket Lean cho action/trigger mới. Hỏi tối đa 5 câu hỏi trước khi output ticket hoàn chỉnh."
---

# Skill: Write Ticket

## Khi nào kích hoạt

Khi user yêu cầu viết ticket cho một action, trigger, hoặc feature mới trên platform.

---

## Quy trình bắt buộc

### Bước 1: Thu thập thông tin

Trước khi viết ticket, **bắt buộc hỏi user tối đa 5 câu hỏi** để làm rõ scope. Không hỏi nhiều hơn 5 câu. Không hỏi những gì đã rõ ràng từ context.

**Bắt buộc:** Mỗi câu hỏi phải kèm theo **câu trả lời đề xuất** (suggested answer) để user có thể tham khảo, confirm nhanh, hoặc điều chỉnh.

**Nguyên tắc chọn câu hỏi:**
- Chỉ hỏi khi thiếu thông tin sẽ ảnh hưởng trực tiếp đến scope, business rules, hoặc output design.
- Không hỏi về platform behaviors đã chuẩn hoá (retry, auth, connect step) — assume theo standards.
- Không hỏi về implementation details (đó là scope của Engineering).
- Ưu tiên hỏi về: business behavior đặc thù, edge cases quan trọng, input/output expectations, và scope boundaries.

**Các loại câu hỏi thường cần:**
1. **Scope boundary** — Action/trigger này cover những gì, exclude những gì?
2. **Core behavior** — Behavior chính khi happy path? Có gì khác biệt so với node tương tự đã có?
3. **Input fields** — Có field nào đặc biệt ngoài pattern chuẩn? Conditional fields? Khi hỏi về input fields, bắt buộc hỏi: required? default value? support dynamic variable?
4. **Output expectations** — Downstream steps cần data gì từ node này?
5. **Edge cases / business rules đặc thù** — Có rule nào khác standard platform behavior?

**Nếu user đã cung cấp đủ thông tin** (ví dụ: paste API docs, mô tả chi tiết behavior), có thể hỏi ít hơn 5 câu hoặc confirm assumptions thay vì hỏi mở.

**Format câu hỏi bắt buộc:**

```
N. [Câu hỏi cụ thể]
   **Đề xuất:** [Câu trả lời gợi ý dựa trên patterns đã có hoặc best practices]
   *(Lý do: [1 câu giải thích — tham chiếu node tương tự, convention, hoặc product context])*
```

**Nguyên tắc viết suggested answer:**
- Đề xuất phải concrete và actionable, không vague.
- Reference existing nodes hoặc conventions khi có thể.
- Nếu có nhiều options hợp lý, list 2–3 options và highlight option đề xuất.
- Đề xuất phải align với product context: end-user-first, simple surface, business-friendly.
- User chỉ cần reply "OK đề xuất 1, 2, 3" hoặc điều chỉnh từng câu là đủ.

### Bước 2: Tham chiếu existing nodes & standards

Đọc các file sau:
- `/Users/ryan/Documents/Claude/Projects/A2Z Product Claude/references/existing-nodes.md` — xác định node cùng provider đã có logic gì reusable, tránh duplicate.
- `/Users/ryan/Documents/Claude/Projects/A2Z Product Claude/references/ticket-template-lean.md` — **source of truth** cho ticket structure, AC format, language rules, validation behavior rules, và tất cả format conventions.
- `/Users/ryan/Documents/Claude/Projects/A2Z Product Claude/references/error-message-existed.md` — error message catalog.
- `/Users/ryan/Documents/Claude/Projects/A2Z Product Claude/core/memory.md` — decisions & rules đã confirm.

### Bước 3: Viết ticket

Sau khi có đủ thông tin, viết ticket theo format trong `ticket-template-lean.md`. Tuân theo tất cả rules trong file đó (AC format, language rules, validation behavior, dropdown SSoT, error handling format, AI node exceptions...).

**Checklist trước khi output:**
- [ ] Tuân theo Lean Ticket structure
- [ ] Error messages đúng theo `error-message-existed.md`
- [ ] Input/Output đúng conventions (`input-output-conventions.md`)
- [ ] Reference existing logic thay vì mô tả lại
- [ ] Story 1 dùng fixed template (chỉ thay [Action Name])
- [ ] Story cuối dùng Error Handling fixed template
- [ ] Có Out of Scope, Business Rules, Edge Cases
- [ ] AC testable, observable, dùng Given/When/Then
- [ ] Language rules: heading EN, AC body VN-EN mix, error messages EN only

### Bước 4: Lưu file `final-ticket.md`

Lưu ticket vào:
```
/Users/ryan/Documents/Claude/Projects/A2Z Product Claude/specs/[tên-node]/final-ticket.md
```

Quy tắc đặt tên thư mục: lowercase, dùng dấu gạch ngang (`-`) thay khoảng trắng.

### Bước 5: Hỏi user có muốn review không

Sau khi đã lưu file, **bắt buộc hỏi user**:

> "Ticket đã được lưu. Bạn có muốn tôi review và chỉnh lại không?"

Nếu user đồng ý → chạy self-review theo checklist dưới đây, cập nhật file. Nếu user từ chối → dừng.

---

## Self-Review Checklist (chỉ dùng khi user đồng ý ở Bước 5)

Review theo thứ tự. Sửa trực tiếp vào ticket — không liệt kê vấn đề ra chat.

**Structure & Completeness**
- [ ] Có đủ sections bắt buộc: Epic Summary, User Stories, Out of Scope, Inputs, Outputs, Business Rules, Error Handling
- [ ] Story 1 dùng đúng fixed template
- [ ] Story cuối là Error Handling với fixed template
- [ ] Không có section thừa: Implementation Notes, Technical Notes, Success Criteria Summary

**AC Quality**
- [ ] Mỗi AC mô tả đúng một observable behavior
- [ ] AC dùng Given/When/Then, mỗi keyword trên dòng riêng
- [ ] AC ID sequential (AC.01, AC.02, ...) và có one-line title English bold
- [ ] Không có AC nào vague: "works correctly", "handles properly", "user-friendly"
- [ ] Dropdown fields có embed table với UI Label + API value

**Validation & Error — tuân theo rules trong `ticket-template-lean.md`**
- [ ] Error Handling table chỉ có 3 cột
- [ ] Không có mục "Validation Rules" riêng
- [ ] Error messages đúng theo catalog
- [ ] Required field validation dùng từ "ngay" (real-time)
- [ ] Dropdown có default value → KHÔNG có AC validation "{Field} is required."
- [ ] Config-time / Runtime message consistency

**Lean Rules**
- [ ] Dùng standard reference sentences cho logic đã có
- [ ] Edge cases chỉ giữ lại khi có business impact
- [ ] Không có nội dung lặp giữa sections
- [ ] Dropdown options chỉ mô tả 1 lần trong AC (single source of truth)

**Input/Output**
- [ ] Mỗi input field có: Type, Required, Default, Manual Input, Expression support
- [ ] Output có đủ required fields: success, action, executed_at
- [ ] Nullable fields đánh dấu rõ, field names dùng snake_case

**Business Rules**
- [ ] Đánh số thứ tự (1, 2, 3...) — không dùng mã BR-01
- [ ] Mỗi rule ngắn gọn, business-first, không lặp AC

Sau khi review xong, cập nhật file và thông báo ngắn gọn những gì đã sửa.

---

## Tham chiếu thêm

Khi cần ví dụ ticket chuẩn để đối chiếu chất lượng, tham khảo `/Users/ryan/Documents/Claude/Projects/A2Z Product Claude/references/sample-good-tickets.md`.