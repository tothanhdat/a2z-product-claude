# Concept Demo — Google Calendar Triggers (2026-06-26)

> **Mục đích:** Tài liệu này mô tả các workflow demo được xây dựng từ các trigger Google Calendar hoàn thành trong sprint: Event Ends, New Calendar, New Event, Event Cancelled. Dùng cho QC tạo workflow thực tế và PO đánh giá giá trị sản phẩm.

---

## 1. Workflow Meeting Ended Notification – Tự động thông báo kết thúc cuộc họp đến người tham dự

**Trigger:** Google Calendar – Event Ends

**Problem:**
Sau khi cuộc họp kết thúc, người tổ chức thường không có thời gian gửi thông báo cho toàn bộ người tham dự — đặc biệt khi họp xong là chuyển ngay sang việc khác. Người tham dự từ xa hoặc tham dự muộn đôi khi không biết cuộc họp đã kết thúc và vẫn đang chờ, không biết liên hệ ai nếu có thắc mắc phát sinh sau đó.

**Solution:**
Xây dựng workflow tự động: ngay khi một sự kiện lịch kết thúc:

* Hệ thống phát hiện event vừa kết thúc (Google Calendar – Event Ends)
* Kiểm tra người tổ chức có phải là chính mình không — nếu không phải, workflow dừng lại để tránh gửi nhầm cho các meeting do người khác tổ chức (Condition – IF/ELSE)
* AI soạn nội dung email thông báo kết thúc, tự điều chỉnh tone phù hợp dựa trên tiêu đề và độ dài của event (ví dụ: họp ngắn 30 phút vs. workshop 3 tiếng sẽ có lời kết khác nhau) (Ask AI)
* Hệ thống gửi email đến người tổ chức để xem trước nội dung và chờ xác nhận — người tổ chức click Approve hoặc Reject ngay trong Gmail trước khi email được gửi đi (Human in the Loop – Wait for Confirmation)
* Nếu được Approve, hệ thống gửi email đến toàn bộ người tham dự với nội dung: cuộc họp đã kết thúc, nếu có thắc mắc vui lòng liên hệ người tổ chức (Gmail – Send Email)

**Value:**

* Người tham dự luôn biết cuộc họp đã chính thức kết thúc và có đầu mối liên hệ rõ ràng nếu có thắc mắc phát sinh sau đó
* Người tổ chức không cần nhớ gửi email thủ công sau mỗi cuộc họp
* IF/ELSE đảm bảo workflow chỉ chạy cho meeting do chính mình tổ chức — không gửi nhầm cho các event được mời tham dự
* AI cá nhân hóa nội dung email theo từng sự kiện, tránh cảm giác template cứng nhắc khi người nhận đọc

---

## 2. Workflow Meeting Preparation Brief – Chuẩn bị tài liệu tự động khi có lịch họp mới

**Trigger:** Google Calendar – New Event (Instant)

**Problem:**
Khi một cuộc họp mới được tạo trên lịch, người tham dự thường không được chuẩn bị gì — họ chỉ nhận được invite với tiêu đề và giờ họp. Đến ngày họp, nhiều người vào mà không biết mục tiêu cuộc họp là gì, cần chuẩn bị tài liệu gì, hay cần đặt câu hỏi gì. Cuộc họp bị kéo dài vì phải dành đầu giờ để giới thiệu context.

**Solution:**
Xây dựng workflow tự động: ngay khi một event mới được tạo trên Google Calendar:

* Hệ thống nhận thông tin sự kiện ngay lập tức qua cơ chế webhook (Google Calendar – New Event)
* Kiểm tra người tổ chức có phải là chính mình hoặc nằm trong danh sách được phép không — nếu không, workflow dừng lại (Condition – IF/ELSE)
* AI phân tích tiêu đề và mô tả của event, tự động tạo một bản briefing gồm: mục tiêu cuộc họp (gợi ý), các câu hỏi nên chuẩn bị, và checklist tài liệu cần mang theo (Ask AI)
* Hệ thống gửi email đến người tổ chức để xem trước nội dung brief và chờ xác nhận — người tổ chức click Approve hoặc Reject ngay trong Gmail trước khi email được gửi đi (Human in the Loop – Wait for Confirmation)
* Nếu được Approve, hệ thống gửi email preparation brief đến toàn bộ người tham dự — kèm thời gian, địa điểm, và link tài liệu nếu có trong mô tả (Gmail – Send Email)

**Value:**

* Người tham dự nhận brief ngay khi được mời họp, có đủ thời gian chuẩn bị dù lịch được tạo gấp
* Giảm thời gian "khởi động" ở đầu mỗi cuộc họp — mọi người đã có context trước
* Người tổ chức không cần gửi email chuẩn bị thủ công, tiết kiệm effort đặc biệt khi manage nhiều meetings song song
* Instant trigger đảm bảo brief được gửi trong vài giây — không chờ polling như các trigger dạng Scheduled

---

## 3. Workflow Cancellation Notification – Thông báo huỷ lịch tự động kèm đề xuất lịch mới

**Trigger:** Google Calendar – Event Cancelled

**Problem:**
Khi một cuộc họp bị huỷ trên Google Calendar, người tổ chức thường phải gửi thêm email riêng để giải thích lý do và đề xuất thời gian mới. Nếu quên gửi, người tham dự không biết meeting đã bị cancel và vẫn dành thời gian chờ đợi hoặc chuẩn bị vô ích. Điều này đặc biệt nghiêm trọng khi cancel xảy ra sát giờ họp.

**Solution:**
Xây dựng workflow tự động: ngay khi một sự kiện bị xoá hoặc hủy trên Google Calendar:

* Hệ thống phát hiện sự kiện đã bị hủy (Google Calendar – Event Cancelled)
* Kiểm tra người tổ chức có phải là chính mình hoặc nằm trong danh sách được phép không — nếu không, workflow dừng lại (Condition – IF/ELSE)
* AI soạn thảo email thông báo hủy lịch với ngôn ngữ lịch sự, tự động điền tên sự kiện và thời gian gốc, đồng thời đề xuất 2–3 khung giờ thay thế dựa trên context của event (Ask AI)
* Hệ thống gửi email đến người tổ chức để xem trước nội dung thông báo và chờ xác nhận — người tổ chức click Approve hoặc Reject ngay trong Gmail trước khi email được gửi đi (Human in the Loop – Wait for Confirmation)
* Nếu được Approve, hệ thống gửi email thông báo hủy đến toàn bộ người tham dự trong event bị hủy (Gmail – Send Email)
* Sự kiện hủy được ghi nhận vào sheet theo dõi để team có thể review pattern và cải thiện lịch họp về sau (Google Sheets – Insert Row)

**Value:**

* Toàn bộ người tham dự được thông báo ngay khi lịch bị hủy — không có ai bị bỏ lại ngồi chờ
* Người tổ chức không cần gửi email giải thích thủ công, tiết kiệm effort đặc biệt khi phải hủy nhiều lịch cùng lúc
* AI soạn sẵn lời đề xuất lịch mới giúp việc reschedule diễn ra nhanh hơn
* Sheet log giúp PM nhìn ra team nào/loại meeting nào hay bị hủy để điều chỉnh quy trình lên lịch

---

## 4. Workflow Shared Calendar Briefing – Tự động tóm tắt calendar mới được chia sẻ

**Trigger:** Google Calendar – New Calendar

**Problem:**
Khi đồng nghiệp share một calendar mới với bạn, bạn thường không biết calendar đó dùng để làm gì — tên calendar đôi khi quá ngắn hoặc không rõ nghĩa. Phải nhắn hỏi lại người share mới biết đây là calendar của dự án nào, team nào phụ trách, và mình cần quan tâm đến event loại nào. Việc này gây mất thời gian, đặc biệt khi bạn thường xuyên được thêm vào nhiều calendar cùng lúc.

**Solution:**
Xây dựng workflow tự động: khi một calendar mới xuất hiện trong tài khoản của bạn:

* Hệ thống phát hiện calendar mới vừa được share hoặc tạo (Google Calendar – New Calendar)
* AI đọc tên và mô tả của calendar, tự động soạn một briefing note ngắn gọn: calendar này dùng cho mục đích gì, thuộc dự án/team nào, và những loại event nào có thể xuất hiện trong đó (Ask AI)
* Hệ thống gửi briefing note đến chính bạn như một email thông báo nội bộ, giúp bạn nắm context ngay mà không cần hỏi người share (Gmail – Send Email)

**Value:**

* Hiểu ngay mục đích của calendar mới trong vài phút mà không cần liên hệ người share
* Không bị bỡ ngỡ khi thấy event lạ xuất hiện trong lịch — đã có context từ trước
* Đặc biệt hữu ích khi được thêm vào nhiều calendar cùng lúc (ví dụ: onboard dự án mới, join team mới)
* Briefing do AI tổng hợp từ tên và mô tả — không cần người share phải viết thêm gì

---

*Được tạo bởi AI Assistant — 2026-06-26*
