# SF-12: Service Fee Calculation & Paywall

## Overview

```
Scope:            Quản lý toàn bộ vòng đời phí dịch vụ kết nối của nền tảng — từ tính phí tự động,
                  hiển thị Bức tường thu phí (Paywall) độc lập cho mỗi bên, theo dõi trạng thái
                  thanh toán thời gian thực, mở khóa thông tin liên hệ khi hoàn tất, cho đến
                  xuất hóa đơn VAT cho Phí quản lý chiến dịch.
System Boundary:
  IN:             Tính phí dịch vụ theo cơ cấu cấu hình; Hiển thị Paywall cho từng bên;
                  Thu thập Mã số thuế doanh nghiệp; Tạo mã thanh toán QR;
                  Theo dõi trạng thái thanh toán qua webhook; Mở khóa thông tin liên hệ;
                  Kích hoạt tạo hợp đồng điện tử; Xuất hóa đơn VAT cho phí dịch vụ nền tảng.
  OUT:            Thương thảo và đồng thuận (SF-04 — đã hoàn tất, deal ở trạng thái AWAITING_PAYMENT);
                  Data Masking / Gỡ bỏ masking (SF-13 — trigger từ SF-12);
                  Soạn thảo hợp đồng (SF-05 — bắt đầu sau khi payment hoàn tất);
                  Hoàn tiền khi hết hạn (SF-14 — xử lý rủi ro thanh toán);
                  Tích hợp cổng thanh toán bên ngoài (VietQR/MoMo/ZaloPay — ngoài phạm vi hệ thống).
Assumptions:
  - [ASSUMED] Hệ thống sử dụng mô hình thanh toán ngoài nền tảng (external payment) kết hợp
    webhook xác nhận. Nền tảng không giữ tiền trung gian (escrow), không xử lý giao dịch trực tiếp.
  - [ASSUMED] Cổng thanh toán hỗ trợ giao thức webhook gửi thông báo khi hoàn tất thanh toán.
    Hệ thống đối soát dựa trên mã giao dịch (transaction reference) duy nhất.
  - [ASSUMED] Mã QR thanh toán được tạo theo chuẩn VietQR hoặc quy chuẩn cổng thanh toán liên kết.
  - [ASSUMED] Cơ cấu phí là thông số cấu hình hệ thống, Admin có thể điều chỉnh phần trăm
    và mức trần phí cho cả hai bên (CLB và Doanh nghiệp).
  - [ASSUMED] Hệ thống xuất hóa đơn VAT điện tử CHỈ cho khoản Phí quản lý chiến dịch thu từ
    Doanh nghiệp — tuyệt đối không xuất hóa đơn cho giá trị gói tài trợ.
Gaps Detected:
  - BP03 nêu tỷ lệ phí doanh nghiệp là "Tính theo % giá trị hợp đồng (Có mức trần tối đa)"
    nhưng không chỉ rõ tỷ lệ cụ thể → đã xác nhận: 2-5%, trần 3.000.000 VNĐ, admin cấu hình.
  - BP03 nêu "Kế toán thực hiện thao tác" cho xuất hóa đơn → đề xuất tự động hóa hoàn toàn
    qua API, với fallback manual nếu cần.
  - Không nêu cơ chế đối soát thanh toán khi webhook thất bại → cần bổ sung retry + manual reconciliation.
  - Không nêu cơ chế phí xem trước (fee preview) trước khi lock-in → bổ sung FR-1202.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Tài khoản đại diện Ban tổ chức (CLB) | `organizer` | Xem Paywall phí CLB; Quét mã QR thanh toán; Xem trạng thái thanh toán |
| Tài khoản đại diện Doanh nghiệp | `sponsor` | Xem Paywall phí DN; Nhập Mã số thuế; Quét mã QR thanh toán; Xem trạng thái thanh toán |
| Hệ thống | `system` | Tính phí; Tạo mã QR; Nhận webhook; Cập nhật trạng thái; Mở khóa thông tin; Kích hoạt hợp đồng; Xuất hóa đơn VAT |
| Admin | `admin` | Cấu hình thông số phí (phần trăm, mức trần) cho cả hai bên |
| Cổng thanh toán (External) | `payment_gateway` | Nhận thanh toán; Gửi webhook xác nhận về hệ thống |

---

## Functional Requirements

### FR-1201: Tính phí dịch vụ kết nối theo cơ cấu cấu hình

```
ID:            FR-1201
Name:          Tự động tính phí dịch vụ kết nối cho cả hai bên
Description:   Hệ thống SHALL tự động tính phí dịch vụ cho CLB và Doanh nghiệp khi deal
               chuyển sang trạng thái AWAITING_PAYMENT (sau khi hai bên đồng thuận lock-in).
               Phí được tính dựa trên cơ cấu phí cấu hình bởi Admin:
               - CLB (Phí duy trì nền tảng):
                 + Tài trợ tiền mặt: fee_percentage_organizer_cash % × giá trị hợp đồng
                   (Tối đa: fee_cap_organizer_cash VNĐ). Mặc định: 1%, trần 100.000 VNĐ.
                 + Tài trợ hiện vật: phí cố định fee_fixed_organizer_inkind VNĐ.
                   Mặc định: 50.000 VNĐ.
               - Doanh nghiệp (Phí quản lý chiến dịch):
                 + Tài trợ tiền mặt: fee_percentage_sponsor_cash % × giá trị hợp đồng
                   (Tối đa: fee_cap_sponsor_cash VNĐ). Mặc định: 2-5%, trần 3.000.000 VNĐ.
                 + Tài trợ hiện vật: phí cố định fee_fixed_sponsor_inkind VNĐ.
                   Mặc định: 500.000 VNĐ.
               Hệ thống SHALL tạo bản ghi ServiceFeeTransaction cho mỗi bên.
Classification: FULLY AUTOMATED
Actor:         System (khởi tạo)
Trigger:       Deal chuyển sang trạng thái AWAITING_PAYMENT (từ SF-04, FR-0406 sửa đổi)
Inputs:        deal_id, sponsorship_type (CASH | IN_KIND | COMBINED),
               sponsorship_value (decimal — nếu CASH hoặc COMBINED),
               fee_configuration (từ FeeConfiguration entity)
Outputs:       organizer_fee_transaction: { fee_id, amount, currency, status = PENDING },
               sponsor_fee_transaction: { fee_id, amount, currency, status = PENDING },
               paywall_session_id (UUID), paywall_expires_at (timestamp = now + 48h)
Business Rules: BR-1201, BR-1202
Acceptance Criteria:
  Given   deal "deal-001" vừa lock-in với sponsorship_type = CASH, value = 20.000.000 VNĐ
  And     fee config: organizer_cash = 1% (cap 100K), sponsor_cash = 3% (cap 3M)
  When    hệ thống tính phí
  Then    organizer_fee = min(20.000.000 × 1%, 100.000) = 100.000 VNĐ
  And     sponsor_fee = min(20.000.000 × 3%, 3.000.000) = 600.000 VNĐ
  And     hệ thống SHALL tạo PaywallSession với expires_at = now + 48h

  Given   deal "deal-002" lock-in với sponsorship_type = IN_KIND
  When    hệ thống tính phí
  Then    organizer_fee = 50.000 VNĐ (cố định)
  And     sponsor_fee = 500.000 VNĐ (cố định)

  Given   deal "deal-003" lock-in với sponsorship_type = COMBINED,
          cash_value = 50.000.000, in_kind phần hiện vật
  When    hệ thống tính phí
  Then    organizer_fee = min(50.000.000 × 1%, 100.000) + 50.000 = 100.000 + 50.000 = 150.000 VNĐ
  And     sponsor_fee = min(50.000.000 × 3%, 3.000.000) + 500.000 = 1.500.000 + 500.000 = 2.000.000 VNĐ
Priority:      MUST
```

### FR-1202: Hiển thị Bức tường thu phí (Paywall) và Xem trước phí

```
ID:            FR-1202
Name:          Hiển thị Paywall độc lập cho mỗi bên và hỗ trợ xem trước phí
Description:   Hệ thống SHALL hiển thị Bức tường thu phí (Paywall) ĐỘC LẬP cho từng bên
               khi deal ở trạng thái AWAITING_PAYMENT. Mỗi bên chỉ thấy khoản phí của mình,
               không thấy khoản phí của đối tác. Paywall hiển thị: số tiền phí, mã QR thanh toán,
               thời hạn thanh toán (đếm ngược 48 giờ), và trạng thái thanh toán tổng thể
               (0/2, 1/2, 2/2). Ngoài ra, hệ thống SHALL hỗ trợ xem trước phí ước tính
               (Fee Preview) trong quá trình thương thảo để tránh bất ngờ về chi phí.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Actor truy cập trang deal đang ở trạng thái AWAITING_PAYMENT,
               hoặc xem ước tính phí trong quá trình thương thảo (preview mode)
Inputs:        deal_id, actor_id (để xác định bên nào)
               Hoặc: sponsorship_type + sponsorship_value (preview mode)
Outputs:       PaywallView: { my_fee_amount, qr_code_url, deadline_countdown,
               payment_status_overall (0/2 | 1/2 | 2/2), my_payment_status (PENDING | PAID) }
               Hoặc: FeePreview: { estimated_organizer_fee, estimated_sponsor_fee }
Business Rules: BR-1202, BR-1203
Acceptance Criteria:
  Given   deal ở trạng thái AWAITING_PAYMENT
  And     organizer truy cập trang deal
  When    hệ thống hiển thị Paywall
  Then    hệ thống SHALL hiển thị organizer_fee = 100.000 VNĐ
  And     hệ thống SHALL hiển thị mã QR thanh toán
  And     hệ thống SHALL hiển thị đếm ngược "Còn 47 giờ 30 phút"
  And     hệ thống SHALL hiển thị trạng thái "0/2 Đã thanh toán"
  But     hệ thống SHALL KHÔNG hiển thị khoản phí của sponsor

  Given   hai bên đang thương thảo (deal IN_PROGRESS)
  When    actor xem "Ước tính phí dịch vụ"
  Then    hệ thống SHALL hiển thị phí ước tính dựa trên giá trị thỏa thuận hiện tại
Priority:      MUST
```

### FR-1203: Thu thập thông tin Mã số thuế Doanh nghiệp

```
ID:            FR-1203
Name:          Yêu cầu nhập Mã số thuế trước thanh toán (Doanh nghiệp)
Description:   Hệ thống SHALL yêu cầu Doanh nghiệp nhập thông tin Mã số thuế (MST) trước khi
               cho phép thanh toán phí dịch vụ. Thông tin MST được sử dụng để chuẩn bị
               xuất hóa đơn VAT cho Phí quản lý chiến dịch. Hệ thống SHALL xác thực
               định dạng MST (10 hoặc 13 chữ số).
Classification: SYSTEM-SUPPORTED
Actor:         Sponsor
Trigger:       Sponsor truy cập Paywall lần đầu hoặc nhấn "Cập nhật MST"
Inputs:        deal_id, tax_code (string), business_name (string),
               business_address (string)
Outputs:       fee_transaction.tax_info_captured = true,
               nút thanh toán được mở khóa
Business Rules: BR-1204
Acceptance Criteria:
  Given   sponsor truy cập Paywall lần đầu và chưa nhập MST
  When    hệ thống hiển thị Paywall
  Then    nút "Thanh toán" SHALL bị vô hiệu hóa
  And     hệ thống SHALL hiển thị form nhập MST

  Given   sponsor nhập MST = "0312345678" (10 chữ số hợp lệ)
  When    sponsor nhấn "Xác nhận"
  Then    hệ thống SHALL lưu thông tin thuế
  And     hệ thống SHALL mở khóa nút "Thanh toán"

  Given   sponsor nhập MST = "ABC123"
  When    sponsor nhấn "Xác nhận"
  Then    hệ thống SHALL từ chối "Mã số thuế phải là 10 hoặc 13 chữ số"
Priority:      MUST
```

### FR-1204: Tạo mã thanh toán QR

```
ID:            FR-1204
Name:          Tạo mã QR thanh toán cho phí dịch vụ
Description:   Hệ thống SHALL tạo mã QR thanh toán cho mỗi bên dựa trên số tiền phí đã tính.
               Mã QR SHALL chứa đầy đủ thông tin: số tài khoản thụ hưởng của nền tảng,
               số tiền chính xác, và mã giao dịch tham chiếu (transaction_reference) duy nhất
               để đối soát thanh toán. Mã QR tuân theo chuẩn VietQR hoặc quy chuẩn cổng
               thanh toán liên kết.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       PaywallSession được tạo thành công (từ FR-1201)
Inputs:        fee_transaction_id, amount, platform_bank_account,
               transaction_reference (auto-generated)
Outputs:       qr_code_url (string), qr_code_image (binary),
               transaction_reference (string — unique identifier)
Business Rules: BR-1205
Acceptance Criteria:
  Given   PaywallSession mới được tạo cho deal "deal-001"
  When    hệ thống tạo mã QR cho organizer_fee = 100.000 VNĐ
  Then    hệ thống SHALL tạo mã QR chứa đúng số tiền 100.000 VNĐ
  And     mã QR SHALL chứa transaction_reference duy nhất (ví dụ: "UNI-FEE-20260501-001")
  And     mã QR SHALL hiển thị trên Paywall của organizer

  Given   mã QR đã tạo cho bên organizer
  When    hệ thống tạo mã QR cho sponsor_fee
  Then    sponsor_qr SHALL có transaction_reference KHÁC organizer_qr
Priority:      MUST
```

### FR-1205: Theo dõi trạng thái thanh toán thời gian thực

```
ID:            FR-1205
Name:          Nhận và xử lý webhook xác nhận thanh toán
Description:   Hệ thống SHALL nhận webhook từ cổng thanh toán bên ngoài để cập nhật trạng thái
               thanh toán của mỗi bên. Hệ thống SHALL đối soát dựa trên transaction_reference
               và số tiền chính xác. Khi xác nhận thanh toán thành công, hệ thống SHALL cập nhật
               ServiceFeeTransaction sang PAID và cập nhật PaywallSession.payment_status
               (0/2 → 1/2, hoặc 1/2 → 2/2). Hệ thống SHALL hiển thị trạng thái thanh toán
               theo thời gian thực cho cả hai bên.
Classification: FULLY AUTOMATED
Actor:         System (nhận webhook), Payment Gateway (gửi webhook)
Trigger:       Cổng thanh toán gửi webhook xác nhận giao dịch thành công
Inputs:        webhook_payload: { transaction_reference, amount, paid_at, bank_reference }
Outputs:       fee_transaction.status = PAID, fee_transaction.paid_at,
               paywall_session.payment_count (0 → 1 hoặc 1 → 2),
               real-time notification cho cả hai bên
Business Rules: BR-1205, BR-1206
Acceptance Criteria:
  Given   organizer đã quét QR và chuyển khoản 100.000 VNĐ
  When    cổng thanh toán gửi webhook với transaction_reference = "UNI-FEE-20260501-001"
          và amount = 100.000
  Then    hệ thống SHALL cập nhật organizer_fee.status = PAID
  And     hệ thống SHALL cập nhật paywall.payment_count = 1
  And     hệ thống SHALL hiển thị "1/2 Đã thanh toán" cho cả hai bên
  And     hệ thống SHALL gửi thông báo cho sponsor "Đối tác đã hoàn tất thanh toán phí"

  Given   webhook nhận được với amount = 99.000 (không khớp)
  When    hệ thống đối soát
  Then    hệ thống SHALL đánh dấu giao dịch MISMATCH
  And     hệ thống SHALL ghi log để admin đối soát thủ công
  And     hệ thống SHALL KHÔNG cập nhật trạng thái PAID

  Given   webhook nhận được với transaction_reference không tồn tại
  When    hệ thống đối soát
  Then    hệ thống SHALL từ chối và ghi log UNMATCHED_WEBHOOK
Priority:      MUST
```

### FR-1206: Mở khóa thông tin liên hệ khi 2/2 thanh toán

```
ID:            FR-1206
Name:          Tự động mở khóa thông tin liên hệ khi cả hai bên hoàn tất thanh toán
Description:   Hệ thống SHALL tự động kích hoạt gỡ bỏ Data Masking (→ SF-13 FR-1302) ngay khi
               PaywallSession.payment_count đạt 2/2. Sau khi mở khóa, thông tin liên hệ thật
               của hai bên (số điện thoại, email, link mạng xã hội) SHALL được hiển thị công khai
               trong deal context. Hệ thống SHALL ghi nhận thời điểm mở khóa.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       paywall_session.payment_count chuyển từ 1 sang 2 (2/2 hoàn tất)
Inputs:        paywall_session_id, deal_id
Outputs:       deal.contact_unlocked = true, deal.unlocked_at (timestamp),
               trigger SF-13 FR-1302 (gỡ Data Masking),
               notification cho cả hai bên "Thông tin liên hệ đã được mở khóa"
Business Rules: BR-1206
Acceptance Criteria:
  Given   paywall_session có payment_count = 1/2 (organizer đã trả)
  When    sponsor hoàn tất thanh toán (payment_count → 2/2)
  Then    hệ thống SHALL gỡ bỏ Data Masking cho deal này (→ SF-13)
  And     hệ thống SHALL hiển thị thông tin liên hệ thật
  And     hệ thống SHALL ghi nhận unlocked_at
  And     hệ thống SHALL thông báo "Thông tin liên hệ đã mở khóa. Sẵn sàng ký hợp đồng."
Priority:      MUST
```

### FR-1207: Kích hoạt tạo hợp đồng điện tử sau thanh toán

```
ID:            FR-1207
Name:          Tự động chuyển deal sang AGREED và kích hoạt tạo hợp đồng
Description:   Hệ thống SHALL tự động chuyển deal từ trạng thái AWAITING_PAYMENT sang AGREED
               ngay khi PaywallSession hoàn tất (2/2). Sự kiện AGREED kích hoạt SF-05 FR-0501
               tạo bản nháp hợp đồng điện tử tự động và mở giai đoạn ký kết hard-lock 72 giờ.
               Quy trình liên tiếp:
               2/2 payment → Unlock contacts → Deal = AGREED → Auto-create contract (SF-05)
               → Start 72h signing countdown.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       paywall_session.status = COMPLETED (2/2)
Inputs:        paywall_session_id, deal_id
Outputs:       deal.status = AGREED, deal.agreed_at (timestamp),
               trigger SF-05 FR-0501 (tạo hợp đồng),
               contract.signing_window_started_at = agreed_at,
               contract.signing_deadline_at = agreed_at + 72h
Business Rules: BR-1207
Acceptance Criteria:
  Given   paywall_session vừa hoàn tất 2/2
  When    hệ thống xử lý sự kiện
  Then    hệ thống SHALL chuyển deal sang AGREED
  And     hệ thống SHALL kích hoạt tạo hợp đồng (SF-05)
  And     hệ thống SHALL khởi tạo đếm ngược 72 giờ cho giai đoạn ký kết
  And     hệ thống SHALL thông báo "Thanh toán hoàn tất. Hợp đồng đã sẵn sàng để soạn thảo."
Priority:      MUST
```

### FR-1208: Xuất hóa đơn VAT cho Phí quản lý chiến dịch

```
ID:            FR-1208
Name:          Tạo và xuất hóa đơn VAT điện tử cho Phí quản lý chiến dịch
Description:   Hệ thống SHALL tự động (hoặc qua kế toán) tạo hóa đơn VAT điện tử CHỈ cho khoản
               Phí quản lý chiến dịch thu từ Doanh nghiệp. Hóa đơn bao gồm: tên và MST doanh
               nghiệp (từ FR-1203), nội dung dịch vụ = "Phí quản lý chiến dịch kết nối tài trợ",
               giá trị trước thuế, thuế suất VAT, tổng giá trị. Hóa đơn được gửi qua email
               cho Doanh nghiệp. Nền tảng TUYỆT ĐỐI KHÔNG xuất hóa đơn cho giá trị gói tài trợ.
Classification: SYSTEM-SUPPORTED (kế toán xác nhận) / FULLY AUTOMATED (qua API e-invoice)
Actor:         System (tạo hóa đơn), Sponsor (nhận hóa đơn)
Trigger:       PaywallSession hoàn tất (2/2) VÀ sponsor_fee.status = PAID
Inputs:        sponsor_fee_transaction_id, tax_code, business_name, business_address,
               fee_amount (phí quản lý chiến dịch)
Outputs:       platform_invoice_id (UUID), invoice_number (sequential),
               invoice_pdf (binary), sent_to_email (string)
Business Rules: BR-1207, BR-1208
Acceptance Criteria:
  Given   sponsor đã thanh toán phí dịch vụ 600.000 VNĐ và paywall 2/2 hoàn tất
  When    hệ thống xử lý phát hành hóa đơn
  Then    hệ thống SHALL tạo hóa đơn VAT với nội dung "Phí quản lý chiến dịch kết nối tài trợ"
  And     hệ thống SHALL gửi hóa đơn PDF qua email cho doanh nghiệp
  And     hóa đơn SHALL chứa MST, tên DN, giá trị trước thuế, VAT 10%, tổng giá trị

  Given   organizer đã thanh toán phí 100.000 VNĐ
  When    hệ thống kiểm tra xuất hóa đơn
  Then    hệ thống SHALL KHÔNG xuất hóa đơn VAT cho khoản phí CLB
         (CLB là tổ chức sinh viên, không cần hóa đơn VAT)

  Given   giá trị gói tài trợ là 20.000.000 VNĐ
  When    bất kỳ actor yêu cầu hóa đơn cho giá trị tài trợ
  Then    hệ thống SHALL từ chối — nền tảng không xuất hóa đơn cho giá trị tài trợ
Priority:      MUST
```

---

## Business Rules

```
ID:          BR-1201
Rule:        Cơ cấu phí dịch vụ là THÔNG SỐ CẤU HÌNH HỆ THỐNG, Admin có thể điều chỉnh.
             Giá trị mặc định:
             - CLB — Tài trợ tiền mặt: 1% giá trị hợp đồng, tối đa 100.000 VNĐ.
             - CLB — Tài trợ hiện vật: cố định 50.000 VNĐ.
             - Doanh nghiệp — Tài trợ tiền mặt: 2-5% giá trị hợp đồng, tối đa 3.000.000 VNĐ.
             - Doanh nghiệp — Tài trợ hiện vật: cố định 500.000 VNĐ.
             Với tài trợ kết hợp (COMBINED): phí = phí_tiền_mặt + phí_hiện_vật.
Source:      BP03 — Bảng Cơ cấu Thu phí + Xác nhận từ stakeholder
Type:        Calculation

ID:          BR-1202
Rule:        Paywall hiển thị ĐỘC LẬP cho mỗi bên. CLB chỉ thấy phí CLB, DN chỉ thấy phí DN.
             KHÔNG bên nào được thấy khoản phí cụ thể của đối tác.
             Cả hai bên đều thấy trạng thái thanh toán tổng thể (0/2, 1/2, 2/2).
Source:      BP03 — Bước 2 (Paywall độc lập cho từng bên)
Type:        Authorization + Display

ID:          BR-1203
Rule:        Hai bên có tối đa 48 GIỜ kể từ khi deal chuyển sang AWAITING_PAYMENT
             để hoàn tất thanh toán phí dịch vụ. Thời hạn bắt đầu từ thời điểm lock-in
             và ÁP DỤNG CHUNG cho cả hai bên (không tính riêng).
Source:      BP03 — Bước 2 (48 giờ); BP03 — Sec 1 (Double Handshake)
Type:        Time-based

ID:          BR-1204
Rule:        Doanh nghiệp PHẢI nhập Mã số thuế (MST) hợp lệ (10 hoặc 13 chữ số) trước khi
             được phép thanh toán. MST được sử dụng để xuất hóa đơn VAT cho Phí quản lý
             chiến dịch. CLB KHÔNG cần nhập MST.
Source:      BP03 — Bước 2 (DN nhập MST để chuẩn bị xuất hóa đơn VAT)
Type:        Validation

ID:          BR-1205
Rule:        Hệ thống đối soát thanh toán dựa trên transaction_reference DUY NHẤT
             và số tiền CHÍNH XÁC (exact match). Webhook không khớp (sai reference hoặc sai
             số tiền) SHALL được ghi log để đối soát thủ công. Hệ thống SHALL hỗ trợ cơ chế
             retry webhook (tối đa 3 lần) và manual reconciliation bởi admin khi webhook
             thất bại.
Source:      [INFERRED — đảm bảo an toàn dòng tiền thanh toán ngoài nền tảng]
Type:        Validation + Safety

ID:          BR-1206
Rule:        Thông tin liên hệ CHỈ được mở khóa khi VÀ CHỈ KHI cả hai bên đều đã thanh toán
             (payment_count = 2/2). Thanh toán 1 bên (1/2) KHÔNG kích hoạt mở khóa.
Source:      BP03 — Bước 3 (Ngay khi 2/2 hoàn tất, Data Masking được gỡ bỏ)
Type:        Routing

ID:          BR-1207
Rule:        Hóa đơn VAT điện tử CHỈ được xuất cho khoản Phí quản lý chiến dịch thu từ
             Doanh nghiệp. Nền tảng TUYỆT ĐỐI KHÔNG xuất hóa đơn cho giá trị gói tài trợ.
             Mỗi ServiceFeeTransaction (sponsor) chỉ có tối đa MỘT hóa đơn VAT.
Source:      BP03 — Sec 1 (Hóa đơn VAT); Xác nhận từ stakeholder
Type:        Validation

ID:          BR-1208
Rule:        Cấu hình phí dịch vụ CHỈ Admin mới có quyền thay đổi. Mọi thay đổi PHẢI được
             ghi audit log (ai đổi, khi nào, giá trị cũ, giá trị mới). Thay đổi cấu hình
             CHỈ áp dụng cho các deal MỚI, KHÔNG ảnh hưởng deal đã lock-in.
Source:      [INFERRED — bảo vệ tính nhất quán phí và audit trail]
Type:        Authorization + Validation
```

---

## Data Model

```
Entity:        FeeConfiguration
Description:   Cấu hình thông số phí dịch vụ, Admin có thể điều chỉnh.
Attributes:
  - config_id: UUID (PK)
  - fee_percentage_organizer_cash: Decimal (default: 0.01 — 1%)
  - fee_cap_organizer_cash: Decimal (default: 100000 — 100K VNĐ)
  - fee_fixed_organizer_inkind: Decimal (default: 50000 — 50K VNĐ)
  - fee_percentage_sponsor_cash: Decimal (default: 0.03 — 3%)
  - fee_cap_sponsor_cash: Decimal (default: 3000000 — 3M VNĐ)
  - fee_fixed_sponsor_inkind: Decimal (default: 500000 — 500K VNĐ)
  - effective_from: DateTime
  - updated_at: DateTime
  - updated_by: UUID (FK → Admin)
Relationships:
  - FeeConfiguration —(singleton, versioned)→ System

Entity:        PaywallSession
Description:   Phiên thanh toán 48 giờ cho một deal, theo dõi trạng thái hai bên.
Attributes:
  - session_id: UUID (PK)
  - deal_id: UUID (FK → Deal, UNIQUE)
  - status: Enum [ACTIVE, COMPLETED, EXPIRED, CANCELLED] (default: ACTIVE)
  - payment_count: Integer (0, 1, 2) (default: 0)
  - started_at: DateTime
  - expires_at: DateTime (= started_at + 48h)
  - completed_at: DateTime (nullable — khi 2/2)
  - expired_at: DateTime (nullable — khi hết hạn)
Relationships:
  - PaywallSession —(1:1)→ Deal
  - PaywallSession —(1:2)→ ServiceFeeTransaction

Entity:        ServiceFeeTransaction
Description:   Bản ghi phí dịch vụ cho MỘT bên (organizer hoặc sponsor).
Attributes:
  - fee_id: UUID (PK)
  - session_id: UUID (FK → PaywallSession)
  - deal_id: UUID (FK → Deal)
  - payer_role: Enum [ORGANIZER, SPONSOR]
  - payer_id: UUID (FK → User)
  - fee_type: Enum [PLATFORM_MAINTENANCE, CAMPAIGN_MANAGEMENT]
  - sponsorship_type: Enum [CASH, IN_KIND, COMBINED]
  - contract_value: Decimal (nullable — giá trị hợp đồng tiền mặt, nếu có)
  - fee_amount: Decimal (required — số tiền phí đã tính)
  - currency: Enum [VND] (default: VND)
  - transaction_reference: String (unique — mã đối soát thanh toán)
  - qr_code_url: String (nullable)
  - status: Enum [PENDING, PAID, REFUNDED, MISMATCH, EXPIRED] (default: PENDING)
  - tax_code: String (nullable — chỉ sponsor)
  - business_name: String (nullable — chỉ sponsor)
  - business_address: String (nullable — chỉ sponsor)
  - paid_at: DateTime (nullable)
  - webhook_received_at: DateTime (nullable)
  - webhook_payload: JSON (nullable — raw webhook data cho audit)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - ServiceFeeTransaction —(N:1)→ PaywallSession
  - ServiceFeeTransaction —(N:1)→ Deal
  - ServiceFeeTransaction —(0..1:1)→ PlatformVATInvoice (chỉ sponsor)
  - ServiceFeeTransaction —(0..1:1)→ RefundTransaction (SF-14, nếu có hoàn tiền)

Entity:        PlatformVATInvoice
Description:   Hóa đơn VAT cho Phí quản lý chiến dịch — CHỈ cho Doanh nghiệp.
               Tách biệt hoàn toàn với hóa đơn giá trị tài trợ (đã bị loại bỏ khỏi SF-05).
Attributes:
  - invoice_id: UUID (PK)
  - fee_transaction_id: UUID (FK → ServiceFeeTransaction, UNIQUE)
  - invoice_number: String (auto-generated, sequential)
  - business_name: String (required)
  - tax_code: String (required, 10 or 13 digits)
  - business_address: String (required)
  - service_description: String (default: "Phí quản lý chiến dịch kết nối tài trợ")
  - amount_before_tax: Decimal
  - vat_rate: Decimal (default: 0.10 — 10%)
  - vat_amount: Decimal (calculated)
  - total_amount: Decimal (calculated)
  - issued_at: DateTime
  - sent_to_email: String
  - invoice_pdf_url: String (nullable)
Relationships:
  - PlatformVATInvoice —(1:1)→ ServiceFeeTransaction
```

---

## Traceability Matrix

| Process Step (BP03) | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| Sec 1. Cơ cấu thu phí (Split Fee) | FULLY AUTOMATED | FR-1201 | BR-1201 | FeeConfiguration, ServiceFeeTransaction |
| Bước 2. Hiển thị Paywall độc lập | SYSTEM-SUPPORTED | FR-1202 | BR-1202, BR-1203 | PaywallSession |
| Bước 2. DN nhập Mã số thuế | SYSTEM-SUPPORTED | FR-1203 | BR-1204 | ServiceFeeTransaction |
| Bước 2. Tạo mã QR thanh toán | FULLY AUTOMATED | FR-1204 | BR-1205 | ServiceFeeTransaction |
| Bước 2. Theo dõi trạng thái thanh toán | FULLY AUTOMATED | FR-1205 | BR-1205, BR-1206 | ServiceFeeTransaction, PaywallSession |
| Bước 3. Mở khóa thông tin liên hệ | FULLY AUTOMATED | FR-1206 | BR-1206 | PaywallSession, Deal |
| Bước 3. Kích hoạt tạo hợp đồng và đếm ngược ký kết | FULLY AUTOMATED | FR-1207 | BR-1207 | Deal, Contract (SF-05) |
| Bước 3. Xuất hóa đơn VAT (phí dịch vụ) | SYSTEM-SUPPORTED / FULLY AUTOMATED | FR-1208 | BR-1207, BR-1208 | PlatformVATInvoice |
| [INFERRED] Cấu hình phí bởi Admin | SYSTEM-SUPPORTED | FR-1201 (config) | BR-1208 | FeeConfiguration |
| [INFERRED] Xem trước phí ước tính | SYSTEM-SUPPORTED | FR-1202 (preview) | BR-1201 | — |
