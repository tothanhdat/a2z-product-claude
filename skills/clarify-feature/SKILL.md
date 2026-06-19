---
name: Clarify & Analyze Feature
description: Phân tích và làm rõ một feature về mặt business TRƯỚC khi viết ticket. Tư vấn, brainstorm, surface scope/business rules/decisions. KHÔNG viết ticket.
inclusion: manual
---

# Skill: Clarify & Analyze Feature

## Khi nào kích hoạt

Khi user muốn **hiểu rõ và định hình một feature về mặt business** trước khi viết ticket — không phải yêu cầu output ticket ngay.

**Từ khoá nhận diện:** "hãy làm rõ", "làm rõ feature", "hãy brainstorm", "brainstorm", "phân tích feature", "phân tích giúp", "tư vấn", "tư vấn business", "feature này nên làm gì", "giúp tôi nghĩ về", "đánh giá ý tưởng", "có nên làm", "scope của feature này".

---

## Skill này LÀ gì và KHÔNG LÀ gì

| Đây là | Đây KHÔNG phải |
|---|---|
| Bước **suy nghĩ / tư vấn** trước khi viết ticket | Bước viết ticket (đó là `write-ticket`) |
| Làm rõ intent, scope, business rules, decisions | Output User Stories / AC hoàn chỉnh |
| Brainstorm options và tradeoffs | Quyết định thay user — chỉ đề xuất |
| Surface những gì cần quyết định | Đi sâu vào implementation/technical design |

**Nguyên tắc tối quan trọng:** Skill này **không tạo ticket**. Output là một bản **Feature Analysis** giúp user (và sau đó là skill `write-ticket`) có đủ thông tin business để viết ticket tốt.

---

## Phân biệt với các skill khác

- **`clarify-feature` (skill này):** Làm rõ / brainstorm feature ở giai đoạn **trước ticket**. Khi scope còn mơ hồ, cần tư vấn business, cần cân nhắc options.
- **`write-ticket`:** Khi đã rõ scope và sẵn sàng output ticket cho **node/action/trigger mới**.
- **`write-user-story`:** Khi viết delta ticket cho node **đã tồn tại**.
- **`review-ticket`:** Khi review ticket **đã viết xong**.

> Nếu trong lúc làm rõ, user nói "ok giờ viết ticket đi" → kết thúc skill này và route sang `write-ticket` (hoặc `write-user-story` nếu là delta), dùng Feature Analysis làm input.

---

## Hai chế độ (cùng một backbone)

| Chế độ | Trigger điển hình | Trọng tâm |
|---|---|---|
| **Clarify Mode** | "hãy làm rõ", "phân tích", "scope thế nào" | Gỡ mơ hồ, định scope, chốt business rules, surface open questions |
| **Brainstorm Mode** | "hãy brainstorm", "có những hướng nào", "nên làm thế nào" | Sinh nhiều options/approach, so sánh tradeoffs, recommend hướng đi |

Khi không rõ user muốn mode nào, mặc định chạy **Clarify Mode** và lồng brainstorm khi gặp điểm có nhiều hướng giải quyết hợp lý.

---

## Quy trình bắt buộc

### Bước 1: Khôi phục & xác nhận cách hiểu

Trước khi phân tích, **diễn đạt lại feature theo cách hiểu của mình** (2–4 câu): feature này là gì, dành cho ai, giải quyết vấn đề gì. Mục đích là để user xác nhận hoặc chỉnh ngay từ đầu, tránh phân tích sai hướng.

Format:

```
**Cách mình hiểu feature này:**
[2–4 câu: WHAT — feature làm gì; WHO — cho user nào; WHY — giải quyết vấn đề/value gì.]

Mình hiểu đúng chưa, hay cần chỉnh chỗ nào?
```

Nếu user mô tả còn quá sơ sài để diễn đạt lại → hỏi 1 câu mở để nắm intent trước, rồi mới tiếp tục.

### Bước 2: Tham chiếu existing nodes & product context

Đọc `references/existing-nodes.md` để:
- Tìm node/feature tương tự đã có → reuse pattern, reference thay vì sáng tạo mới.
- Phát hiện logic có thể tái sử dụng (selector, picker, auth, retry...).
- Tránh đề xuất trùng với cái đã Done.

Luôn giữ `core/product-context.md` trong đầu: end-user-first, simple surface / powerful core, business process clarity, safe by default.

### Bước 3: Phân tích đa chiều

Đi qua **Analysis Lens** (checklist bên dưới). Chỉ report những chiều thực sự có nội dung — không nhồi cho đủ mục. Mỗi phát hiện phải concrete, business-first, testable được về sau.

### Bước 4: Brainstorm options (khi có nhiều hướng hợp lý)

Khi một quyết định có ≥ 2 hướng giải quyết hợp lý, trình bày dưới dạng options + tradeoffs + recommendation. Không bỏ qua bước này ở Brainstorm Mode.

Format options:

```
**Quyết định: [Tên điểm cần quyết]**

| Option | Mô tả | Ưu | Nhược | Phù hợp khi |
|---|---|---|---|---|
| A | ... | ... | ... | ... |
| B | ... | ... | ... | ... |

**Đề xuất:** Option [X] — [1 câu lý do, tham chiếu node tương tự / product context.]
```

### Bước 5: Câu hỏi trọng tâm + đề xuất

Liệt kê những gì **cần user quyết** trước khi viết ticket. **Mỗi câu hỏi bắt buộc kèm suggested answer** (giống `write-ticket`) để user confirm nhanh.

Số lượng câu hỏi linh hoạt theo độ mơ hồ — nhưng ưu tiên gom vào những quyết định **thực sự ảnh hưởng scope, business rules, UX, hoặc output**. Không hỏi về platform behaviors đã chuẩn hoá (auth, retry, connect step, validation real-time).

Format:

```
N. [Câu hỏi cụ thể]
   **Đề xuất:** [Câu trả lời gợi ý concrete, actionable]
   *(Lý do: [1 câu — tham chiếu node tương tự, convention, hoặc product context])*
```

### Bước 6: Output Feature Analysis

Tổng hợp theo **Output Template** bên dưới. Đây là sản phẩm chính của skill.

### Bước 7: Lưu file & handoff

Sau khi output, hỏi user đúng 2 việc:

> "Bạn muốn mình **lưu lại bản phân tích này** để dùng khi viết ticket không? Và sau khi bạn chốt các quyết định ở trên, mình **viết ticket luôn** chứ?"

- **Nếu user muốn lưu** → lưu vào `specs/[tên-feature]/feature-analysis.md` (lowercase, dấu gạch ngang). Đây là file cầu nối — khi chạy `write-ticket` cho feature này, đọc file này làm input.
- **Nếu user muốn viết ticket** → route sang `write-ticket` (hoặc `write-user-story` nếu là delta trên node đã có), dùng các quyết định đã chốt làm input, **không hỏi lại** những gì đã làm rõ ở đây.
- **Nếu user chỉ muốn thảo luận tiếp** → tiếp tục iterate, không ép lưu/viết.

---

## Analysis Lens (checklist phân tích)

Dùng làm khung phân tích ở Bước 3. Bỏ qua chiều nào không có nội dung thực chất.

| Chiều | Câu hỏi tự đặt | Mục tiêu |
|---|---|---|
| **Intent & Value** | Feature giải quyết vấn đề gì? Không có nó user đau ở đâu? | Chốt lý do tồn tại |
| **Personas & Use Cases** | Ai dùng? Trong workflow nào? 1–3 use case cụ thể? | Đảm bảo đúng đối tượng end-user |
| **Scope Boundaries** | Cái gì chắc chắn IN? Cái gì nên OUT (defer/ticket khác)? | Ngăn scope creep từ sớm |
| **Reuse & Reference** | Node nào tương tự đã có? Logic nào reuse được? | Tránh sáng tạo lại, giữ consistency |
| **Inputs (high-level)** | Cần user cung cấp gì? Field nào required/optional/conditional? | Phác thảo input, chưa cần full spec |
| **Outputs (high-level)** | Downstream steps cần data gì từ feature này? | Đảm bảo output hữu ích |
| **Business Rules & Constraints** | Có rule/constraint business nào? Default behavior? Permission? | Surface rule trước khi viết AC |
| **Edge Cases & Risks** | Trường hợp lạ nào ảnh hưởng UX/data/security? | Chỉ giữ edge case có impact thật |
| **Dependencies** | Phụ thuộc node/feature/provider nào? Provider có hỗ trợ không? | Phát hiện blocker sớm |
| **Decisions Needed** | Điểm nào còn mơ hồ, cần user/stakeholder quyết? | Tách "đã rõ" khỏi "cần quyết" |

---

## Output Template

```markdown
# Feature Analysis: [Tên Feature]

## 1. Cách hiểu feature
[2–4 câu: WHAT / WHO / WHY — bản đã được user confirm.]

## 2. Personas & Use Cases
- **Persona:** [Ai dùng]
- **Use case 1:** [Tình huống cụ thể]
- **Use case 2:** [Tình huống cụ thể]

## 3. Scope đề xuất

**Khả năng cốt lõi (proposed In):**
- [Capability 1]
- [Capability 2]

**Đề xuất Out of Scope:**
- [Cái gì nên loại / defer — kèm lý do hoặc ticket sẽ cover]

## 4. Reuse & Reference
- [Node tương tự đã có + logic reuse được — từ existing-nodes.md]

## 5. Business Rules & Constraints (draft)
1. [Rule business ngắn gọn — happy path]
2. ...

## 6. Inputs (high-level)
| Field (tạm) | Bắt buộc? | Default | Ghi chú |
|---|---|---|---|
| ... | ... | ... | ... |

## 7. Outputs (high-level)
- [Data downstream cần]

## 8. Edge Cases & Risks
- [Edge case / rủi ro có impact thật]

## 9. Options & Tradeoffs (nếu có điểm nhiều hướng)
[Bảng options + recommendation cho từng quyết định lớn]

## 10. Quyết định cần chốt (Open Questions + Đề xuất)
1. [Câu hỏi]
   **Đề xuất:** [Gợi ý]
   *(Lý do: ...)*
2. ...

## 11. Hướng ticket đề xuất
- **Loại:** Action / Trigger / Feature mới — hay delta trên node đã có?
- **Story outline (tạm):** [Liệt kê nhanh các story chính sẽ có khi viết ticket]
- **Template phù hợp:** Lean Ticket (write-ticket) / Delta Ticket (write-user-story)
```

> **Brainstorm Mode:** ưu tiên phần **9. Options & Tradeoffs** lên đầu, làm trọng tâm. Các phần khác giữ ngắn gọn hơn.

---

## Nguyên tắc

- **Business-first, không technical.** Không bàn API call sequence, schema, retry mechanism. Nói về behavior, value, scope, rules.
- **Tư vấn, không áp đặt.** Luôn đưa option kèm recommendation và lý do; user là người quyết.
- **Tách "đã rõ" khỏi "cần quyết".** Cái gì có thể assume an toàn thì ghi vào Assumptions/Business Rules; cái gì ảnh hưởng lớn thì đẩy thành Open Question có đề xuất.
- **Reuse trước khi sáng tạo.** Luôn soi existing-nodes.md để reference pattern đã có.
- **Không viết ticket.** Không output User Stories/AC hoàn chỉnh. Nếu user muốn ticket → handoff sang `write-ticket`/`write-user-story`.
- **Không hỏi lại cái đã chuẩn hoá.** Auth, connect step, retry, validation real-time — assume theo standards.
- **Adaptive depth.** Feature nhỏ → analysis ngắn, vài quyết định. Feature lớn → đầy đủ lens + options.
- **Không nhồi dồn.** Diễn đạt lại cách hiểu → phân tích → hỏi gọn các quyết định; iterate theo phản hồi user.

---

## Ví dụ interaction — Clarify Mode

**User:** Hãy làm rõ giúp mình feature "Action - Google Drive - Rename File" trước khi viết ticket.

**AI:**

**Cách mình hiểu feature này:**
Action cho phép workflow builder đổi tên một file đã có trên Google Drive, dùng trong các flow như chuẩn hoá tên file sau khi upload, gắn timestamp, hoặc đặt tên theo data từ upstream step. User xác định file cần đổi tên rồi nhập tên mới.

Mình hiểu đúng chưa, hay cần chỉnh chỗ nào?

**(sau khi user confirm, AI output Feature Analysis: scope đề xuất, reuse pattern từ Delete File, business rules draft về extension, edge cases trùng tên, và các Open Questions kèm đề xuất — chưa viết AC.)**

Cuối cùng:
> "Bạn muốn mình lưu lại bản phân tích này để dùng khi viết ticket không? Và sau khi bạn chốt các quyết định ở trên, mình viết ticket luôn chứ?"

---

## Ví dụ interaction — Brainstorm Mode

**User:** Brainstorm giúp mình các hướng cho trigger "New Event" của Google Calendar — instant hay polling?

**AI:** (trình bày Options & Tradeoffs lên đầu)

**Quyết định: Cơ chế detect event mới**

| Option | Mô tả | Ưu | Nhược | Phù hợp khi |
|---|---|---|---|---|
| A. Instant (Push Notification) | Dùng Google Calendar webhook | Real-time (vài giây), tiết kiệm API quota | Cần webhook infra, quản lý lifecycle | Use case nhạy thời gian |
| B. Polling | Quét định kỳ | Đơn giản, reuse polling sẵn có | Có độ trễ, tốn API call | Không cần real-time |

**Đề xuất:** Option A — vì các trigger "New" khác (Sheets New Row) đã đi hướng instant, và use case notification cần real-time. (tham chiếu existing-nodes.md: New Event đang ở Planning với "Instant webhook, push notification mechanism".)

**(tiếp tục với các quyết định khác: scope calendar, recurring events... rồi tổng hợp Feature Analysis.)**
