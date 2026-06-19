# User Story - Human Input Web Form - Response Mode Configuration

## Epic Summary

**Epic Name:** Human Input – Web Form (Response Mode)

**Goal:**
Bổ sung tính năng Response Mode cho Web Form Trigger, cho phép builder kiểm soát trải nghiệm người dùng sau khi submit form trên Publish URL.

**Context:**
Tiếp nối ticket MART-1866. Web Form Trigger đã có nền tảng từ Sprint 12. Ticket này bổ sung tab Advanced với field Response Mode — kiểm soát thời điểm form page phản hồi respondent. Response Mode chỉ áp dụng cho Publish URL; Draft URL luôn hiển thị fixed message. Form page cần polling hoặc websocket để nhận kết quả workflow execution cho mode "Workflow Finishes".

**Expected UI Fields:**
- Tab Advanced: Response Mode (Dropdown)

---

## User Stories

### Story 1: Response Mode configuration for Publish URL

**Story:** As an automation workflow builder, I want to configure when the form respondent receives a response on Publish URL, so that I can control the user experience after form submission.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Response Mode — dropdown options**
Given user đang ở tab Advanced,
When user mở dropdown Response Mode,
Then hệ thống hiển thị các options:

| UI Label | API Value | Description |
|---|---|---|
| Form Is Submitted | form_submitted | Phản hồi ngay khi nhận submission |
| Workflow Finishes | workflow_finishes | Phản hồi sau khi node cuối cùng chạy xong |

**AC.02. Response Mode default is "Form Is Submitted"**
Given user chưa thay đổi config,
Then hệ thống mặc định chọn "Form Is Submitted"

**AC.03. Response Mode only applies to Publish URL**
Given Response Mode được cấu hình,
When form được submit trên Draft URL,
Then Response Mode setting không áp dụng,
And form page luôn hiển thị fixed message: "Thank you! Your response has been recorded."

**AC.04. "Form Is Submitted" — immediate response to respondent**
Given Response Mode = "Form Is Submitted",
When người điền form submit thành công trên Publish URL,
Then form page hiển thị ngay lập tức: "Thank you! Your response has been recorded.",
And không đợi workflow execution hoàn tất

**AC.05. "Workflow Finishes" — immediate progress feedback after submit**
Given Response Mode = "Workflow Finishes",
When người điền form submit thành công trên Publish URL,
Then form page hiển thị message: "Your response has been submitted. Please wait while we process your request...",
And kèm animated progress indicator,
And form page đợi cho đến khi workflow hoàn tất hoặc timeout

**AC.06. "Workflow Finishes" — success result**
Given Response Mode = "Workflow Finishes",
And workflow execution hoàn tất thành công,
When form page nhận kết quả,
Then hiển thị message: "All done! Your request has been processed successfully."

**AC.07. "Workflow Finishes" — failure result**
Given Response Mode = "Workflow Finishes",
And workflow execution thất bại,
When form page nhận kết quả,
Then hiển thị message: "Something went wrong while processing your request. Please try again later or contact support."

**AC.08. "Workflow Finishes" — graceful timeout fallback**
Given Response Mode = "Workflow Finishes",
And workflow chưa hoàn tất sau 5 phút,
When timeout xảy ra,
Then form page hiển thị message: "Your response has been submitted successfully. Processing is taking longer than usual. You may close this page.",
And workflow vẫn tiếp tục chạy ngầm

---

## Out of Scope

- **Authentication configuration** — đã tách sang ticket riêng (sprint hiện tại).
- **Custom response messages** — builder không thể tùy chỉnh nội dung message; tất cả messages là fixed system strings.
- **Custom redirect URL sau khi submit** — defer sang phase sau.
- **Response Mode trên Draft URL** — Draft URL luôn dùng fixed message, không configurable.

---

## Business Rules

1. Response Mode chỉ áp dụng cho Publish URL — Draft URL luôn hiển thị fixed message bất kể config.
2. Timeout mặc định cho "Workflow Finishes" là 5 phút.
3. Khi timeout xảy ra, workflow vẫn tiếp tục chạy ngầm — không bị cancel.
4. Response Mode có default là "Form Is Submitted" — field luôn có giá trị, không thể trống.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Workflow execution failed (Workflow Finishes mode) | Runtime — sau khi workflow fail | "Something went wrong while processing your request. Please try again later or contact support." |
| Execution timeout sau 5 phút (Workflow Finishes mode) | Runtime — timeout | "Your response has been submitted successfully. Processing is taking longer than usual. You may close this page." |

---

## Assumptions

1. Form page có khả năng polling hoặc websocket để nhận kết quả workflow execution.
2. Platform có mechanism để track workflow execution status và trả về kết quả cho form page.
3. Draft URL và Publish URL đã được implement trong MART-1866.
4. Timeout 5 phút là reasonable cho majority of workflows — có thể điều chỉnh sau dựa trên data thực tế.
