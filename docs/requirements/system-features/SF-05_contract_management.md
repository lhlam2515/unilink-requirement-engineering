# SF-05: Contract Management

## Overview

```
Scope:            Quản lý toàn bộ vòng đời hợp đồng tài trợ — từ soạn thảo, chỉnh sửa nội dung,
                  xác nhận song phương, ký chữ ký điện tử, xuất tài liệu PDF, đến phát hành
                  hóa đơn VAT cho giao dịch tiền mặt.
System Boundary:
  IN:             Tạo bản nháp hợp đồng từ deal đã đồng thuận; Nhập/chỉnh sửa các điều khoản;
                  Xác nhận nội dung; Ký chữ ký điện tử; Xuất PDF; Phát hành hóa đơn VAT.
  OUT:            Thương thảo (SF-04 — đã hoàn tất trước khi bắt đầu);
                  Thực hiện nghĩa vụ (SF-06 — bắt đầu sau khi ký kết).
Assumptions:
  - [ASSUMED] Hệ thống sử dụng chữ ký điện tử đơn giản (drawn/typed signature) chứ không phải
    chữ ký số mã hóa (digital signature with PKI) ở phiên bản đầu.
  - [ASSUMED] Hợp đồng được tạo từ template có sẵn, điền thông tin từ deal context.
  - [ASSUMED] Hóa đơn VAT (hóa đơn đỏ) được tạo bởi hệ thống dựa trên yêu cầu của doanh nghiệp.
Gaps Detected:
  - Quy trình gốc không nêu rõ quy trình hủy hợp đồng sau khi ký → cần bổ sung.
  - Không nêu phiên bản (versioning) cho hợp đồng trong quá trình chỉnh sửa.
  - Không nêu rõ hóa đơn VAT tuân theo quy định nào cụ thể → đánh dấu ASSUMED.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Ban tổ chức (BTC) | `organizer` | Soạn thảo, chỉnh sửa, xác nhận, ký chữ ký điện tử |
| Doanh nghiệp | `sponsor` | Soạn thảo, chỉnh sửa, xác nhận, ký chữ ký điện tử, yêu cầu hóa đơn VAT |
| Hệ thống | `system` | Tạo bản nháp từ template, xác thực nội dung, tạo PDF, tạo hóa đơn, ghi log |

---

## Functional Requirements

### FR-0501: Tạo bản nháp hợp đồng từ deal

```
ID:            FR-0501
Name:          Khởi tạo bản nháp hợp đồng tài trợ
Description:   Hệ thống SHALL tự động tạo bản nháp hợp đồng khi deal chuyển sang trạng thái AGREED.
               Bản nháp SHALL được điền sẵn thông tin từ deal context: thông tin BTC, thông tin
               doanh nghiệp, thông tin sự kiện, gói tài trợ đã thảo luận. Hệ thống SHALL sử dụng
               template hợp đồng chuẩn của nền tảng.
Classification: FULLY AUTOMATED (khởi tạo) → SYSTEM-SUPPORTED (chỉnh sửa)
Actor:         System (khởi tạo), Organizer & Sponsor (chỉnh sửa)
Trigger:       Deal chuyển sang trạng thái AGREED (từ SF-04, FR-0406)
Inputs:        deal_id, proposal_id, organizer_info, sponsor_info, agreed_terms (từ deal notes)
Outputs:       contract_id (UUID), status = DRAFTING, pre-populated contract fields
Business Rules: BR-0501
Acceptance Criteria:
  Given   deal "deal-001" vừa chuyển sang trạng thái AGREED
  When    hệ thống xử lý sự kiện
  Then    hệ thống SHALL tạo contract mới với trạng thái DRAFTING
  And     hệ thống SHALL điền sẵn thông tin BTC, doanh nghiệp, và sự kiện từ deal
  And     hệ thống SHALL thông báo cho cả hai bên "Hợp đồng đã sẵn sàng để soạn thảo"
Priority:      MUST
```

### FR-0502: Chỉnh sửa nội dung hợp đồng

```
ID:            FR-0502
Name:          Chỉnh sửa các điều khoản hợp đồng tài trợ
Description:   Hệ thống SHALL cho phép cả hai bên chỉnh sửa các mục trong hợp đồng:
               - Thông tin nhà tài trợ và ban tổ chức
               - Ngày ký kết và thời gian hiệu lực
               - Thời hạn thực hiện giao dịch
               - Hình thức tài trợ, giá trị tài trợ, lợi ích tài trợ
               - Quyền lợi của hai bên
               - Cam kết và trách nhiệm của hai bên
               Hệ thống SHALL lưu lịch sử chỉnh sửa (ai, khi nào, thay đổi gì).
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Actor chỉnh sửa nội dung trong trang soạn thảo hợp đồng
Inputs:        contract_id, section (enum), updated_content (text/structured),
               edited_by (UUID)
Outputs:       Contract đã cập nhật, edit_log entry, updated_at, version_number
Business Rules: BR-0502, BR-0503
Acceptance Criteria:
  Given   contract ở trạng thái DRAFTING
  When    organizer cập nhật giá trị tài trợ từ 20,000,000 VND sang 25,000,000 VND
  Then    hệ thống SHALL lưu thay đổi
  And     hệ thống SHALL ghi log: "organizer thay đổi sponsorship_value: 20M → 25M vào [timestamp]"
  And     hệ thống SHALL tăng version_number

  Given   contract ở trạng thái SIGNED
  When    actor cố chỉnh sửa nội dung
  Then    hệ thống SHALL từ chối "Hợp đồng đã ký không thể chỉnh sửa"
Priority:      MUST
```

### FR-0503: Xác nhận nội dung hợp đồng song phương

```
ID:            FR-0503
Name:          Xác nhận nội dung hợp đồng bởi cả hai bên
Description:   Hệ thống SHALL yêu cầu CẢ HAI bên xác nhận nội dung hợp đồng trước khi
               cho phép ký. Khi một bên xác nhận, bên còn lại được thông báo. Khi CẢ HAI
               đã xác nhận, hệ thống SHALL chuyển contract sang trạng thái CONFIRMED
               và mở khóa tính năng ký chữ ký điện tử. Nếu một bên chỉnh sửa sau khi
               đã xác nhận → tất cả xác nhận bị reset.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer (xác nhận), Sponsor (xác nhận)
Trigger:       Actor nhấn "Xác nhận nội dung hợp đồng"
Inputs:        contract_id, confirmed_by (UUID)
Outputs:       contract.organizer_content_confirmed (boolean),
               contract.sponsor_content_confirmed (boolean),
               contract.status = CONFIRMED (khi cả hai xác nhận)
Business Rules: BR-0504
Acceptance Criteria:
  Given   contract ở trạng thái DRAFTING
  When    organizer nhấn "Xác nhận nội dung"
  Then    hệ thống SHALL ghi nhận organizer_content_confirmed = true
  And     hệ thống SHALL thông báo cho sponsor

  Given   organizer_content_confirmed = true VÀ sponsor_content_confirmed = true
  When    hệ thống kiểm tra
  Then    hệ thống SHALL chuyển contract sang trạng thái CONFIRMED
  And     hệ thống SHALL mở khóa nút "Ký chữ ký điện tử"

  Given   organizer đã xác nhận nội dung
  When    sponsor chỉnh sửa một điều khoản
  Then    hệ thống SHALL reset organizer_content_confirmed = false
  And     hệ thống SHALL thông báo "Nội dung đã thay đổi, cần xác nhận lại"
Priority:      MUST
```

### FR-0504: Ký chữ ký điện tử

```
ID:            FR-0504
Name:          Ký hợp đồng bằng chữ ký điện tử
Description:   Hệ thống SHALL cho phép mỗi bên ký chữ ký điện tử lên hợp đồng đã CONFIRMED.
               Hệ thống hỗ trợ chữ ký vẽ tay (drawn) hoặc gõ tên (typed). Khi CẢ HAI bên
               đã ký, hệ thống SHALL chuyển contract sang trạng thái SIGNED, ghi nhận thời điểm
               ký của mỗi bên, và tự động tạo nghĩa vụ tài trợ (liên kết SF-06).
Classification: SYSTEM-SUPPORTED
Actor:         Organizer (ký), Sponsor (ký)
Trigger:       Actor nhấn "Ký hợp đồng" trên trang hợp đồng đã xác nhận
Inputs:        contract_id, signer_id (UUID), signature_type (enum: DRAWN | TYPED),
               signature_data (binary hoặc string)
Outputs:       contract.organizer_signed (boolean), contract.sponsor_signed (boolean),
               contract.status = SIGNED (khi cả hai ký), signed_at (timestamp per signer)
Business Rules: BR-0505, BR-0506
Acceptance Criteria:
  Given   contract ở trạng thái CONFIRMED
  When    organizer vẽ chữ ký và nhấn "Ký"
  Then    hệ thống SHALL lưu chữ ký và ghi nhận organizer_signed = true, organizer_signed_at

  Given   organizer_signed = true VÀ sponsor vẽ chữ ký
  When    sponsor nhấn "Ký"
  Then    hệ thống SHALL chuyển contract sang SIGNED
  And     hệ thống SHALL tạo danh sách nghĩa vụ tài trợ tự động (SF-06)
  And     hệ thống SHALL thông báo cho cả hai bên "Hợp đồng đã được ký kết thành công"

  Given   contract chưa ở trạng thái CONFIRMED
  When    actor cố ký
  Then    hệ thống SHALL từ chối "Cần xác nhận nội dung bởi cả hai bên trước khi ký"
Priority:      MUST
```

### FR-0505: Xuất hợp đồng điện tử dạng PDF

```
ID:            FR-0505
Name:          Xuất tài liệu hợp đồng điện tử dạng PDF
Description:   Hệ thống SHALL cho phép cả hai bên xuất hợp đồng đã ký dưới dạng file PDF
               để lưu trữ. PDF SHALL bao gồm: toàn bộ nội dung hợp đồng, chữ ký điện tử
               của hai bên, ngày ký, mã hợp đồng, và dấu thời gian xuất tài liệu.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Actor nhấn "Xuất PDF" trên trang hợp đồng đã ký
Inputs:        contract_id
Outputs:       pdf_file (binary), file_name (string: "[contract_id]_signed.pdf")
Business Rules: BR-0507
Acceptance Criteria:
  Given   contract ở trạng thái SIGNED
  When    organizer nhấn "Xuất PDF"
  Then    hệ thống SHALL tạo file PDF chứa toàn bộ nội dung hợp đồng
  And     PDF SHALL hiển thị chữ ký điện tử của hai bên
  And     PDF SHALL bao gồm mã hợp đồng và dấu thời gian

  Given   contract ở trạng thái DRAFTING
  When    actor nhấn "Xuất PDF"
  Then    hệ thống SHALL từ chối "Chỉ có thể xuất PDF cho hợp đồng đã ký"
Priority:      MUST
```

### FR-0506: Phát hành hóa đơn VAT

```
ID:            FR-0506
Name:          Tạo và phát hành hóa đơn VAT cho giao dịch tiền mặt
Description:   Hệ thống SHALL cho phép sponsor yêu cầu hóa đơn VAT (hóa đơn đỏ) cho các
               giao dịch tài trợ tiền mặt. Hệ thống SHALL tạo hóa đơn với thông tin:
               tên và mã số thuế doanh nghiệp, nội dung dịch vụ, giá trị trước thuế,
               thuế suất VAT, tổng giá trị. Hóa đơn có thể được xuất dạng PDF.
Classification: SYSTEM-SUPPORTED
Actor:         Sponsor (yêu cầu), System (tạo hóa đơn)
Trigger:       Sponsor nhấn "Yêu cầu hóa đơn VAT" trên trang hợp đồng đã ký
Inputs:        contract_id, business_name (string), tax_code (string),
               business_address (string), service_description (text)
Outputs:       invoice_id (UUID), invoice_number (string — sequential),
               invoice_pdf (binary), issued_at (timestamp)
Business Rules: BR-0508
Acceptance Criteria:
  Given   contract ở trạng thái SIGNED và hình thức tài trợ bao gồm CASH
  When    sponsor nhấn "Yêu cầu hóa đơn VAT" và nhập mã số thuế = "0312345678"
  Then    hệ thống SHALL tạo hóa đơn VAT với số hóa đơn tự động
  And     hệ thống SHALL cho phép xuất hóa đơn dạng PDF

  Given   hình thức tài trợ chỉ có IN_KIND (không có tiền mặt)
  When    sponsor yêu cầu hóa đơn VAT
  Then    hệ thống SHALL từ chối "Hóa đơn VAT chỉ áp dụng cho giao dịch tiền mặt"
Priority:      SHOULD
```

---

## Business Rules

```
ID:          BR-0501
Rule:        Hợp đồng chỉ có thể được tạo từ deal ở trạng thái AGREED.
             Mỗi deal chỉ có DUY NHẤT một hợp đồng.
Source:      Quy trình gốc — Bước 4 (Sau khi thương thảo thành công)
Type:        Validation

ID:          BR-0502
Rule:        Hợp đồng chỉ có thể chỉnh sửa khi ở trạng thái DRAFTING.
             Trạng thái CONFIRMED hoặc SIGNED không cho phép chỉnh sửa.
Source:      Quy trình gốc — Bước 4 (Soạn thảo, điền thông tin)
Type:        Authorization

ID:          BR-0503
Rule:        Hệ thống PHẢI lưu lịch sử mọi thay đổi trên hợp đồng bao gồm:
             người chỉnh sửa, thời gian, trường thay đổi, giá trị cũ, giá trị mới.
Source:      [ASSUMED — audit trail cho tài liệu pháp lý]
Type:        Validation

ID:          BR-0504
Rule:        Khi bất kỳ bên nào chỉnh sửa nội dung hợp đồng sau khi đã có xác nhận,
             TẤT CẢ xác nhận trước đó PHẢI được reset. Quy trình xác nhận bắt đầu lại.
Source:      [INFERRED — đảm bảo cả hai bên đồng ý với phiên bản cuối cùng]
Type:        Routing

ID:          BR-0505
Rule:        Chữ ký điện tử chỉ có thể thực hiện khi hợp đồng ở trạng thái CONFIRMED.
             Mỗi bên chỉ ký MỘT LẦN. Sau khi ký không thể rút lại.
Source:      Quy trình gốc — Bước 4 (Cả hai ký chữ ký điện tử)
Type:        Authorization

ID:          BR-0506
Rule:        Hợp đồng chuyển sang trạng thái SIGNED khi VÀ CHỈ KHI cả hai bên
             (organizer VÀ sponsor) đều đã ký chữ ký điện tử.
Source:      Quy trình gốc — Bước 4 (Cả hai ký chữ ký điện tử)
Type:        Routing

ID:          BR-0507
Rule:        Chỉ hợp đồng ở trạng thái SIGNED mới có thể xuất PDF.
             PDF SHALL là tài liệu chỉ đọc (read-only) với watermark "BẢN GỐC ĐIỆN TỬ". [ASSUMED]
Source:      Quy trình gốc — Bước 4 (Xuất tài liệu hợp đồng điện tử)
Type:        Validation

ID:          BR-0508
Rule:        Hóa đơn VAT chỉ phát hành cho hợp đồng SIGNED có hình thức tài trợ bao gồm CASH.
             Mã số thuế (tax_code) là BẮT BUỘC và PHẢI đúng định dạng (10 hoặc 13 chữ số). [ASSUMED]
             Mỗi hợp đồng chỉ có tối đa MỘT hóa đơn VAT.
Source:      Quy trình gốc — Bước 4 (DN cần hóa đơn đỏ cho giao dịch tiền mặt)
Type:        Validation
```

---

## Data Model

```
Entity:        Contract
Attributes:
  - contract_id: UUID (PK)
  - deal_id: UUID (FK → Deal, UNIQUE)
  - contract_number: String (auto-generated, human-readable)
  - status: Enum [DRAFTING, CONFIRMED, SIGNED] (default: DRAFTING)
  - version_number: Integer (default: 1, increment on edit)
  - organizer_info: JSON { name, representative, address, phone, email }
  - sponsor_info: JSON { name, representative, tax_code, address, phone, email }
  - signing_date: Date (nullable — populated on sign)
  - validity_start: Date (required)
  - validity_end: Date (required)
  - transaction_deadline: Date (required)
  - sponsorship_type: Enum [CASH, IN_KIND, COMBINED]
  - sponsorship_value: Decimal (required)
  - sponsorship_value_currency: Enum [VND]
  - sponsorship_details: RichText (chi tiết nội dung hoạt động tài trợ)
  - sponsor_benefits: RichText (quyền lợi nhà tài trợ)
  - organizer_benefits: RichText (quyền lợi BTC)
  - sponsor_obligations: RichText (cam kết và trách nhiệm doanh nghiệp)
  - organizer_obligations: RichText (cam kết và trách nhiệm BTC)
  - organizer_content_confirmed: Boolean (default: false)
  - sponsor_content_confirmed: Boolean (default: false)
  - organizer_signed: Boolean (default: false)
  - sponsor_signed: Boolean (default: false)
  - organizer_signature_data: Binary (nullable)
  - sponsor_signature_data: Binary (nullable)
  - organizer_signed_at: DateTime (nullable)
  - sponsor_signed_at: DateTime (nullable)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - Contract —(1:1)→ Deal
  - Contract —(1:N)→ ContractEditLog
  - Contract —(0..1:1)→ VATInvoice
  - Contract —(1:N)→ Obligation (SF-06)

Entity:        ContractEditLog
Attributes:
  - log_id: UUID (PK)
  - contract_id: UUID (FK → Contract)
  - edited_by: UUID (FK → User)
  - edited_at: DateTime
  - section: String (trường/mục thay đổi)
  - old_value: Text
  - new_value: Text
  - version_number: Integer
Relationships:
  - ContractEditLog —(N:1)→ Contract

Entity:        VATInvoice
Attributes:
  - invoice_id: UUID (PK)
  - contract_id: UUID (FK → Contract, UNIQUE)
  - invoice_number: String (auto-generated, sequential)
  - business_name: String (required)
  - tax_code: String (required, 10 or 13 digits)
  - business_address: String (required)
  - service_description: Text (required)
  - amount_before_tax: Decimal
  - vat_rate: Decimal (default: 0.10 — 10%)
  - vat_amount: Decimal (calculated)
  - total_amount: Decimal (calculated)
  - issued_at: DateTime
  - issued_by: UUID (FK → System)
Relationships:
  - VATInvoice —(1:1)→ Contract
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 4. Soạn thảo hợp đồng (khởi tạo) | FULLY AUTOMATED | FR-0501 | BR-0501 | Contract |
| 4. Điền thông tin hợp đồng | SYSTEM-SUPPORTED | FR-0502 | BR-0502, BR-0503 | Contract, ContractEditLog |
| 4. Xác nhận nội dung hợp đồng | SYSTEM-SUPPORTED | FR-0503 | BR-0504 | Contract |
| 4. Ký chữ ký điện tử | SYSTEM-SUPPORTED | FR-0504 | BR-0505, BR-0506 | Contract |
| 4. Xuất tài liệu hợp đồng điện tử | SYSTEM-SUPPORTED | FR-0505 | BR-0507 | Contract |
| 4. Hóa đơn đỏ cho giao dịch tiền mặt | SYSTEM-SUPPORTED | FR-0506 | BR-0508 | VATInvoice |
