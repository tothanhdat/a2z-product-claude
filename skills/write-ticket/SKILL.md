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
- `references/existing-nodes.md` — xác định node cùng provider đã có logic gì reusable, tránh duplicate.
- `references/ticket-template-lean.md` — **source of truth** cho ticket structure, AC format, language rules, validation behavior rules, và tất cả format conventions.
- `references/error-message-catalog.md` — error message catalog.
- `core/memory.md` — decisions & rules đã confirm.

### Bước 3: Viết ticket

Viết ticket theo đúng structure và rules trong `references/ticket-template-lean.md` — đây là source of truth cho mọi format convention (AC format, language rules, validation behavior, dropdown SSoT, error handling, AI node exceptions...).

Tham chiếu thêm `references/input-output-conventions.md` cho Inputs/Outputs section và `references/error-message-catalog.md` cho error messages.

### Bước 4: Lưu file `final-ticket.md`

Lưu ticket vào:
```
specs/[tên-node]/final-ticket.md
```

Quy tắc đặt tên thư mục: lowercase, dùng dấu gạch ngang (`-`) thay khoảng trắng.

### Bước 5: Hỏi user có muốn review không

Sau khi đã lưu file, **bắt buộc hỏi user**:

> "Ticket đã được lưu. Bạn có muốn tôi review và chỉnh lại không?"

Nếu user đồng ý → chạy `skills/review-ticket/SKILL.md`, sửa trực tiếp vào file. Nếu user từ chối → dừng.

---

## Tham chiếu

- `references/ticket-template-lean.md` — structure, AC format, language rules, fixed templates
- `references/existing-nodes.md` — node đã có, pattern reuse
- `references/input-output-conventions.md` — Inputs/Outputs section format
- `references/error-message-catalog.md` — error messages chuẩn
- `references/sample-good-tickets.md` — ví dụ ticket tốt để đối chiếu
- `core/memory.md` — decisions & rules đã confirm
