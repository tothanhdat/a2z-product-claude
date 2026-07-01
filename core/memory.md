---
inclusion: always
---

# Memory

Ghi chú quan trọng, decisions đã confirm, và các rule bổ sung cần nhớ xuyên suốt workspace.

---

## Decisions & Rules

<!-- Thêm các quyết định đã confirm ở đây -->
- Storage cloud của hệ thống là S3.
- Không define lại retry count, backoff, status code nếu không có custom behavior mà sử dụng lại logic retry của hệ thống.
- **Đối với các node AI** (OpenAI, Google Gemini, Anthropic Claude, AI Generic): các ý liên quan đến retry cứ để là "Không retry. Chỉ dùng cơ chế retry node của hệ thống" — không cần mô tả chi tiết retry mechanism.
- Action * displays "{action name}" as pre-selected value and user can be changed other action.
- **Validation behavior:** Validation error hiển thị ngay khi field invalid (real-time) và block Continue button. KHÔNG phải click Continue rồi mới báo error. Cụ thể: khi user nhập giá trị không hợp lệ hoặc field required đang trống sau khi touched, error hiển thị ngay và Continue bị disabled. User không cần click Continue để trigger validation.
- **AC Writing Rule for Real-time Validation:** Khi viết AC về required field validation, phải dùng từ "ngay" để làm rõ real-time behavior. Format chuẩn: "Then the 'Continue' button is disabled, And inline validation error hiển thị ngay dưới field: '[Error message]'." Không viết "And if the user clicks 'Continue', a validation error is shown" vì điều này gây nhầm lẫn rằng error chỉ hiển thị khi click Continue.
- Các node AI có phần configure Session ID thì chỉ cần mô tả như thế này "Follow theo standard Ticket MART-1885: Session ID Field Spec - (AI Nodes)"
- **Dropdown with Default Value — No Required Validation Error:** Nếu field là dropdown và đã có mô tả default value rồi thì ở config-time validation KHÔNG cần báo error "{Field} is required." vì case này không bao giờ xảy ra (field luôn có giá trị default nên không thể trống).
- **Dropdown Options — Single Source of Truth:** Khi field là dropdown list, danh sách options chỉ mô tả MỘT LẦN duy nhất trong User Story (AC có embed table). KHÔNG liệt kê lại options ở Business Rules hoặc bất kỳ section nào khác. Business Rules chỉ nói constraint (VD: "Model là required, không có default") mà không list lại từng option. Lý do: tránh phải sửa nhiều chỗ khi options thay đổi.
- **Trigger fields không support Expression (kéo variable):** Các field trong Trigger node KHÔNG support kéo mapped variable / expression vì không có step nào phía trước. Tất cả field của Trigger chỉ nhận Static Value.
- **Không có Inputs section trong ticket:** Ticket KHÔNG có section "Inputs" riêng. Thông tin input fields (type, required, default, validation) được mô tả trực tiếp trong User Stories/ACs. Chỉ giữ Outputs section. Lý do: Inputs table lặp lại thông tin đã có trong ACs.
- **Dropdown Options — Single Source of Truth (cập nhật):** Không còn Inputs section nên chỉ cần đảm bảo options không bị lặp ở Business Rules hoặc section khác. AC là single source of truth.
- **Thứ tự sections chuẩn trong ticket:** Epic Summary → User Stories → Out of Scope → Outputs → Business Rules → Error Handling → Tooltips (chỉ khi được khai báo).
- **Placeholder không mô tả trong ticket:** Không viết placeholder text cho bất kỳ field nào trong ticket. Placeholder hiển thị trên Figma design — dev follow từ Figma. Ticket không cần nhắc.
- **Tooltips section:** Chỉ thêm section Tooltips vào ticket khi user khai báo rõ từ đầu field nào cần tooltip. Không tự thêm nếu user không đề cập. Format: bảng 2 cột Field | Tooltip Text, đặt sau Error Handling.
- **Max length error — thống nhất message config-time và runtime:** Khi field có giới hạn ký tự (max length), dùng cùng một error message cho cả config-time (static input) và runtime (expression resolve). Không dùng message riêng kiểu "too long, please shorten the mapped value" cho runtime. Gộp thành 1 dòng trong Error Handling table với When Detected = "Config-time / Runtime". VD: `"Message must be 4096 characters or less."` dùng cho cả hai case.
- **Resource not found — gộp các case cùng bản chất:** Khi resource (space, file, folder…) không tồn tại tại runtime — dù do bị xoá trước, bị xoá sau config, hay mất quyền access — gộp thành 1 dòng duy nhất trong Error Handling table với chung 1 message. Không tách thành 2 dòng "resource not found" và "selected resource no longer available". VD: `"Space not found. It may have been deleted or moved."` cover cả case xoá trước và xoá sau config.
---

## Notes
Không viết ticket quá dài nếu user không yêu cầu full technical specification.

<!-- Thêm ghi chú quan trọng ở đây -->
