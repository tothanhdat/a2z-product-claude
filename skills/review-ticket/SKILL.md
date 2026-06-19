---
name: review-ticket
description: Review ticket đã viết theo 5 nhóm phân tích. Phát hiện issue, đề xuất fix cụ thể, và chỉ chỉnh sửa khi được user đồng ý.
---

# Skill: Review Ticket

## Khi nào kích hoạt

Khi user yêu cầu review, đánh giá, kiểm tra, hoặc rà soát lại một ticket đã viết.

**Từ khoá:** "hãy review", "review ticket", "đánh giá ticket", "kiểm tra ticket", "ticket có ổn không", "ticket này ok chưa".

---

## Nguyên tắc

- **Không tự ý chỉnh ticket** — chỉ output danh sách issues, chờ user đồng ý trước khi sửa.
- **Mỗi issue bắt buộc kèm đề xuất cụ thể** — nói rõ sửa thành gì, bỏ phần nào, hoặc thêm gì.
- **Chỉ flag issue có business impact** — không flagging lỗi format nhỏ nhặt không ảnh hưởng test/deliver.
- **Ưu tiên happy case có giá trị** — edge case chỉ giữ khi ảnh hưởng thực sự đến UX, data, security, hoặc scope.

---

## Quy trình

**Bước 1:** Xác định ticket. Nếu user chưa chỉ rõ → hỏi tên node hoặc đường dẫn file.

**Bước 2:** Đọc ticket tại `specs/[tên-node]/final-ticket.md`. Đọc thêm `core/memory.md` và `references/ticket-template-lean.md` để làm baseline so sánh.

**Bước 3:** Phân tích theo 5 nhóm bên dưới. Ghi nhận issue thực sự — không nhồi cho đủ mục.

**Bước 4:** Output kết quả. Nhóm nào không có issue → ghi ✅.

**Bước 5:** Hỏi user trước khi sửa:
> "Mình tìm thấy [N] issue. Bạn muốn mình **chỉnh tất cả**, **chỉnh một số** (báo số thứ tự), hay **thảo luận** trước?"

---

## 5 Nhóm Phân Tích

### Nhóm 1: Structure & Completeness
Kiểm tra ticket có đủ sections bắt buộc (Epic Summary, User Stories, Out of Scope, Inputs, Outputs, Business Rules, Error Handling), Story 1 và Story cuối dùng đúng fixed template, không có section thừa (Implementation Notes, In Scope, Success Criteria Summary).

### Nhóm 2: Technical Overreach
Phát hiện ý quá technical — thuộc Engineering quyết định, không phải BA spec. Flag khi thấy: mô tả API call sequence hay HTTP status code ngoài Context block, chỉ định implementation approach (webhook, Redis, background job), mô tả retry/backoff mechanism, database schema, performance/caching detail.

Không flag: API endpoint trong Context block (BA convention của project), error message text cụ thể, field type trong Inputs.

### Nhóm 3: Testability
Phát hiện ý mơ hồ mà QC không đủ dữ liệu để test. Flag khi thấy: AC dùng ngôn ngữ vague ("works correctly", "handles properly", "appropriate error"), Then không mô tả observable outcome, error message không quote chính xác, dropdown AC thiếu embed table UI Label + API Value, required field AC thiếu từ "ngay".

### Nhóm 4: Scope Realism
Phát hiện edge case không thực tế (đã có platform-level handling, không xảy ra với UI hiện tại, chỉ relevant cho dev testing) và happy case quan trọng bị thiếu (behavior chính của node chưa có AC, conditional field chưa được test, default behavior chưa cover).

### Nhóm 5: Standards Compliance
Đối chiếu với rules đã confirm trong `core/memory.md`:
- Required field validation dùng từ "ngay", không viết "if user clicks Continue"
- Dropdown có default value → không có AC validation "is required"
- Config-time / Runtime message phải giống nhau với cùng scenario
- Dropdown options chỉ mô tả 1 lần trong AC, không lặp ở Inputs hay Business Rules
- Error Handling table đúng 3 cột
- AI node: không retry, Session ID follow MART-1885

---

## Output Format

```
## Review: [Tên ticket]

### Nhóm 1: Structure & Completeness
✅ hoặc danh sách issue

### Nhóm 2: Technical Overreach
✅ hoặc danh sách issue

### Nhóm 3: Testability
✅ hoặc danh sách issue

### Nhóm 4: Scope Realism
✅ hoặc danh sách issue

### Nhóm 5: Standards Compliance
✅ hoặc danh sách issue

---
Tổng: [N] issue.
Mình chỉnh tất cả, chỉnh một số (báo số), hay thảo luận trước?
```

**Format mỗi issue:**
```
[số]. [Vị trí — Story N / AC.NN / Section]
Vấn đề: [mô tả ngắn]
Đề xuất: [sửa thành gì / bỏ phần nào / thêm gì]
```

---

## Khi apply fix

Chỉ sửa những issue được user approve. Sửa trực tiếp vào `final-ticket.md`, không tạo file mới. Sau khi sửa, báo ngắn gọn những gì đã thay đổi.
