# SF-14: Payment Risk Management & Compliance

## Overview

```
Scope:            Quản lý rủi ro thanh toán phí dịch vụ nền tảng — bao gồm hoàn tiền tự động
                  khi hết hạn (Auto-Refund), thực thi chính sách không hoàn phí (Non-refundable),
                  thông báo nhắc nhở thanh toán, ghi nhận doanh thu nền tảng, xử lý vi phạm
                  ký kết hợp đồng sau hard-lock, và các cơ chế an toàn bổ sung cho dòng tiền
                  vận hành qua thanh toán bên ngoài.
System Boundary:
  IN:             Hoàn tiền tự động khi timeout; Thực thi non-refundable post-unlock;
                  Gửi nhắc nhở thanh toán; Ghi nhận doanh thu nền tảng; Đối soát webhook;
                  Cảnh báo bất thường dòng tiền; Tố cáo hành vi trì hoãn ký kết; Tạm khóa
                  tài khoản bị báo cáo; Cảnh báo cuối cùng; Kích hoạt chế tài.
  OUT:            Tính phí và theo dõi thanh toán (SF-12 — dữ liệu nguồn);
                  Cổng thanh toán bên ngoài (xử lý refund thực tế);
                  Hệ thống kế toán doanh nghiệp (ngoài phạm vi nền tảng).
Assumptions:
  - [ASSUMED] Hoàn tiền được thực hiện qua API cổng thanh toán hoặc chuyển khoản thủ công
    bởi kế toán nền tảng. Hệ thống chỉ ghi nhận yêu cầu hoàn tiền và theo dõi trạng thái.
  - [ASSUMED] Không có escrow — tiền phí dịch vụ được chuyển trực tiếp vào tài khoản nền tảng.
    Hoàn tiền từ tài khoản nền tảng về tài khoản gốc của bên đã nộp.
  - [ASSUMED] Doanh thu nền tảng = tổng phí dịch vụ đã thu (status = PAID) trừ tổng hoàn tiền
    (status = REFUNDED). Báo cáo doanh thu mang tính tham chiếu, không thay thế phần mềm kế toán.
Gaps Detected:
  - BP03 nêu "hoàn tiền 100% về tài khoản gốc" nhưng không chỉ rõ tài khoản gốc → cần
    xác nhận source account từ webhook bank_reference.
  - Không nêu cơ chế xử lý khi hoàn tiền thất bại (bank reject) → bổ sung retry + admin alert.
  - Không nêu cơ chế phát hiện thanh toán trùng (double payment) → bổ sung dedup check.
  - Không nêu luồng xử lý khi một bên cố tình trì hoãn ký sau hard-lock 72 giờ → cần
    bổ sung report/freeze/final warning/execution.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Tài khoản đại diện Ban tổ chức (CLB) | `organizer` | Nhận hoàn tiền; Nhận nhắc nhở thanh toán |
| Tài khoản đại diện Doanh nghiệp | `sponsor` | Nhận hoàn tiền; Nhận nhắc nhở thanh toán |
| Hệ thống | `system` | Kiểm tra timeout; Khởi tạo hoàn tiền; Gửi nhắc nhở; Ghi doanh thu; Phát hiện bất thường |
| Admin / Kế toán | `admin` | Xử lý hoàn tiền thủ công; Đối soát doanh thu; Xử lý bất thường |

---

## Functional Requirements

### FR-1401: Hoàn tiền tự động khi hết hạn 48 giờ (Auto-Refund)

```
ID:            FR-1401
Name:          Tự động hoàn tiền khi đối tác không thanh toán trong 48 giờ
Description:   Hệ thống SHALL tự động xử lý hoàn tiền khi PaywallSession hết hạn 48 giờ
               và chỉ có MỘT bên đã thanh toán (payment_count = 1/2).
               Quy trình Auto-Refund:
               1. Scheduled job kiểm tra PaywallSession có expires_at < now và status = ACTIVE.
               2. Nếu payment_count = 0: hủy thương vụ, không cần hoàn tiền.
               3. Nếu payment_count = 1: hoàn tiền 100% cho bên đã thanh toán.
               4. Cập nhật ServiceFeeTransaction sang REFUNDED.
               5. Cập nhật PaywallSession sang EXPIRED.
               6. Cập nhật Deal sang TERMINATED (lý do: "Hết hạn thanh toán phí dịch vụ").
               7. Thông báo cho cả hai bên.
               Hệ thống SHALL tạo bản ghi RefundTransaction cho audit trail.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       Scheduled job chạy mỗi 15 phút, kiểm tra PaywallSession hết hạn
Inputs:        Danh sách PaywallSession có status = ACTIVE và expires_at < now
Outputs:       refund_transaction: { refund_id, original_fee_id, refund_amount, status },
               paywall_session.status = EXPIRED,
               deal.status = TERMINATED,
               notifications cho cả hai bên
Business Rules: BR-1401, BR-1402
Acceptance Criteria:
  Given   PaywallSession cho deal "deal-001" hết hạn 48 giờ
  And     organizer đã thanh toán 100.000 VNĐ, sponsor chưa thanh toán
  When    scheduled job chạy
  Then    hệ thống SHALL tạo RefundTransaction hoàn 100.000 VNĐ cho organizer
  And     hệ thống SHALL cập nhật organizer_fee.status = REFUNDED
  And     hệ thống SHALL cập nhật paywall.status = EXPIRED
  And     hệ thống SHALL cập nhật deal.status = TERMINATED
  And     hệ thống SHALL thông báo organizer "Phí dịch vụ 100.000 VNĐ sẽ được hoàn về
          tài khoản gốc trong 3-5 ngày làm việc"
  And     hệ thống SHALL thông báo sponsor "Thương vụ đã bị hủy do không hoàn tất
          thanh toán trong thời hạn"

  Given   PaywallSession hết hạn và cả hai bên đều chưa thanh toán (0/2)
  When    scheduled job chạy
  Then    hệ thống SHALL hủy thương vụ (deal = TERMINATED) mà KHÔNG tạo RefundTransaction
  And     hệ thống SHALL thông báo cho cả hai bên

  Given   PaywallSession đã hoàn tất (2/2) trước khi hết hạn
  When    scheduled job chạy
  Then    hệ thống SHALL bỏ qua (paywall.status đã là COMPLETED)
Priority:      MUST
```

### FR-1402: Thực thi chính sách không hoàn phí (Non-refundable)

```
ID:            FR-1402
Name:          Thực thi chính sách không hoàn phí sau mở khóa thông tin
Description:   Hệ thống SHALL thực thi chính sách: ngay khi PaywallSession đạt trạng thái
               COMPLETED (2/2) và thông tin liên hệ đã được mở khóa, khoản phí dịch vụ
               KHÔNG ĐƯỢC HOÀN LẠI dưới bất kỳ hình thức nào, kể cả khi:
               - Hai bên hủy hợp đồng sau khi ký.
               - Phát sinh tranh chấp giữa hai bên.
               - Một bên yêu cầu hoàn tiền vì lý do cá nhân.
               Hệ thống SHALL từ chối mọi yêu cầu hoàn tiền cho ServiceFeeTransaction
               có status = PAID và paywall.status = COMPLETED.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       Bất kỳ yêu cầu hoàn tiền hoặc kiểm tra refund eligibility
Inputs:        fee_transaction_id, paywall_session.status
Outputs:       refund_eligible = false (khi paywall COMPLETED),
               rejection_message (string)
Business Rules: BR-1402
Acceptance Criteria:
  Given   paywall đã COMPLETED, deal đã mở khóa, hợp đồng đã ký
  When    sponsor yêu cầu hoàn tiền phí dịch vụ vì "Hủy tài trợ"
  Then    hệ thống SHALL từ chối "Phí dịch vụ không được hoàn lại sau khi thông tin
          liên hệ đã được mở khóa (Chính sách Non-refundable)"

  Given   admin cố gắng tạo refund thủ công cho fee đã COMPLETED
  When    admin kiểm tra refund eligibility
  Then    hệ thống SHALL cảnh báo "Giao dịch thuộc chính sách Non-refundable.
          Vui lòng xác nhận lý do đặc biệt nếu muốn tiếp tục."
  And     hệ thống SHALL yêu cầu admin nhập lý do ngoại lệ
Priority:      MUST
```

### FR-1403: Thông báo nhắc nhở thanh toán

```
ID:            FR-1403
Name:          Gửi thông báo nhắc nhở thanh toán phí dịch vụ
Description:   Hệ thống SHALL tự động gửi thông báo nhắc nhở qua in-app và email cho
               các bên chưa hoàn tất thanh toán phí dịch vụ theo lịch:
               - T-24h: "Còn 24 giờ để hoàn tất thanh toán phí dịch vụ"
               - T-6h: "Còn 6 giờ! Thương vụ sẽ bị hủy nếu không thanh toán kịp"
               - T-1h: "CẬP NHẬT KHẨN: Chỉ còn 1 giờ để thanh toán"
               Nhắc nhở chỉ gửi cho bên CHƯA thanh toán.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       Scheduled job kiểm tra PaywallSession đang ACTIVE
Inputs:        Danh sách PaywallSession status = ACTIVE, ServiceFeeTransaction status = PENDING
Outputs:       Notifications (in-app + email) cho bên chưa thanh toán
Business Rules: BR-1403
Acceptance Criteria:
  Given   PaywallSession bắt đầu lúc 10:00 ngày 01/05 (expires 10:00 ngày 03/05)
  And     hôm nay là 02/05 lúc 10:00 (T-24h)
  And     sponsor chưa thanh toán
  When    scheduled job chạy
  Then    hệ thống SHALL gửi nhắc nhở cho sponsor
          "Còn 24 giờ để hoàn tất thanh toán phí dịch vụ cho thương vụ [tên]"
  But     hệ thống SHALL KHÔNG gửi nhắc nhở cho organizer (đã PAID)

  Given   T-1h trước hết hạn
  When    scheduled job chạy
  Then    hệ thống SHALL gửi nhắc nhở khẩn cấp với mức ưu tiên cao
Priority:      SHOULD
```

### FR-1404: Ghi nhận và theo dõi doanh thu nền tảng

```
ID:            FR-1404
Name:          Ghi nhận doanh thu phí dịch vụ của nền tảng
Description:   Hệ thống SHALL tự động ghi nhận doanh thu nền tảng khí ServiceFeeTransaction
               chuyển sang trạng thái PAID. Hệ thống SHALL duy trì bảng PlatformRevenueLog
               để theo dõi: tổng thu, tổng hoàn tiền, doanh thu ròng. Admin có thể xem
               báo cáo doanh thu theo khoảng thời gian. Báo cáo mang tính tham chiếu,
               không thay thế phần mềm kế toán chính thức.
Classification: FULLY AUTOMATED (ghi nhận) + SYSTEM-SUPPORTED (xem báo cáo)
Actor:         System (ghi nhận), Admin (xem báo cáo)
Trigger:       ServiceFeeTransaction chuyển sang PAID hoặc REFUNDED
Inputs:        fee_transaction_id, amount, status, payer_role, timestamp
Outputs:       revenue_log_entry: { log_id, type (INCOME | REFUND), amount, net_balance }
Business Rules: BR-1404
Acceptance Criteria:
  Given   organizer thanh toán phí 100.000 VNĐ (PAID)
  When    hệ thống ghi nhận
  Then    hệ thống SHALL tạo revenue_log type = INCOME, amount = 100.000

  Given   phí 100.000 VNĐ bị hoàn tiền (REFUNDED)
  When    hệ thống ghi nhận
  Then    hệ thống SHALL tạo revenue_log type = REFUND, amount = -100.000

  Given   admin truy cập báo cáo doanh thu tháng 05/2026
  When    hệ thống tổng hợp
  Then    hệ thống SHALL hiển thị tổng thu, tổng hoàn, doanh thu ròng
Priority:      SHOULD
```

### FR-1405: Cơ chế an toàn bổ sung cho dòng tiền

```
ID:            FR-1405
Name:          Phát hiện và xử lý bất thường dòng tiền thanh toán
Description:   Hệ thống SHALL triển khai các cơ chế an toàn bổ sung để bảo vệ dòng tiền:
               1. Phát hiện thanh toán trùng (duplicate payment detection): Nếu nhận được
                  nhiều webhook với cùng transaction_reference → chỉ xử lý lần đầu,
                  các lần sau đánh dấu DUPLICATE và cảnh báo admin.
               2. Giám sát webhook health: Nếu cổng thanh toán không gửi webhook trong
                  thời gian dài (> 4 giờ kể từ khi tạo QR) → cảnh báo admin kiểm tra.
               3. Manual reconciliation interface: Admin có thể đối soát thủ công khi
                  webhook thất bại, đánh dấu ServiceFeeTransaction = PAID manually
                  với audit log.
               4. Idempotent webhook processing: Hệ thống PHẢI xử lý webhook idempotent,
                  đảm bảo cùng webhook gửi lại không gây side effect.
Classification: FULLY AUTOMATED (detection) + SYSTEM-SUPPORTED (admin reconciliation)
Actor:         System (giám sát), Admin (đối soát thủ công)
Trigger:       Webhook nhận được hoặc scheduled job giám sát
Inputs:        webhook_payload, existing_transactions
Outputs:       duplicate_alert (nếu trùng), health_alert (nếu timeout),
               manual_reconciliation_log
Business Rules: BR-1405
Acceptance Criteria:
  Given   webhook với transaction_reference "UNI-001" đã được xử lý thành công
  When    cổng thanh toán gửi lại webhook với cùng reference "UNI-001"
  Then    hệ thống SHALL KHÔNG xử lý lại (idempotent)
  And     hệ thống SHALL ghi log DUPLICATE_WEBHOOK
  And     hệ thống SHALL trả về HTTP 200 OK cho cổng thanh toán

  Given   QR code đã tạo 5 giờ trước nhưng chưa nhận webhook
  When    scheduled job giám sát chạy
  Then    hệ thống SHALL gửi cảnh báo admin "Chưa nhận webhook cho giao dịch [ref]
          sau 5 giờ. Vui lòng kiểm tra cổng thanh toán."

  Given   webhook bị thất bại nhưng admin xác nhận tiền đã vào tài khoản
  When    admin nhấn "Đối soát thủ công" và nhập bank_reference
  Then    hệ thống SHALL cập nhật ServiceFeeTransaction.status = PAID
  And     hệ thống SHALL ghi audit log "Manual reconciliation by [admin] at [time]"
Priority:      MUST
```

### FR-1406: Xử lý vi phạm ký kết hợp đồng sau hard-lock

```
ID:            FR-1406
Name:          Mở quyền tố cáo, đóng băng tạm thời và thực thi chế tài khi quá hạn ký
Description:   Hệ thống SHALL tự động mở quyền "Tố cáo đối tác" cho bên đã hoàn tất nghĩa vụ
               khi hợp đồng quá hạn ký 72 giờ mà chưa đủ 2 chữ ký. Khi nhận tố cáo hợp lệ,
               hệ thống SHALL tạm thời đóng băng tài khoản của bên bị tố cáo, gửi cảnh báo
               cuối cùng và cho phép thêm 24 giờ ân hạn để hoàn tất chữ ký. Nếu hết 24 giờ
               ân hạn mà vẫn chưa đủ 2 chữ ký, hệ thống SHALL đóng thương vụ và kích hoạt
               chế tài tương ứng.
Classification: FULLY AUTOMATED
Actor:         System, Completed Party, Reported Party
Trigger:       signing_deadline_at < now và contract.status != SIGNED; hoặc actor nhấn
               "Tố cáo đối tác" sau khi quyền tố cáo được mở
Inputs:        contract_id, deal_id, reported_by, report_reason, evidence[] (optional)
Outputs:       contract_breach_case_id, breach_case.status,
               reported_account_status = FROZEN,
               final_warning_expires_at = now + 24h,
               enforcement_status = PENDING
Business Rules: BR-1406, BR-1407, BR-1408
Acceptance Criteria:
  Given   hợp đồng đã qua 72 giờ kể từ signing_deadline_at và mới có 1 chữ ký
  When    hệ thống kiểm tra deadline
  Then    hệ thống SHALL mở quyền "Tố cáo đối tác" cho bên đã hoàn tất nghĩa vụ

  Given   bên đã hoàn tất nghĩa vụ gửi tố cáo hợp lệ
  When    hệ thống tiếp nhận report
  Then    hệ thống SHALL tạm thời đóng băng tài khoản của bên bị tố cáo
  And     hệ thống SHALL gửi cảnh báo cuối cùng và đặt final_warning_expires_at = now + 24 giờ

  Given   đã qua 24 giờ ân hạn mà contract vẫn chưa đủ 2 chữ ký
  When    hệ thống xử lý cảnh báo cuối cùng
  Then    hệ thống SHALL đóng thương vụ
  And     hệ thống SHALL kích hoạt chế tài tương ứng
Priority:      MUST
```

---

## Business Rules

```
ID:          BR-1401
Rule:        Auto-Refund kích hoạt khi PaywallSession hết hạn (48h) VÀ payment_count = 1
             (chỉ một bên đã nộp). Hệ thống PHẢI hoàn tiền 100% VỀ TÀI KHOẢN GỐC
             (xác định qua bank_reference trong webhook gốc).
             Nếu payment_count = 0: chỉ hủy deal, không cần hoàn tiền.
             Nếu payment_count = 2: paywall đã COMPLETED, không áp dụng auto-refund.
Source:      BP03 — Quy tắc 1 (Auto-Refund)
Type:        Routing + Time-based

ID:          BR-1402
Rule:        Phí dịch vụ KHÔNG ĐƯỢC HOÀN LẠI sau khi PaywallSession đạt trạng thái
             COMPLETED (2/2 thanh toán). Chính sách áp dụng TUYỆT ĐỐI, kể cả khi
             hai bên phát sinh tranh chấp, hủy hợp đồng, hoặc hủy tài trợ sau đó.
             Ngoại lệ chỉ được xử lý bởi Admin với lý do đặc biệt và phải ghi audit log.
Source:      BP03 — Quy tắc 2 (Non-refundable)
Type:        Authorization + Validation

ID:          BR-1403
Rule:        Hệ thống gửi nhắc nhở thanh toán theo lịch:
             - T-24h trước hết hạn: mức thông tin
             - T-6h trước hết hạn: mức cảnh báo
             - T-1h trước hết hạn: mức khẩn cấp
             Nhắc nhở chỉ gửi cho bên CHƯA thanh toán (status = PENDING).
             Gửi qua in-app và email. Tối đa 3 lần nhắc nhở.
Source:      [INFERRED — giảm tỷ lệ timeout và mất deal]
Type:        Time-based + Notification

ID:          BR-1404
Rule:        Mọi ServiceFeeTransaction chuyển trạng thái (PAID, REFUNDED) PHẢI được ghi
             vào PlatformRevenueLog. Log bao gồm: loại (INCOME/REFUND), số tiền, thời điểm,
             và tham chiếu giao dịch gốc. Doanh thu ròng = Σ INCOME - Σ REFUND.
Source:      [INFERRED — theo dõi doanh thu nền tảng, đối soát kế toán]
Type:        Validation + Calculation

ID:          BR-1405
Rule:        Hệ thống PHẢI xử lý webhook idempotent: cùng transaction_reference gửi
             nhiều lần SHALL chỉ kích hoạt xử lý thanh toán MỘT LẦN duy nhất.
             Webhook trùng SHALL được ghi log DUPLICATE và trả về HTTP 200 OK.
             Admin PHẢI được cảnh báo khi không nhận webhook > 4 giờ sau khi tạo QR.
             Manual reconciliation yêu cầu audit log bắt buộc.
Source:      [INFERRED — đảm bảo an toàn và chính xác dòng tiền ngoài nền tảng]
Type:        Validation + Safety

ID:          BR-1406
Rule:        Khi hợp đồng quá hạn ký 72 giờ mà chưa đủ 2 chữ ký, hệ thống PHẢI mở quyền
             tố cáo cho bên đã hoàn tất nghĩa vụ; chỉ report hợp lệ mới kích hoạt đóng băng
             tạm thời đối với tài khoản bị tố cáo.
Source:      [INFERRED — xử lý bùng ký kết sau hard-lock]
Type:        Authorization + Routing

ID:          BR-1407
Rule:        Sau khi report hợp lệ, hệ thống PHẢI gửi cảnh báo cuối cùng và cấp thêm
             24 giờ ân hạn để hoàn tất chữ ký. Trong thời gian này, chỉ tác vụ ký hợp đồng
             mới được phép mở lại; mọi hành động khác chịu policy đóng băng.
Source:      [INFERRED — final warning / grace period]
Type:        Time-based + Safety

ID:          BR-1408
Rule:        Nếu hết 24 giờ ân hạn mà contract vẫn chưa đủ 2 chữ ký, hệ thống PHẢI đóng
             thương vụ và kích hoạt chế tài tương ứng; mọi hành động tiếp theo phải bị chặn
             cho đến khi admin xử lý.
Source:      [INFERRED — enforcement sau vi phạm ký kết]
Type:        Routing + Safety
```

---

## Data Model

```
Entity:        RefundTransaction
Description:   Bản ghi hoàn tiền phí dịch vụ.
Attributes:
  - refund_id: UUID (PK)
  - original_fee_id: UUID (FK → ServiceFeeTransaction)
  - refund_amount: Decimal (required — luôn = original fee_amount)
  - currency: Enum [VND]
  - refund_reason: Enum [TIMEOUT_48H, ADMIN_EXCEPTION]
  - status: Enum [PENDING, PROCESSED, FAILED] (default: PENDING)
  - refund_to_account: String (nullable — bank_reference từ webhook gốc)
  - initiated_at: DateTime
  - processed_at: DateTime (nullable)
  - failed_at: DateTime (nullable)
  - failure_reason: Text (nullable)
  - retry_count: Integer (default: 0, max: 3)
  - admin_note: Text (nullable — chỉ cho ADMIN_EXCEPTION)
Relationships:
  - RefundTransaction —(1:1)→ ServiceFeeTransaction

Entity:        PlatformRevenueLog
Description:   Nhật ký doanh thu nền tảng từ phí dịch vụ.
Attributes:
  - log_id: UUID (PK)
  - fee_transaction_id: UUID (FK → ServiceFeeTransaction)
  - log_type: Enum [INCOME, REFUND]
  - amount: Decimal (dương cho INCOME, âm cho REFUND)
  - currency: Enum [VND]
  - logged_at: DateTime
  - deal_id: UUID (FK → Deal — tham chiếu thương vụ)
  - payer_role: Enum [ORGANIZER, SPONSOR]
  - notes: Text (nullable)
Relationships:
  - PlatformRevenueLog —(N:1)→ ServiceFeeTransaction

Entity:        WebhookAuditLog
Description:   Nhật ký tất cả webhook nhận được, bao gồm trùng lặp và không khớp.
Attributes:
  - audit_id: UUID (PK)
  - transaction_reference: String (index)
  - webhook_payload: JSON (raw payload)
  - processing_result: Enum [PROCESSED, DUPLICATE, UNMATCHED, MISMATCH, ERROR]
  - received_at: DateTime
  - processed_at: DateTime (nullable)
  - error_message: Text (nullable)
Relationships:
  - WebhookAuditLog —(standalone audit table)

Entity:        ContractBreachCase
Description:   Hồ sơ vi phạm ký kết khi quá hạn hard-lock mà chưa hoàn tất 2 chữ ký.
Attributes:
  - breach_case_id: UUID (PK)
  - contract_id: UUID (FK → Contract)
  - deal_id: UUID (FK → Deal)
  - reported_by: UUID (FK → User)
  - reported_user_id: UUID (FK → User)
  - report_reason: Text (required)
  - evidence: JSON (nullable)
  - status: Enum [OPEN, REPORTED, FROZEN, WARNING_SENT, ESCALATED, CLOSED]
  - hard_locked_at: DateTime
  - report_received_at: DateTime (nullable)
  - frozen_at: DateTime (nullable)
  - final_warning_sent_at: DateTime (nullable)
  - final_warning_expires_at: DateTime (nullable)
  - closed_at: DateTime (nullable)
  - enforcement_action: Enum [NONE, FREEZE, WARNING, CLOSE_DEAL, SANCTION]
Relationships:
  - ContractBreachCase —(N:1)→ Contract
  - ContractBreachCase —(N:1)→ Deal
```

---

## Traceability Matrix

| Process Step (BP03) | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| Quy tắc 1. Hoàn tiền tự động (Auto-Refund) | FULLY AUTOMATED | FR-1401 | BR-1401 | RefundTransaction |
| Quy tắc 2. Không hoàn phí (Non-refundable) | FULLY AUTOMATED | FR-1402 | BR-1402 | ServiceFeeTransaction |
| [INFERRED] Nhắc nhở thanh toán | FULLY AUTOMATED | FR-1403 | BR-1403 | Notification |
| [INFERRED] Ghi nhận doanh thu nền tảng | FULLY AUTOMATED + SYSTEM-SUPPORTED | FR-1404 | BR-1404 | PlatformRevenueLog |
| [INFERRED] An toàn dòng tiền (dedup, reconciliation) | FULLY AUTOMATED + SYSTEM-SUPPORTED | FR-1405 | BR-1405 | WebhookAuditLog |
