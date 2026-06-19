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
- **Config-time / Runtime Error Message Consistency:** Với các validation đơn giản (empty, length exceeded, invalid characters), runtime message PHẢI GIỐNG config-time message. Không được dùng message generic ("resolved to an empty value", "resolved to an invalid value", "is too long. Please shorten the mapped value") khi có thể trả message tường minh giống FE. Chỉ dùng message khác khi runtime detect lỗi mà config-time không thể validate (VD: URL trỏ tới resource không tồn tại, format phức tạp chỉ provider mới biết).
- **Dropdown Options — Single Source of Truth:** Khi field là dropdown list, danh sách options chỉ mô tả MỘT LẦN duy nhất trong User Story (AC có embed table). KHÔNG liệt kê lại options ở Inputs section, Business Rules, hoặc bất kỳ section nào khác. Inputs section chỉ ghi tên field + note ngắn (VD: "Options: xem AC.02 Story 2"). Business Rules chỉ nói constraint (VD: "Model là required, không có default") mà không list lại từng option. Lý do: tránh phải sửa nhiều chỗ khi options thay đổi.
---

## Notes
Không viết ticket quá dài nếu user không yêu cầu full technical specification.

<!-- Thêm ghi chú quan trọng ở đây -->
