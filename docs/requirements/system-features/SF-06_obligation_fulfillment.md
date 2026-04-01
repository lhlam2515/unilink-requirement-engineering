# SF-06: Obligation Fulfillment

## Overview

```
Scope:            Theo dõi và quản lý việc thực hiện nghĩa vụ tài trợ của cả hai bên
                  (doanh nghiệp và BTC) trong suốt thời gian hợp đồng có hiệu lực —
                  từ tạo danh sách nghĩa vụ, ghi nhận hoàn thành, đến nộp báo cáo kết quả sự kiện.
System Boundary:
  IN:             Tạo danh sách nghĩa vụ từ hợp đồng đã ký; Theo dõi trạng thái từng nghĩa vụ;
                  Ghi nhận hoàn thành với bằng chứng; Nộp báo cáo kết quả sự kiện;
                  Thông báo nhắc nhở nghĩa vụ sắp đến hạn và quá hạn.
  OUT:            Soạn thảo và ký kết hợp đồng (SF-05 — đã hoàn tất);
                  Đánh giá sau hợp đồng (SF-07 — bắt đầu sau khi hợp đồng kết thúc).
Assumptions:
  - [ASSUMED] Danh sách nghĩa vụ được tạo tự động từ điều khoản hợp đồng khi ký kết.
  - [ASSUMED] Bằng chứng hoàn thành bao gồm file đính kèm (ảnh, PDF) và/hoặc mô tả văn bản.
  - [ASSUMED] Hệ thống gửi nhắc nhở tự động 3 ngày trước hạn và vào ngày hạn.
  - [ASSUMED] Bên xác nhận hoàn thành nghĩa vụ là bên đối tác (không phải bên thực hiện).
Gaps Detected:
  - Quy trình gốc không nêu rõ cơ chế giải quyết tranh chấp khi một bên không đồng ý
    xác nhận hoàn thành → cần bổ sung flow dispute.
  - Không nêu rõ quy trình khi một bên không thực hiện nghĩa vụ đúng hạn.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Doanh nghiệp | `sponsor` | Ghi nhận hoàn thành nghĩa vụ tài trợ (chuyển tiền/hiện vật); Xác nhận hoàn thành nghĩa vụ BTC; Xem báo cáo sự kiện |
| Ban tổ chức (BTC) | `organizer` | Ghi nhận hoàn thành nghĩa vụ quảng bá/tổ chức; Xác nhận hoàn thành nghĩa vụ doanh nghiệp; Nộp báo cáo kết quả sự kiện |
| Hệ thống | `system` | Tạo nghĩa vụ tự động, gửi nhắc nhở, theo dõi deadline, cập nhật trạng thái hợp đồng |

---

## Functional Requirements

### FR-0601: Tạo danh sách nghĩa vụ từ hợp đồng

```
ID:            FR-0601
Name:          Tự động tạo danh sách nghĩa vụ tài trợ từ hợp đồng đã ký
Description:   Hệ thống SHALL tự động tạo danh sách nghĩa vụ cho cả hai bên khi hợp đồng
               chuyển sang trạng thái SIGNED. Nghĩa vụ của doanh nghiệp bao gồm: chuyển khoản
               tiền mặt và/hoặc bàn giao hiện vật. Nghĩa vụ của BTC bao gồm: thực hiện
               quảng bá, truyền thông, tổ chức sự kiện, và nộp báo cáo kết quả.
               Mỗi nghĩa vụ có thời hạn dựa trên hợp đồng.
Classification: FULLY AUTOMATED
Actor:         System (khởi tạo)
Trigger:       Contract chuyển sang trạng thái SIGNED (từ SF-05, FR-0504)
Inputs:        contract_id, sponsor_obligations (từ hợp đồng), organizer_obligations (từ hợp đồng),
               transaction_deadline, validity_end
Outputs:       obligations[]: { obligation_id, type, responsible_party, description, deadline, status = PENDING }
Business Rules: BR-0601
Acceptance Criteria:
  Given   hợp đồng "contract-001" vừa được ký với hình thức CASH + IN_KIND
  When    hệ thống xử lý sự kiện SIGNED
  Then    hệ thống SHALL tạo nghĩa vụ cho sponsor: "Chuyển khoản XXX VND trước [deadline]"
  And     hệ thống SHALL tạo nghĩa vụ cho sponsor: "Bàn giao hiện vật trước [deadline]"
  And     hệ thống SHALL tạo nghĩa vụ cho organizer: "Thực hiện quảng bá theo cam kết"
  And     hệ thống SHALL tạo nghĩa vụ cho organizer: "Nộp báo cáo kết quả sự kiện"
  And     tất cả nghĩa vụ có trạng thái ban đầu PENDING
Priority:      MUST
```

### FR-0602: Theo dõi trạng thái nghĩa vụ

```
ID:            FR-0602
Name:          Xem và theo dõi tiến trình thực hiện nghĩa vụ
Description:   Hệ thống SHALL hiển thị dashboard nghĩa vụ cho cả hai bên, bao gồm:
               danh sách nghĩa vụ của mình và đối tác, trạng thái từng nghĩa vụ
               (PENDING, IN_PROGRESS, SUBMITTED, CONFIRMED, OVERDUE),
               deadline, và tiến trình tổng thể.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Actor truy cập trang "Nghĩa vụ tài trợ" của hợp đồng
Inputs:        contract_id, actor_id
Outputs:       obligations[]: { obligation_id, type, responsible_party, description,
               deadline, status, evidence[], submitted_at, confirmed_at },
               progress_summary: { total, completed, pending, overdue }
Business Rules: Không có BR riêng
Acceptance Criteria:
  Given   hợp đồng "contract-001" có 4 nghĩa vụ (2 sponsor, 2 organizer)
  When    organizer truy cập trang nghĩa vụ
  Then    hệ thống SHALL hiển thị tất cả 4 nghĩa vụ với trạng thái hiện tại
  And     hệ thống SHALL hiển thị progress: "1/4 hoàn thành"
  And     nghĩa vụ quá hạn SHALL đánh dấu màu đỏ hoặc label "QUÁ HẠN"
Priority:      MUST
```

### FR-0603: Ghi nhận hoàn thành nghĩa vụ

```
ID:            FR-0603
Name:          Nộp bằng chứng hoàn thành nghĩa vụ
Description:   Hệ thống SHALL cho phép bên chịu trách nhiệm ghi nhận việc hoàn thành nghĩa vụ
               bằng cách nộp bằng chứng (file đính kèm và/hoặc mô tả). Hệ thống SHALL chuyển
               trạng thái nghĩa vụ sang SUBMITTED và thông báo cho bên đối tác xác nhận.
Classification: SYSTEM-SUPPORTED
Actor:         Bên chịu trách nhiệm (Organizer hoặc Sponsor tùy nghĩa vụ)
Trigger:       Actor nhấn "Báo cáo hoàn thành" trên một nghĩa vụ cụ thể
Inputs:        obligation_id, evidence_description (text, required),
               evidence_files[] (binary[], optional), submitted_by (UUID)
Outputs:       status = SUBMITTED, submitted_at (timestamp),
               notification cho bên đối tác
Business Rules: BR-0602
Acceptance Criteria:
  Given   sponsor có nghĩa vụ "Chuyển khoản 20,000,000 VND"
  When    sponsor nhấn "Báo cáo hoàn thành" với mô tả = "Đã chuyển khoản ngày 10/05"
          và đính kèm ảnh chụp biên lai chuyển khoản
  Then    hệ thống SHALL chuyển nghĩa vụ sang SUBMITTED
  And     hệ thống SHALL thông báo cho organizer "Doanh nghiệp báo cáo hoàn thành nghĩa vụ [tên]"

  Given   nghĩa vụ đã ở trạng thái CONFIRMED
  When    actor cố nộp lại
  Then    hệ thống SHALL từ chối "Nghĩa vụ đã được xác nhận hoàn thành"
Priority:      MUST
```

### FR-0604: Xác nhận hoàn thành nghĩa vụ bởi đối tác

```
ID:            FR-0604
Name:          Xác nhận hoàn thành nghĩa vụ bởi bên đối tác
Description:   Hệ thống SHALL cho phép bên ĐỐI TÁC (không phải bên thực hiện) xác nhận
               rằng nghĩa vụ đã được hoàn thành đúng cam kết. Khi xác nhận,
               hệ thống SHALL chuyển trạng thái sang CONFIRMED. Nếu bên đối tác
               không đồng ý, có thể từ chối xác nhận với lý do.
Classification: SYSTEM-SUPPORTED
Actor:         Bên đối tác (Organizer xác nhận nghĩa vụ Sponsor, và ngược lại)
Trigger:       Bên đối tác nhấn "Xác nhận" hoặc "Từ chối" trên nghĩa vụ đã SUBMITTED
Inputs:        obligation_id, action (enum: CONFIRM | REJECT),
               rejection_reason (text, required if REJECT), confirmed_by (UUID)
Outputs:       status = CONFIRMED (if accepted) hoặc DISPUTED (if rejected),
               confirmed_at hoặc disputed_at (timestamp)
Business Rules: BR-0603
Acceptance Criteria:
  Given   sponsor đã báo cáo hoàn thành "Chuyển khoản 20,000,000 VND" (SUBMITTED)
  When    organizer kiểm tra biên lai và nhấn "Xác nhận"
  Then    hệ thống SHALL chuyển nghĩa vụ sang CONFIRMED
  And     hệ thống SHALL thông báo cho sponsor "Nghĩa vụ [tên] đã được xác nhận"

  Given   organizer không đồng ý với bằng chứng
  When    organizer nhấn "Từ chối" với lý do = "Số tiền chuyển không khớp hợp đồng"
  Then    hệ thống SHALL chuyển nghĩa vụ sang DISPUTED
  And     hệ thống SHALL thông báo cho sponsor kèm lý do từ chối
  And     sponsor có thể nộp bằng chứng mới (flow quay lại FR-0603)
Priority:      MUST
```

### FR-0605: Nộp báo cáo kết quả sự kiện

```
ID:            FR-0605
Name:          Nộp báo cáo kết quả sự kiện cho nhà tài trợ
Description:   Hệ thống SHALL cho phép organizer nộp báo cáo kết quả sự kiện cho sponsor,
               bao gồm: tóm tắt sự kiện, số liệu thực tế (lượng khán giả, reach truyền thông),
               hình ảnh/video sự kiện, đánh giá mức độ hoàn thành quyền lợi nhà tài trợ,
               và các file đính kèm báo cáo chi tiết.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer (nộp), Sponsor (xem)
Trigger:       Organizer nhấn "Nộp báo cáo kết quả" trong trang hợp đồng
Inputs:        contract_id, report_summary (text), actual_attendance (integer),
               media_reach (text), event_photos[] (binary[]),
               benefit_fulfillment_notes (text), report_files[] (binary[])
Outputs:       report_id (UUID), submitted_at (timestamp),
               notification cho sponsor
Business Rules: BR-0604
Acceptance Criteria:
  Given   sự kiện "UniHack 2026" đã kết thúc
  When    organizer nộp báo cáo với actual_attendance = 450,
          media_reach = "10,000 views trên fanpage", và đính kèm ảnh sự kiện
  Then    hệ thống SHALL lưu báo cáo và gắn vào hợp đồng
  And     hệ thống SHALL thông báo cho sponsor "BTC đã nộp báo cáo kết quả sự kiện"

  Given   hợp đồng chưa ở trạng thái SIGNED
  When    organizer cố nộp báo cáo
  Then    hệ thống SHALL từ chối "Chỉ có thể nộp báo cáo cho hợp đồng đã ký"
Priority:      MUST
```

### FR-0606: Thông báo nhắc nhở nghĩa vụ

```
ID:            FR-0606
Name:          Gửi thông báo nhắc nhở nghĩa vụ sắp đến hạn và quá hạn
Description:   Hệ thống SHALL tự động gửi thông báo nhắc nhở qua in-app và email:
               - 3 ngày trước hạn: "Nghĩa vụ [tên] sẽ đến hạn vào [ngày]"
               - Ngày đến hạn: "Nghĩa vụ [tên] đến hạn hôm nay"
               - 1 ngày sau hạn (quá hạn): "Nghĩa vụ [tên] đã quá hạn"
               Hệ thống SHALL tự động chuyển trạng thái sang OVERDUE khi quá hạn.
Classification: FULLY AUTOMATED
Actor:         System (khởi tạo)
Trigger:       Scheduled job kiểm tra deadline nghĩa vụ (chạy hàng ngày)
Inputs:        Danh sách nghĩa vụ có trạng thái PENDING hoặc IN_PROGRESS
Outputs:       Notifications cho bên chịu trách nhiệm, status = OVERDUE (nếu quá hạn)
Business Rules: BR-0605
Acceptance Criteria:
  Given   nghĩa vụ "Chuyển khoản" có deadline = 15/05/2026
  And     hôm nay là 12/05/2026 (3 ngày trước hạn)
  When    scheduled job chạy
  Then    hệ thống SHALL gửi thông báo in-app và email cho sponsor
          "Nghĩa vụ 'Chuyển khoản' sẽ đến hạn vào 15/05/2026"

  Given   hôm nay là 16/05/2026 và nghĩa vụ vẫn PENDING
  When    scheduled job chạy
  Then    hệ thống SHALL chuyển trạng thái sang OVERDUE
  And     hệ thống SHALL thông báo cho cả hai bên
Priority:      SHOULD
```

---

## Business Rules

```
ID:          BR-0601
Rule:        Danh sách nghĩa vụ PHẢI được tạo tự động khi hợp đồng SIGNED.
             Nghĩa vụ doanh nghiệp: chuyển khoản/bàn giao hiện vật.
             Nghĩa vụ BTC: quảng bá, truyền thông, tổ chức, báo cáo.
             Deadline nghĩa vụ dựa trên transaction_deadline và validity_end của hợp đồng.
Source:      Quy trình gốc — Bước 5 (Thực hiện nghĩa vụ tài trợ)
Type:        Routing

ID:          BR-0602
Rule:        Bằng chứng hoàn thành PHẢI bao gồm mô tả văn bản tối thiểu 20 ký tự.
             File đính kèm là tùy chọn nhưng được khuyến khích. [ASSUMED]
             Nghĩa vụ ở trạng thái CONFIRMED không thể nộp lại.
Source:      [INFERRED — đảm bảo chất lượng bằng chứng]
Type:        Validation

ID:          BR-0603
Rule:        Chỉ BÊN ĐỐI TÁC (không phải bên thực hiện) mới có quyền xác nhận hoặc từ chối
             hoàn thành nghĩa vụ. Organizer xác nhận nghĩa vụ sponsor, sponsor xác nhận nghĩa vụ organizer.
             Lý do từ chối là BẮT BUỘC.
Source:      [INFERRED — kiểm soát chéo, ngăn tự xác nhận]
Type:        Authorization

ID:          BR-0604
Rule:        Báo cáo kết quả sự kiện chỉ nộp được cho hợp đồng ở trạng thái SIGNED.
             Mỗi hợp đồng chỉ có MỘT báo cáo kết quả. [ASSUMED]
Source:      Quy trình gốc — Bước 5 (BTC báo cáo kết quả sự kiện cho nhà tài trợ)
Type:        Validation

ID:          BR-0605
Rule:        Hệ thống PHẢI gửi nhắc nhở nghĩa vụ theo lịch: T-3 ngày, T-0 (ngày hạn), T+1 ngày.
             Nghĩa vụ PENDING hoặc IN_PROGRESS quá deadline SHALL tự động chuyển sang OVERDUE.
             Thông báo gửi qua in-app và email.
Source:      [INFERRED — đảm bảo thực thi hợp đồng]
Type:        Time-based
```

---

## Data Model

```
Entity:        Obligation
Attributes:
  - obligation_id: UUID (PK)
  - contract_id: UUID (FK → Contract)
  - responsible_party: Enum [ORGANIZER, SPONSOR]
  - obligation_type: Enum [CASH_TRANSFER, IN_KIND_DELIVERY, BRANDING, PROMOTION,
                           EVENT_EXECUTION, REPORTING, OTHER]
  - description: Text (required)
  - deadline: Date (required)
  - status: Enum [PENDING, IN_PROGRESS, SUBMITTED, CONFIRMED, DISPUTED, OVERDUE]
             (default: PENDING)
  - evidence_description: Text (nullable)
  - submitted_at: DateTime (nullable)
  - submitted_by: UUID (FK → User, nullable)
  - confirmed_at: DateTime (nullable)
  - confirmed_by: UUID (FK → User, nullable)
  - rejection_reason: Text (nullable)
  - disputed_at: DateTime (nullable)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - Obligation —(N:1)→ Contract
  - Obligation —(1:N)→ ObligationEvidence

Entity:        ObligationEvidence
Attributes:
  - evidence_id: UUID (PK)
  - obligation_id: UUID (FK → Obligation)
  - file_name: String (required)
  - file_url: String (required)
  - file_size: Integer (bytes)
  - file_type: String (MIME type)
  - uploaded_at: DateTime
  - uploaded_by: UUID (FK → User)
Relationships:
  - ObligationEvidence —(N:1)→ Obligation

Entity:        EventReport
Attributes:
  - report_id: UUID (PK)
  - contract_id: UUID (FK → Contract, UNIQUE)
  - report_summary: Text (required)
  - actual_attendance: Integer
  - media_reach: Text
  - benefit_fulfillment_notes: Text
  - submitted_at: DateTime
  - submitted_by: UUID (FK → User)
Relationships:
  - EventReport —(1:1)→ Contract
  - EventReport —(1:N)→ EventReportFile

Entity:        EventReportFile
Attributes:
  - file_id: UUID (PK)
  - report_id: UUID (FK → EventReport)
  - file_name: String
  - file_url: String
  - file_type: Enum [PHOTO, VIDEO, DOCUMENT]
  - file_size: Integer
  - uploaded_at: DateTime
Relationships:
  - EventReportFile —(N:1)→ EventReport
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 5. Tạo nghĩa vụ từ hợp đồng | FULLY AUTOMATED | FR-0601 | BR-0601 | Obligation |
| 5. Theo dõi trạng thái nghĩa vụ | SYSTEM-SUPPORTED | FR-0602 | — | Obligation |
| 5. DN chuyển khoản/bàn giao hiện vật | SYSTEM-SUPPORTED | FR-0603 | BR-0602 | Obligation, ObligationEvidence |
| 5. BTC thực hiện quảng bá, truyền thông | SYSTEM-SUPPORTED | FR-0603 | BR-0602 | Obligation, ObligationEvidence |
| 5. Đối tác xác nhận hoàn thành | SYSTEM-SUPPORTED | FR-0604 | BR-0603 | Obligation |
| 5. BTC báo cáo kết quả sự kiện | SYSTEM-SUPPORTED | FR-0605 | BR-0604 | EventReport |
| [INFERRED] Nhắc nhở deadline | FULLY AUTOMATED | FR-0606 | BR-0605 | Obligation |
