---
name: write-concept-demo
description: "Viết concept demo workflow kết hợp các node/trigger đã có trong sprint. Output dùng cho QC demo và PO review."
inclusion: manual
---

# Skill: Write Concept Demo

## Khi nào kích hoạt

Khi user muốn soạn concept demo cho một sprint — mô tả các workflow thực tế, có thể sử dụng được trong công việc, kết hợp các node/trigger đã phát triển trong sprint đó.

**Từ khoá nhận diện:** "viết concept demo", "concept demo", "demo workflow", "viết concept cho sprint", "demo cho PO", "workflow demo", "concept sprint".

---

## Skill này LÀ gì và KHÔNG LÀ gì

| Đây là | Đây KHÔNG phải |
|---|---|
| Soạn **concept workflow thực tế** từ các node/trigger trong sprint | Viết technical spec hoặc ticket |
| Mô tả **vấn đề kinh doanh** và giải pháp automation tương ứng | Định nghĩa business rules hay AC |
| Output để **QC tạo workflow demo** và **PO đánh giá giá trị** | Tài liệu phát triển nội bộ |
| Ngôn ngữ **tiếng Việt, gần gũi với business user** | Tài liệu kỹ thuật cho Engineering |

---

## Quy trình bắt buộc

### Bước 1: Thu thập thông tin sprint

Trước khi viết, **hỏi user tối đa 4 câu** để nắm đủ nguyên liệu. Không hỏi nhiều hơn. Không hỏi những gì đã rõ.

**Bắt buộc hỏi:**

```
1. Các node/trigger nào đã hoàn thành trong sprint này?
   (Có thể liệt kê tên, ví dụ: Gmail - Send Email, Web Form Trigger, Imagen 4 - Generate Image...)
   **Đề xuất:** Liệt kê tất cả, kể cả node nhỏ — đôi khi chính node phụ tạo nên concept thú vị.

2. Target audience của demo này là ai (ngành, phòng ban, level)?
   **Đề xuất:** Business user phổ thông (Marketing, Ops, Admin) — không cần technical background.
   *(Lý do: tone viết và ví dụ problem sẽ khác nhau tuỳ đối tượng.)*

3. Có khoảng bao nhiêu concept workflow muốn viết?
   **Đề xuất:** 2–3 concept — đủ để show coverage của sprint mà không quá dàn trải.
```

**Câu hỏi tuỳ chọn (hỏi nếu cần thêm ngữ cảnh):**

```
4. Có usecase hoặc ngành nghề cụ thể nào muốn demo ưu tiên không?
   **Đề xuất:** Không giới hạn — chọn usecase phổ biến nhất để PO dễ hình dung ROI.
```

Nếu user đã cung cấp đủ thông tin (danh sách node, số lượng concept), bỏ qua câu hỏi đã có đáp án và chỉ hỏi những gì còn thiếu.

---

### Bước 2: Phân tích và lên ý tưởng concept

Dựa trên danh sách node/trigger, **tư duy theo hướng combination** — tìm cách kết hợp nhiều node thành một workflow giải quyết vấn đề thực tế.

**Nguyên tắc chọn concept tốt:**

- **Thực tế:** Vấn đề trong Problem phải là điều business user thực sự gặp hàng ngày.
- **Luôn có AI:** Mỗi concept **bắt buộc** có ít nhất một AI node (Ask AI, Summarize, Conversation, Generate Image, v.v.) — đây là yêu cầu không được bỏ qua.
- **Có trigger rõ ràng:** Mỗi workflow phải bắt đầu bằng một sự kiện cụ thể (form submit, email đến, schedule...).
- **Kết hợp ≥ 3 node:** Concept đơn lẻ 1–2 node không đủ sức thuyết phục PO.
- **Phủ coverage rộng:** Nếu có 3 concept, cố gắng cover 3 nhóm node khác nhau — đừng dồn hết vào email.

**Khi danh sách sprint chỉ toàn trigger (không có action):**

Tham chiếu `references/existing-nodes.md` để chọn các action đã phát triển phù hợp, kết hợp với trigger của sprint thành workflow hoàn chỉnh. Ưu tiên action đã có status **Done** hoặc **In progress**. Không cần hỏi lại user — đây là bước AI tự suy luận và chọn combination tốt nhất dựa trên usecase thực tế.

**Nếu không đủ node để tạo workflow phức tạp:** Vẫn viết concept, nhưng ghi chú rõ trong Solution những node nào là "placeholder" (cần node khác hỗ trợ) và những node nào là core từ sprint.

---

### Bước 3: Viết concept demo

Viết theo **Output Template** bên dưới. Mỗi concept là một workflow độc lập.

**Nguyên tắc viết:**

- **Problem:** Mô tả tình huống thực tế của người dùng, nỗi đau cụ thể, hậu quả nếu không có automation. Không dùng buzzword chung chung.
- **Solution:** Mô tả từng bước workflow theo thứ tự thực thi. Ghi rõ tên node/trigger dùng trong ngoặc đơn sau mỗi bước. Dùng dấu `*` cho danh sách.
- **Value:** Ít nhất 3 điểm giá trị cụ thể — tiết kiệm bao nhiêu thời gian/effort, trải nghiệm người dùng tốt hơn thế nào, rủi ro nào được loại bỏ.

**Tone viết:**
- Ngôn ngữ **tiếng Việt**, tự nhiên, gần gũi với business.
- Tránh từ kỹ thuật (API, webhook, payload, endpoint). Dùng từ bình thường: "hệ thống tự nhận", "tự gửi", "phân tích nội dung".
- Problem viết như đang kể chuyện thực tế của team, không phải mô tả chức năng phần mềm.

---

### Bước 4: Lưu file

Lưu concept demo vào:

```
specs/concept-demo-[sprint-name-hoặc-số]/concept-demo.md
```

Ví dụ: `specs/concept-demo-sprint-42/concept-demo.md`

Nếu user không cung cấp tên sprint, dùng ngày hiện tại: `concept-demo-[YYYY-MM-DD]`.

---

### Bước 5: Hỏi sau khi hoàn thành

Sau khi lưu file, hỏi:

> "Concept demo đã được lưu. Bạn muốn mình **thêm concept** nào nữa không, hoặc **chỉnh sửa** concept nào vừa viết?"

---

## Output Template

```markdown
# Concept Demo — [Tên Sprint hoặc Ngày]

> **Mục đích:** Tài liệu này mô tả các workflow demo được xây dựng từ các node/trigger hoàn thành trong sprint. Dùng cho QC tạo workflow thực tế và PO đánh giá giá trị sản phẩm.

---

## [Số thứ tự]. Workflow [Tên] – [Tên phụ tiếng Việt]

**Problem:**
[2–4 câu mô tả tình huống thực tế. Vấn đề đang xảy ra là gì? Team nào đang bị ảnh hưởng? Hậu quả của việc làm thủ công là gì?]

**Solution:**
Xây dựng workflow tự động: khi [sự kiện trigger]:

* [Bước 1 — mô tả hành động + tên node trong ngoặc đơn, ví dụ: (Web Form Trigger)]
* [Bước 2 — mô tả hành động + tên node]
* [Bước 3 — ...]
* [Bước N — ...]

**Value:**

* [Giá trị 1 — cụ thể, đo được nếu có thể]
* [Giá trị 2 — trải nghiệm người dùng]
* [Giá trị 3 — rủi ro/lỗi được loại bỏ]
* [Giá trị 4 — tuỳ chọn nếu có thêm]

---

## [Số thứ tự]. Workflow [Tên] – [Tên phụ tiếng Việt]

[Lặp lại cấu trúc trên]

---

*Được tạo bởi AI Assistant — [Ngày tạo]*
```

---

## Ví dụ interaction

**User:** Viết concept demo cho sprint 15. Các node đã xong: Web Form Trigger, Claude - Analyze Text, Gemini Chatbot, ChatGPT - Draft Email, Gmail - Send Email với Approval, Imagen 4 - Generate Image. Cần 2 concept, demo cho Marketing.

**AI (Bước 1 — bỏ qua câu hỏi đã có đáp án, chỉ xác nhận):**

> Mình sẽ viết 2 concept cho Marketing, dùng tất cả node trong sprint 15. Một câu xác nhận nhanh:
>
> Approval trong "Gmail - Send Email với Approval" là Manager click Approve/Reject ngay trong email, hay cần login vào hệ thống?
> **Đề xuất:** Click Approve/Reject trực tiếp trong email — đơn giản hơn cho Manager.

**(Sau khi user confirm → AI viết 2 concept, ví dụ: Webinar Registration và Content Approval, lưu file, hỏi có cần chỉnh sửa không.)**

---

## Checklist trước khi xuất bản concept

- [ ] Mỗi concept có ít nhất 1 trigger rõ ràng
- [ ] Mỗi bước Solution ghi tên node trong ngoặc đơn
- [ ] Problem viết từ góc nhìn người dùng thực tế, không phải mô tả phần mềm
- [ ] Value có ít nhất 3 điểm, ít nhất 1 điểm cụ thể (thời gian, số lượng, tỉ lệ)
- [ ] Không dùng từ kỹ thuật trong Problem và Value
- [ ] Ngôn ngữ toàn bộ là tiếng Việt
- [ ] Tên node trong ngoặc đơn viết đúng tên thực tế của node trên platform
