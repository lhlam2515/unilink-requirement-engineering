# SF-05: Contract Management

## Overview

```
Scope:            Quản lý toàn bộ vòng đời hợp đồng tài trợ — từ soạn thảo, chỉnh sửa nội dung,
                  xác nhận song phương, khóa cứng giai đoạn ký kết 72 giờ, ký chữ ký điện tử,
                  xuất tài liệu PDF, đến xử lý vi phạm khi quá hạn ký.
                  [UPDATED — BP03] Hóa đơn VAT cho giá trị tài trợ đã được LOẠI Bỏ khỏi
                  phạm vi SF-05. Nền tảng chỉ xuất hóa đơn VAT cho Phí quản lý chiến dịch
                  (SF-12 FR-1208). Việc xuất hóa đơn cho giá trị tài trợ là trách nhiệm
                  của hai bên ngoài nền tảng.
System Boundary:
  IN:             Tạo bản nháp hợp đồng từ deal đã đồng thuận và đã thanh toán phí; Nhập/chỉnh sửa các điều khoản;
                  Xác nhận nội dung; Ký chữ ký điện tử; Xuất PDF; Khóa cứng giai đoạn ký kết 72 giờ sau 2/2 thanh toán.
  OUT:            Thương thảo (SF-04 — đã hoàn tất trước khi bắt đầu);
                  Thanh toán phí dịch vụ (SF-12 — đã hoàn tất trước khi tạo hợp đồng);
                  Hóa đơn VAT phí dịch vụ (SF-12 FR-1208 — không thuộc SF-05);
                  Thực hiện nghĩa vụ (SF-06 — bắt đầu sau khi ký kết).
Assumptions:
  - [ASSUMED] Hệ thống sử dụng chữ ký điện tử đơn giản (drawn/typed signature) chứ không phải
    chữ ký số mã hóa (digital signature with PKI) ở phiên bản đầu.
  - [ASSUMED] Hợp đồng được tạo từ template có sẵn, điền thông tin từ deal context.
  - [REMOVED — BP03] Hóa đơn VAT cho GIÁ TRỊ TÀI TRỢ đã bị loại bỏ khỏi SF-05.
    Nền tảng chỉ xuất hóa đơn VAT cho Phí quản lý chiến dịch (SF-12).
Gaps Detected:
  - Cần bổ sung trạng thái hard-lock và thời hạn ký 72 giờ sau khi 2/2 thanh toán hoàn tất.
  - Không nêu phiên bản (versioning) cho hợp đồng trong quá trình chỉnh sửa.
  - Cần bổ sung cơ chế báo cáo vi phạm khi một bên cố tình trì hoãn hoặc từ chối ký quá hạn.
  - [RESOLVED — BP03] Đã loại bỏ FR-0506 (hóa đơn VAT giá trị tài trợ) và BR-0508.
    Di chuyển chức năng hóa đơn VAT sang SF-12 (chỉ cho phí dịch vụ).
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Tài khoản đại diện Ban tổ chức (BTC) | `organizer` | Soạn thảo, chỉnh sửa, xác nhận, ký chữ ký điện tử |
| Tài khoản đại diện doanh nghiệp | `sponsor` | Soạn thảo, chỉnh sửa, xác nhận, ký chữ ký điện tử |
| Hệ thống | `system` | Tạo bản nháp từ template, xác thực nội dung, tạo PDF, ghi log |

---

## Functional Requirements

### FR-0501: Tạo bản nháp hợp đồng từ deal

```
ID:            FR-0501
Name:          Khởi tạo bản nháp hợp đồng tài trợ
Description:   Hệ thống SHALL tự động tạo bản nháp hợp đồng khi deal chuyển sang trạng thái AGREED
               (sau khi thanh toán phí dịch vụ hoàn tất — SF-12 FR-1207).
               Bản nháp SHALL được điền sẵn thông tin từ deal context: thông tin BTC, thông tin
               doanh nghiệp, thông tin sự kiện, gói tài trợ đã thảo luận, và thông tin từ
               thỏa thuận nháp (SF-04 FR-0408). Hệ thống SHALL sử dụng
               template hợp đồng chuẩn của nền tảng và khởi tạo thời hạn ký 72 giờ.
               [UPDATED — BP03] Trigger thay đổi: deal.status = AGREED (sau khi thanh toán
               2/2 hoàn tất và thông tin liên hệ đã mở khóa).
Classification: FULLY AUTOMATED (khởi tạo) → SYSTEM-SUPPORTED (chỉnh sửa)
Actor:         System (khởi tạo), Organizer & Sponsor (chỉnh sửa)
Trigger:       Deal chuyển sang trạng thái AGREED (từ SF-12, FR-1207 — sau khi 2/2 thanh toán)
Inputs:        deal_id, proposal_id, organizer_info, sponsor_info, agreed_terms (từ deal notes),
               draft_agreement (từ SF-04 FR-0408)
Outputs:       contract_id (UUID), status = DRAFTING, pre-populated contract fields
Business Rules: BR-0501
Acceptance Criteria:
  Given   deal "deal-001" vừa chuyển sang trạng thái AGREED
  When    hệ thống xử lý sự kiện
  Then    hệ thống SHALL tạo contract mới với trạng thái DRAFTING
  And     hệ thống SHALL điền sẵn thông tin BTC, doanh nghiệp, và sự kiện từ deal
  And     hệ thống SHALL khởi tạo signing_deadline_at = agreed_at + 72 giờ
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

### FR-0507: Khóa cứng giai đoạn ký kết hợp đồng

```
ID:            FR-0507
Name:          Khóa cứng tính năng hủy hợp đồng và đếm ngược thời hạn ký
Description:   Hệ thống SHALL tự động vô hiệu hóa mọi hành động hủy hợp đồng/hủy đồng thuận
               ngay khi PaywallSession đạt COMPLETED (2/2). Từ thời điểm này, contract được
               đặt vào trạng thái hard-lock, hiển thị đếm ngược 72 giờ để cả hai bên hoàn tất
               ký chữ ký điện tử. Nếu hết thời hạn mà chưa đủ 2 chữ ký, hệ thống SHALL phát
               sinh sự kiện vi phạm để SF-14 xử lý.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       PaywallSession chuyển sang COMPLETED (2/2)
Inputs:        deal_id, paywall_session_id, agreed_at
Outputs:       contract.hard_locked_at, contract.signing_deadline_at,
               contract.cancel_action_enabled = false,
               contract.signing_status = IN_PROGRESS
Business Rules: BR-0509, BR-0510
Acceptance Criteria:
  Given   PaywallSession vừa chuyển sang COMPLETED (2/2)
  When    hệ thống xử lý sự kiện
  Then    hệ thống SHALL vô hiệu hóa hoàn toàn hành động hủy hợp đồng/hủy đồng thuận
  And     hệ thống SHALL ghi nhận hard_locked_at
  And     hệ thống SHALL thiết lập signing_deadline_at = hard_locked_at + 72 giờ
  And     hệ thống SHALL hiển thị countdown 72 giờ trên màn hình hợp đồng

  Given   contract đã ở trạng thái hard-lock
  When    actor cố tìm hoặc truy cập hành động hủy hợp đồng
  Then    hệ thống SHALL không hiển thị nút hủy
  And     hệ thống SHALL từ chối mọi thao tác hủy qua API/UI với thông báo "Hợp đồng đã vào giai đoạn ký kết, không thể hủy"
Priority:      MUST
```

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

ID:          BR-0509
Rule:        Ngay khi PaywallSession chuyển sang COMPLETED (2/2), hệ thống PHẢI kích hoạt
             hard-lock cho hợp đồng liên kết và vô hiệu hóa mọi hành động hủy hợp đồng/
             hủy đồng thuận. Mọi nút/hành động hủy phải bị ẩn hoặc trả lỗi nếu được gọi
             qua API.
Source:      [INFERRED — hard-lock sau khi cả hai bên hoàn tất nghĩa vụ thanh toán]
Type:        Validation + Authorization

ID:          BR-0510
Rule:        Hợp đồng PHẢI hoàn tất 2 chữ ký trong vòng 72 giờ kể từ khi hard-lock được kích hoạt.
             Nếu hết hạn mà chưa đủ 2 chữ ký, hệ thống PHẢI phát sinh sự kiện vi phạm để SF-14 xử lý.
Source:      [INFERRED — ép tiến độ ký kết sau thanh toán]
Type:        Time-based + Routing

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
  - signing_window_started_at: DateTime (nullable — khi deal chuyển sang AGREED)
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
  - hard_locked_at: DateTime (nullable — khi 2/2 payment hoàn tất)
  - signing_deadline_at: DateTime (nullable — hard_locked_at + 72h)
  - cancel_action_enabled: Boolean (default: true)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - Contract —(1:1)→ Deal
  - Contract —(1:N)→ ContractEditLog
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

[REMOVED — BP03] Entity VATInvoice đã bị loại bỏ khỏi SF-05.
Hóa đơn VAT cho phí dịch vụ nền tảng được quản lý bởi SF-12 (PlatformVATInvoice).
Nền tảng TUYỆT ĐỐI KHÔNG xuất hóa đơn cho giá trị gói tài trợ.
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 4. Soạn thảo hợp đồng (khởi tạo, sau thanh toán 2/2) | FULLY AUTOMATED | FR-0501 | BR-0501 | Contract |
| 4. Điền thông tin hợp đồng | SYSTEM-SUPPORTED | FR-0502 | BR-0502, BR-0503 | Contract, ContractEditLog |
| 4. Xác nhận nội dung hợp đồng | SYSTEM-SUPPORTED | FR-0503 | BR-0504 | Contract |
| 4. Ký chữ ký điện tử | SYSTEM-SUPPORTED | FR-0504 | BR-0505, BR-0506 | Contract |
| 4. Xuất tài liệu hợp đồng điện tử | SYSTEM-SUPPORTED | FR-0505 | BR-0507 | Contract |
| ~~4. Hóa đơn đỏ cho giao dịch tiền mặt~~ | ~~REMOVED~~ | ~~FR-0506~~ | ~~BR-0508~~ | ~~VATInvoice~~ |
| [INFERRED] Khóa cứng giai đoạn ký kết | FULLY AUTOMATED | FR-0507 | BR-0509, BR-0510 | Contract, Deal |

> **Ghi chú [UPDATED — BP03]:**
> - FR-0501: Trigger thay đổi từ `deal.status = AGREED (SF-04)` sang `deal.status = AGREED (SF-12 FR-1207, sau thanh toán 2/2)`.
> - FR-0506 + BR-0508 + VATInvoice: Đã LOẠI BỎ. Nền tảng chỉ xuất hóa đơn VAT cho Phí dịch vụ (SF-12 FR-1208).
> - FR-0507: Đã chuyển từ "hủy đồng thuận" sang "khóa cứng giai đoạn ký kết 72 giờ" theo policy mới.
