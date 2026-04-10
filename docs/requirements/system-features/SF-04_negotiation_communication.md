# SF-04: Negotiation & Communication

## Overview

```
Scope:            Cung cấp các công cụ trao đổi và thương thảo giữa BTC và doanh nghiệp sau khi
                  lời mời tài trợ được chấp nhận. Bao gồm nhắn tin theo ngữ cảnh thương vụ
                  (được áp dụng Data Masking trước khi thanh toán — SF-13), đặt lịch họp, ghi
                  nhận kết quả họp theo dạng notebook minh bạch, chia sẻ file đính kèm, tạo
                  thỏa thuận nháp, và theo dõi tiến trình thương thảo cho đến khi hai bên đồng
                  thuận và chuyển sang thanh toán phí dịch vụ.
System Boundary:
  IN:             Nhắn tin trao đổi trong deal context (có Data Masking — SF-13);
                  Chia sẻ file đính kèm; Đặt lịch họp;
                  Ghi nhận kết quả họp dưới dạng notebook; Theo dõi trạng thái thương thảo;
                  Tạo thỏa thuận nháp (Draft Agreement);
                  Xác nhận đồng thuận hai chiều để chuyển sang thanh toán phí (SF-12);
                  Hủy bỏ thương thảo khi chưa đồng thuận cuối cùng.
  OUT:            Gửi lời mời tài trợ (SF-03 — deal đã được tạo);
                  Data Masking / Anti-bypass (SF-13 — áp dụng lên tin nhắn);
                  Thanh toán phí dịch vụ (SF-12 — bắt đầu sau đồng thuận, AWAITING_PAYMENT);
                  Soạn thảo và ký kết hợp đồng (SF-05 — bắt đầu sau khi thanh toán hoàn tất);
                  Giao diện video call / họp trực tuyến trong nền tảng.
Assumptions:
  - [ASSUMED] Deal/Negotiation context được tạo tự động khi lời mời được chấp nhận (SF-03, FR-0303).
  - [ASSUMED] Nhắn tin hỗ trợ real-time (WebSocket hoặc tương đương).
  - [ASSUMED] File đính kèm hỗ trợ các định dạng phổ biến (PDF, DOCX, XLSX, JPEG, PNG) với giới hạn 10MB/file.
  - [ASSUMED] Hệ thống đặt lịch họp ghi nhận thông tin cuộc họp nhưng không tích hợp lịch bên ngoài (Google Calendar, v.v.) ở phiên bản đầu.
  - [ASSUMED] Hệ thống không cung cấp giao diện video call; meeting online chỉ được ghi nhận như một lịch hẹn có link/địa điểm tham chiếu bên ngoài.
  - [UPDATED — BP03] Tin nhắn trong deal context được áp dụng Data Masking (SF-13) cho đến khi
    thanh toán phí dịch vụ hoàn tất (SF-12, 2/2 payment).
  - [UPDATED — BP03] Đồng thuận (FR-0406) chuyển deal sang AWAITING_PAYMENT thay vì AGREED.
    Deal chỉ chuyển sang AGREED sau khi thanh toán hoàn tất (SF-12 FR-1207).
Gaps Detected:
  - Quy trình gốc không nêu rõ cơ chế xác nhận đồng thuận → cần bổ sung mutual confirmation flow.
  - Không nêu giới hạn thời gian cho giai đoạn thương thảo → cần bổ sung SLA hoặc nhắc nhở.
  - Không nêu rõ ai có thể tham gia thương thảo trong nhóm BTC (toàn bộ hay chỉ đại diện).
  - Cần khẳng định hệ thống không hỗ trợ giao diện thực hiện video call hay sinh nội dung cuộc họp từ video.
  - [RESOLVED — BP03] Bổ sung cơ chế tạo thỏa thuận nháp (FR-0408) và trạng thái AWAITING_PAYMENT.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Tài khoản đại diện Ban tổ chức (BTC) | `organizer` | Nhắn tin, gửi file, đặt lịch họp, ghi nhận kết quả, xác nhận đồng thuận |
| Tài khoản đại diện doanh nghiệp | `sponsor` | Nhắn tin, gửi file, phản hồi lịch họp, ghi nhận kết quả, xác nhận đồng thuận |
| Hệ thống | `system` | Gửi thông báo tin nhắn mới, nhắc nhở lịch họp, xử lý trạng thái deal |

---

## Functional Requirements

### FR-0401: Nhắn tin trao đổi trong deal context

```
ID:            FR-0401
Name:          Gửi và nhận tin nhắn trong phạm vi thương vụ
Description:   Hệ thống SHALL cho phép hai bên (organizer và sponsor) gửi và nhận tin nhắn
               văn bản trong phạm vi một deal cụ thể. Tin nhắn SHALL được hiển thị theo thứ tự
               thời gian và đánh dấu trạng thái đã đọc/chưa đọc. Hệ thống SHALL hỗ trợ
               giao tiếp real-time.
               [UPDATED — BP03] Tất cả tin nhắn văn bản SHALL được áp dụng bộ lọc Data Masking
               (SF-13, FR-1301) khi deal chưa hoàn tất thanh toán phí dịch vụ
               (deal.contact_unlocked = false). Thông tin liên hệ (SĐT, email, link MXH)
               sẽ bị che giấu cho đến khi cả hai bên thanh toán phí (SF-12, 2/2 hoàn tất).
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Actor nhập và gửi tin nhắn trong trang thương thảo của deal
Inputs:        deal_id (UUID), sender_id (UUID), content (text, required)
Outputs:       message_id (UUID), sent_at (timestamp), delivery_status (SENT | DELIVERED | READ),
               display_content (text — đã masking nếu contact_unlocked = false)
Business Rules: BR-0401, BR-0402
Acceptance Criteria:
  Given   deal "deal-001" đang ở trạng thái IN_PROGRESS
  And     organizer đang trong trang thương thảo
  When    organizer gửi tin nhắn "Chúng tôi muốn thảo luận về gói Nhà tài trợ chính"
  Then    hệ thống SHALL lưu tin nhắn và hiển thị cho sponsor trong real-time
  And     hệ thống SHALL gửi thông báo in-app cho sponsor nếu không đang online

  Given   deal chưa mở khóa (contact_unlocked = false)
  When    organizer gửi "Liên hệ tôi qua 0912345678"
  Then    hệ thống SHALL hiển thị "Liên hệ tôi qua **********" (Data Masking — SF-13)

  Given   deal ở trạng thái TERMINATED
  When    actor cố gửi tin nhắn
  Then    hệ thống SHALL từ chối với thông báo "Thương thảo đã kết thúc"
Priority:      MUST
```

### FR-0402: Chia sẻ file đính kèm

```
ID:            FR-0402
Name:          Gửi file đính kèm trong phạm vi thương thảo
Description:   Hệ thống SHALL cho phép hai bên gửi file đính kèm trong tin nhắn bao gồm
               tài liệu đề xuất, bản sửa đổi gói tài trợ, hình ảnh minh họa, v.v.
               Hệ thống SHALL xác thực định dạng và kích thước file theo BR-0403.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Actor đính kèm file vào tin nhắn
Inputs:        deal_id, sender_id, file (binary), file_name (string)
Outputs:       attachment_id (UUID), file_url, file_size, upload_status
Business Rules: BR-0403
Acceptance Criteria:
  Given   deal đang ở trạng thái IN_PROGRESS
  When    organizer gửi file "proposal_v2.pdf" (3MB)
  Then    hệ thống SHALL upload file thành công
  And     hệ thống SHALL hiển thị file đính kèm trong luồng tin nhắn
  And     sponsor có thể tải file về

  Given   organizer gửi file "video.mp4" (50MB)
  When    hệ thống kiểm tra kích thước
  Then    hệ thống SHALL từ chối với thông báo "File vượt quá giới hạn 10MB"

  Given   organizer gửi file "script.exe"
  When    hệ thống kiểm tra định dạng
  Then    hệ thống SHALL từ chối với thông báo "Định dạng file không được hỗ trợ"
Priority:      MUST
```

### FR-0403: Đặt lịch họp/meeting

```
ID:            FR-0403
Name:          Tạo lịch họp thương thảo
Description:   Hệ thống SHALL cho phép một bên đề xuất lịch họp/meeting bao gồm: ngày giờ,
               thời lượng dự kiến, chủ đề, và ghi chú. Bên còn lại có thể chấp nhận,
               từ chối, hoặc đề xuất thời gian khác. Hệ thống SHALL chỉ đóng vai trò
               ghi nhận lịch hẹn và gửi nhắc nhở; hệ thống không tổ chức hoặc host cuộc gọi.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer hoặc Sponsor (đề xuất), đối tác (phản hồi)
Trigger:       Actor nhấn "Đặt lịch họp" trong trang thương thảo
Inputs:        deal_id, proposed_by (UUID), meeting_datetime (datetime),
               duration_minutes (integer), topic (string, required),
               notes (text, optional), meeting_type (enum: ONLINE | IN_PERSON),
               meeting_link_or_location (string, optional)
Outputs:       meeting_id (UUID), status = PROPOSED, proposed_at (timestamp)
Business Rules: BR-0404
Acceptance Criteria:
  Given   deal đang ở trạng thái IN_PROGRESS
  When    organizer đề xuất họp ngày 15/05/2026 lúc 14:00, thời lượng 60 phút,
          chủ đề = "Thảo luận quyền lợi gói Gold"
  Then    hệ thống SHALL tạo meeting với trạng thái PROPOSED
  And     hệ thống SHALL thông báo cho sponsor qua in-app và email

  Given   meeting đã được CONFIRMED
  When    thời gian hiện tại là 30 phút trước giờ họp
  Then    hệ thống SHALL gửi thông báo nhắc nhở cho cả hai bên
Priority:      MUST
```

### FR-0404: Phản hồi lịch họp

```
ID:            FR-0404
Name:          Chấp nhận, từ chối, hoặc đề xuất lại lịch họp
Description:   Hệ thống SHALL cho phép bên nhận phản hồi lịch họp: chấp nhận (CONFIRMED),
               từ chối (DECLINED), hoặc đề xuất thời gian khác (RESCHEDULED).
               Khi đề xuất lại, hệ thống SHALL tạo meeting mới với thời gian mới
               và liên kết đến meeting gốc.
Classification: SYSTEM-SUPPORTED
Actor:         Recipient (Organizer hoặc Sponsor)
Trigger:       Bên nhận phản hồi đề xuất lịch họp
Inputs:        meeting_id, response (enum: CONFIRMED | DECLINED | RESCHEDULED),
               new_datetime (datetime, required if RESCHEDULED),
               response_note (text, optional)
Outputs:       meeting status cập nhật, notification cho bên đề xuất
Business Rules: BR-0404
Acceptance Criteria:
  Given   sponsor nhận được đề xuất họp ngày 15/05/2026
  When    sponsor nhấn "Chấp nhận"
  Then    hệ thống SHALL chuyển meeting sang trạng thái CONFIRMED
  And     hệ thống SHALL thông báo cho organizer

  Given   sponsor muốn đổi lịch
  When    sponsor chọn "Đề xuất lại" với new_datetime = 16/05/2026 lúc 10:00
  Then    hệ thống SHALL chuyển meeting gốc sang RESCHEDULED
  And     hệ thống SHALL tạo meeting mới với thời gian đề xuất
  And     hệ thống SHALL thông báo cho organizer về đề xuất mới
Priority:      MUST
```

### FR-0405: Ghi nhận kết quả họp

```
ID:            FR-0405
Name:          Lưu ghi chú và kết quả cuộc họp
Description:   Hệ thống SHALL cho phép cả hai bên ghi nhận kết quả sau cuộc họp bao gồm:
               tóm tắt nội dung, các quyết định đã thống nhất, và các action items tiếp theo.
               Ghi chú này được lưu trong deal context theo dạng notebook chung để tham khảo
               khi soạn hợp đồng; hệ thống không tạo nội dung tự động từ video meeting.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Actor nhấn "Ghi nhận kết quả" sau cuộc họp hoặc trong trang chi tiết meeting
Inputs:        meeting_id, summary (text), decisions[] (text[]),
               action_items[] (text[]), noted_by (UUID)
Outputs:       meeting_note_id (UUID), noted_at (timestamp)
Business Rules: Không có BR riêng
Acceptance Criteria:
  Given   cuộc họp "Thảo luận quyền lợi gói Gold" đã diễn ra (CONFIRMED)
  When    organizer nhập summary = "Đồng ý gói Gold 20 triệu, bao gồm logo backdrop"
  Then    hệ thống SHALL lưu ghi chú cuộc họp
  And     ghi chú SHALL hiển thị cho cả hai bên trong deal context
Priority:      SHOULD
```

### FR-0406: Xác nhận đồng thuận hai chiều

```
ID:            FR-0406
Name:          Xác nhận đồng thuận và chuyển sang thanh toán phí dịch vụ
Description:   Hệ thống SHALL yêu cầu CẢ HAI bên (organizer VÀ sponsor) xác nhận đồng thuận.
               Mỗi bên nhấn "Xác nhận sẵn sàng". Khi CẢ HAI bên đã xác nhận:
               - Nếu deal có hiện vật (IN_KIND/COMBINED): hệ thống SHALL yêu cầu xác nhận
                 miễn trừ trách nhiệm hiện vật (SF-13, FR-1304) trước khi tiếp tục.
               - Sau khi đủ điều kiện, hệ thống SHALL chuyển deal sang trạng thái
                 AWAITING_PAYMENT và kích hoạt Paywall (SF-12, FR-1201).
               [UPDATED — BP03] Deal KHÔNG trực tiếp chuyển sang AGREED nữa.
               Luồng mới: IN_PROGRESS → AWAITING_PAYMENT → AGREED (sau khi 2/2 thanh toán).
Classification: SYSTEM-SUPPORTED
Actor:         Organizer (xác nhận), Sponsor (xác nhận), System (kiểm tra song phương)
Trigger:       Actor nhấn "Xác nhận đồng thuận" trong trang thương thảo
Inputs:        deal_id, confirmed_by (UUID)
Outputs:       deal.organizer_confirmed (boolean), deal.sponsor_confirmed (boolean),
               deal.status = AWAITING_PAYMENT (khi cả hai đã xác nhận + disclaimer nếu cần)
Business Rules: BR-0405
Acceptance Criteria:
  Given   deal đang ở trạng thái IN_PROGRESS
  When    organizer nhấn "Xác nhận đồng thuận"
  Then    hệ thống SHALL ghi nhận organizer_confirmed = true
  And     hệ thống SHALL thông báo cho sponsor "BTC đã xác nhận sẵn sàng"
  But     deal vẫn ở trạng thái IN_PROGRESS (chờ sponsor xác nhận)

  Given   organizer_confirmed = true VÀ sponsor nhấn "Xác nhận đồng thuận"
  And     deal.sponsorship_type = CASH (không cần disclaimer)
  When    hệ thống kiểm tra
  Then    hệ thống SHALL chuyển deal sang trạng thái AWAITING_PAYMENT
  And     hệ thống SHALL kích hoạt Paywall (SF-12, FR-1201)
  And     hệ thống SHALL thông báo "Đồng thuận hoàn tất. Vui lòng thanh toán phí dịch vụ."

  Given   organizer_confirmed = true VÀ sponsor nhấn "Xác nhận đồng thuận"
  And     deal.sponsorship_type = IN_KIND hoặc COMBINED
  When    hệ thống kiểm tra
  Then    hệ thống SHALL hiển thị checkbox miễn trừ trách nhiệm (SF-13, FR-1304)
  And     deal SHALL KHÔNG chuyển sang AWAITING_PAYMENT cho đến khi cả hai tích chọn
Priority:      MUST
```

### FR-0407: Hủy bỏ thương thảo

```
ID:            FR-0407
Name:          Hủy bỏ thương thảo
Description:   Hệ thống SHALL cho phép một trong hai bên hủy bỏ thương thảo tại bất kỳ thời điểm
               nào khi deal đang ở trạng thái IN_PROGRESS. Bên hủy PHẢI cung cấp lý do.
               Hệ thống SHALL chuyển deal sang trạng thái TERMINATED và thông báo cho bên còn lại.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer hoặc Sponsor
Trigger:       Actor nhấn "Hủy bỏ thương thảo"
Inputs:        deal_id, terminated_by (UUID), termination_reason (text, required)
Outputs:       deal.status = TERMINATED, terminated_at (timestamp)
Business Rules: BR-0406
Acceptance Criteria:
  Given   deal đang ở trạng thái IN_PROGRESS
  When    sponsor nhấn "Hủy bỏ" với lý do = "Đã quá ngân sách năm nay"
  Then    hệ thống SHALL chuyển deal sang TERMINATED
  And     hệ thống SHALL thông báo cho organizer bao gồm lý do hủy

  Given   deal đang ở trạng thái AGREED
  When    actor cố hủy bỏ
  Then    hệ thống SHALL từ chối "Không thể hủy thương thảo đã đồng thuận. Vui lòng liên hệ hỗ trợ."
Priority:      MUST
```

### FR-0408: Tạo thỏa thuận nháp (Draft Agreement)

```
ID:            FR-0408
Name:          Tạo và xác nhận thỏa thuận nháp trước khi lock-in
Description:   Hệ thống SHALL cho phép một bên tạo "Thỏa thuận nháp" (Draft Agreement)
               tóm tắt các điều khoản đã thống nhất trong quá trình thương thảo. Thỏa thuận
               nháp bao gồm: hình thức tài trợ, giá trị tài trợ (nếu tiền mặt), mô tả
               hiện vật (nếu có), và các điều khoản chính đã đồng ý. Bên còn lại xem và
               nhấn "Xác nhận thỏa thuận". Khi cả hai bên xác nhận, thỏa thuận nháp
               trở thành cơ sở để tính phí dịch vụ (SF-12) và soạn hợp đồng (SF-05).
               [ADDED — BP03 Bước 1: "một bên tạo Thỏa thuận nháp, bên còn lại bấm Xác nhận"]
Classification: SYSTEM-SUPPORTED
Actor:         Organizer hoặc Sponsor (tạo), đối tác (xác nhận)
Trigger:       Actor nhấn "Tạo thỏa thuận nháp" trong trang thương thảo
Inputs:        deal_id, created_by (UUID),
               sponsorship_type (CASH | IN_KIND | COMBINED),
               sponsorship_value (decimal, required if CASH/COMBINED),
               in_kind_description (text, required if IN_KIND/COMBINED),
               key_terms (text — tóm tắt điều khoản chính)
Outputs:       draft_agreement_id (UUID), status = PENDING_CONFIRMATION,
               notification cho đối tác
Business Rules: BR-0407
Acceptance Criteria:
  Given   deal đang ở trạng thái IN_PROGRESS
  When    organizer tạo thỏa thuận nháp với sponsorship_type = CASH,
          value = 20.000.000 VNĐ, key_terms = "Gói Nhà tài trợ chính"
  Then    hệ thống SHALL tạo draft agreement với trạng thái PENDING_CONFIRMATION
  And     hệ thống SHALL thông báo cho sponsor "BTC đã tạo thỏa thuận nháp. Vui lòng xem xét."

  Given   sponsor xem thỏa thuận nháp
  When    sponsor nhấn "Xác nhận thỏa thuận"
  Then    hệ thống SHALL chuyển draft agreement sang CONFIRMED
  And     hệ thống SHALL mở khóa nút "Xác nhận đồng thuận" (FR-0406)

  Given   sponsor không đồng ý nội dung
  When    sponsor nhấn "Từ chối" với ghi chú = "Cần điều chỉnh giá trị tài trợ"
  Then    hệ thống SHALL chuyển draft agreement sang REJECTED
  And     bên tạo có thể tạo lại thỏa thuận nháp mới
Priority:      MUST
```

---

## Business Rules

```
ID:          BR-0401
Rule:        Chỉ hai bên liên quan trong deal (sender và recipient của lời mời đã ACCEPTED)
             mới có quyền gửi tin nhắn, đặt lịch họp, và chia sẻ file trong deal context đó.
Source:      Quy trình gốc — Bước 3 (hai bên sẽ trao đổi với nhau)
Type:        Authorization

ID:          BR-0402
Rule:        Tin nhắn chỉ có thể gửi khi deal ở trạng thái IN_PROGRESS, AWAITING_PAYMENT,
             hoặc AGREED. Deal ở trạng thái TERMINATED không cho phép gửi tin nhắn mới.
             [UPDATED — BP03] Thêm AWAITING_PAYMENT vào danh sách trạng thái cho phép nhắn tin.
Source:      [INFERRED — bảo vệ tính nhất quán trạng thái]
Type:        Routing

ID:          BR-0403
Rule:        File đính kèm PHẢI có định dạng: PDF, DOCX, XLSX, JPEG, PNG, WebP.
             Kích thước tối đa mỗi file: 10MB. Tối đa 5 file cho mỗi tin nhắn. [ASSUMED]
Source:      [INFERRED — giới hạn kỹ thuật và bảo mật]
Type:        Validation

ID:          BR-0404
Rule:        Lịch họp chỉ có thể được đề xuất với ngày giờ trong tương lai (>= 1 giờ từ thời điểm hiện tại).
             Hệ thống SHALL gửi thông báo nhắc nhở 30 phút trước giờ họp.
Source:      Quy trình gốc — Bước 3 (Đặt lịch họp/meeting)
Type:        Validation + Time-based

ID:          BR-0405
Rule:        [UPDATED — BP03] Deal chuyển từ IN_PROGRESS sang AWAITING_PAYMENT khi CẢ HAI bên
             đều xác nhận đồng thuận (và miễn trừ hiện vật nếu cần — SF-13 FR-1304).
             Deal chuyển từ AWAITING_PAYMENT sang AGREED chỉ khi thanh toán hoàn tất
             (SF-12, 2/2 payment — FR-1207). Xác nhận đồng thuận có thể rút lại trước khi
             bên còn lại xác nhận.
Source:      Quy trình gốc — Bước 3 + BP03 Bước 1 (lock-in → chờ thanh toán)
Type:        Routing

ID:          BR-0406
Rule:        Lý do hủy bỏ thương thảo là BẮT BUỘC (min 10 ký tự).
             Deal chỉ có thể hủy khi ở trạng thái IN_PROGRESS. [ASSUMED]
             Deal đã AWAITING_PAYMENT hoặc AGREED không thể hủy qua giao diện thương thảo.
             [UPDATED — BP03] Thêm AWAITING_PAYMENT vào danh sách trạng thái không thể hủy.
Source:      [INFERRED — audit trail và bảo vệ deal đã lock-in]
Type:        Validation + Routing

ID:          BR-0407
Rule:        Thỏa thuận nháp (Draft Agreement) là BẮT BUỘC trước khi hai bên có thể xác nhận
             đồng thuận (FR-0406). Mỗi deal chỉ có MỘT thỏa thuận nháp CONFIRMED tại một thời điểm.
             Nếu thỏa thuận nháp bị REJECTED, bên tạo có thể tạo lại. Thỏa thuận nháp xác định
             sponsorship_type và sponsorship_value — cơ sở để tính phí dịch vụ (SF-12).
             [ADDED — BP03 Bước 1]
Source:      BP03 — Bước 1 ("một bên tạo Thỏa thuận nháp, bên còn lại bấm Xác nhận")
Type:        Validation + Routing
```

---

## Data Model

```
Entity:        Deal
Attributes:
  - deal_id: UUID (PK)
  - invitation_id: UUID (FK → SponsorshipInvitation, UNIQUE)
  - proposal_id: UUID (FK → SponsorshipProposal)
  - organizer_id: UUID (FK → User)
  - sponsor_id: UUID (FK → User)
  - status: Enum [IN_PROGRESS, AWAITING_PAYMENT, AGREED, TERMINATED] (default: IN_PROGRESS)
           [UPDATED — BP03] Thêm AWAITING_PAYMENT giữa IN_PROGRESS và AGREED.
  - contact_unlocked: Boolean (default: false)
           [ADDED — BP03] Đánh dấu thông tin liên hệ đã được mở khóa sau thanh toán.
  - unlocked_at: DateTime (nullable)
           [ADDED — BP03] Thời điểm mở khóa thông tin liên hệ.
  - organizer_confirmed: Boolean (default: false)
  - sponsor_confirmed: Boolean (default: false)
  - termination_reason: Text (nullable)
  - terminated_by: UUID (FK → User, nullable)
  - created_at: DateTime
  - updated_at: DateTime
  - awaiting_payment_at: DateTime (nullable)
           [ADDED — BP03] Thời điểm chuyển sang AWAITING_PAYMENT.
  - agreed_at: DateTime (nullable)
  - terminated_at: DateTime (nullable)
Relationships:
  - Deal —(1:1)→ SponsorshipInvitation
  - Deal —(N:1)→ SponsorshipProposal
  - Deal —(1:N)→ DealMessage
  - Deal —(1:N)→ Meeting
  - Deal —(0..1:1)→ DraftAgreement [ADDED — BP03]
  - Deal —(0..1:1)→ PaywallSession (SF-12) [ADDED — BP03]
  - Deal —(1:1)→ Contract (khi tiến sang SF-05, sau khi thanh toán hoàn tất)

Entity:        DraftAgreement
Description:   Thỏa thuận nháp — tóm tắt điều khoản đã thống nhất.
               [ADDED — BP03 Bước 1]
Attributes:
  - agreement_id: UUID (PK)
  - deal_id: UUID (FK → Deal, UNIQUE)
  - created_by: UUID (FK → User)
  - sponsorship_type: Enum [CASH, IN_KIND, COMBINED]
  - sponsorship_value: Decimal (nullable — bắt buộc nếu CASH/COMBINED)
  - in_kind_description: Text (nullable — bắt buộc nếu IN_KIND/COMBINED)
  - key_terms: Text (tóm tắt điều khoản chính)
  - status: Enum [PENDING_CONFIRMATION, CONFIRMED, REJECTED] (default: PENDING_CONFIRMATION)
  - confirmed_by: UUID (FK → User, nullable)
  - confirmed_at: DateTime (nullable)
  - rejection_note: Text (nullable)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - DraftAgreement —(1:1)→ Deal

Entity:        DealMessage
Attributes:
  - message_id: UUID (PK)
  - deal_id: UUID (FK → Deal)
  - sender_id: UUID (FK → User)
  - content: Text (required)
  - sent_at: DateTime
  - read_at: DateTime (nullable)
  - delivery_status: Enum [SENT, DELIVERED, READ]
Relationships:
  - DealMessage —(N:1)→ Deal
  - DealMessage —(1:N)→ MessageAttachment

Entity:        MessageAttachment
Attributes:
  - attachment_id: UUID (PK)
  - message_id: UUID (FK → DealMessage)
  - file_name: String (required)
  - file_url: String (required)
  - file_size: Integer (bytes)
  - file_type: String (MIME type)
  - uploaded_at: DateTime
Relationships:
  - MessageAttachment —(N:1)→ DealMessage

Entity:        Meeting
Attributes:
  - meeting_id: UUID (PK)
  - deal_id: UUID (FK → Deal)
  - proposed_by: UUID (FK → User)
  - meeting_datetime: DateTime (required)
  - duration_minutes: Integer (required)
  - topic: String (required)
  - notes: Text (nullable)
  - meeting_type: Enum [ONLINE, IN_PERSON]
  - meeting_link_or_location: String (nullable)
  - status: Enum [PROPOSED, CONFIRMED, DECLINED, RESCHEDULED, COMPLETED]
  - response_note: Text (nullable)
  - rescheduled_from_id: UUID (FK → Meeting, nullable)
  - proposed_at: DateTime
  - responded_at: DateTime (nullable)
Relationships:
  - Meeting —(N:1)→ Deal
  - Meeting —(0..1:1)→ MeetingNote

Entity:        MeetingNote
Attributes:
  - meeting_note_id: UUID (PK)
  - meeting_id: UUID (FK → Meeting, UNIQUE)
  - summary: Text (required)
  - decisions: Text[] (array of decision strings)
  - action_items: Text[] (array of action item strings)
  - noted_by: UUID (FK → User)
  - noted_at: DateTime
Relationships:
  - MeetingNote —(1:1)→ Meeting
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 3. Nhắn tin trao đổi (có Data Masking — SF-13) | SYSTEM-SUPPORTED | FR-0401 | BR-0401, BR-0402 | DealMessage |
| 3. Chia sẻ file trong thương thảo | SYSTEM-SUPPORTED | FR-0402 | BR-0403 | MessageAttachment |
| 3. Đặt lịch họp/meeting | SYSTEM-SUPPORTED | FR-0403 | BR-0404 | Meeting |
| 3. Phản hồi lịch họp | SYSTEM-SUPPORTED | FR-0404 | BR-0404 | Meeting |
| [INFERRED] Ghi nhận kết quả họp | SYSTEM-SUPPORTED | FR-0405 | — | MeetingNote |
| BP03-1. Tạo thỏa thuận nháp | SYSTEM-SUPPORTED | FR-0408 | BR-0407 | DraftAgreement |
| 3/BP03-1. Đồng thuận → AWAITING_PAYMENT | SYSTEM-SUPPORTED | FR-0406 | BR-0405 | Deal |
| [INFERRED] Hủy bỏ thương thảo | SYSTEM-SUPPORTED | FR-0407 | BR-0406 | Deal |
