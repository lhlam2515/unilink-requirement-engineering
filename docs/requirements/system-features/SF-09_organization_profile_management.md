# SF-09: Organization Profile & Document Management

## Overview

```
Scope:            Quản lý hồ sơ tổ chức sau khi đăng ký — bao gồm bổ sung thông tin tùy chọn,
                  tải lên tài liệu minh chứng, chỉnh sửa hồ sơ, gửi yêu cầu xác thực, phân quyền
                  theo trạng thái xác thực, và quản lý vòng đời tài liệu minh chứng.
System Boundary:
  IN:             Bổ sung thông tin tùy chọn (fanpage, MST, giấy tờ); Tải lên tài liệu minh chứng;
                  Chỉnh sửa hồ sơ tổ chức; Gửi hồ sơ xác thực; Phân quyền chức năng theo trạng thái
                  xác thực; Xóa tài liệu tạm theo chính sách bảo mật.
  OUT:            Đăng ký tài khoản và chọn vai trò (SF-08); Quy trình kiểm duyệt bởi Admin (SF-10);
                  Các chức năng bị giới hạn/mở khóa (SF-01 → SF-07).
Assumptions:
  - [ASSUMED] Tài liệu minh chứng hỗ trợ các định dạng phổ biến: PDF, JPEG, PNG, WebP.
  - [ASSUMED] Kích thước tối đa mỗi file tài liệu là 10MB.
  - [ASSUMED] Tổ chức có thể gửi hồ sơ xác thực nhiều lần (sau khi bị từ chối hoặc yêu cầu bổ sung).
  - [ASSUMED] Deadline 14 ngày được tính từ ngày tạo tài khoản hoặc ngày gửi yêu cầu xác thực
    gần nhất (theo BP02 — Bước 3).
  - [ASSUMED] Hồ sơ đã VERIFIED không thể chỉnh sửa trực tiếp (theo BP02 — Lưu ý 9).
Gaps Detected:
  - BP02 nêu "tài liệu minh chứng được lưu tạm thời và xóa sau 7 ngày" nhưng không rõ 7 ngày
    kể từ khi nào (tải lên? phê duyệt? từ chối?) → cần làm rõ.
  - BP02 không nêu giới hạn số lần gửi lại hồ sơ xác thực sau khi bị từ chối.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Tài khoản đại diện CLB/BTC | `organizer` | Bổ sung thông tin tổ chức, upload tài liệu, gửi hồ sơ xác thực, chỉnh sửa hồ sơ |
| Tài khoản đại diện doanh nghiệp | `sponsor` | Bổ sung thông tin tổ chức, upload tài liệu, gửi hồ sơ xác thực, chỉnh sửa hồ sơ |
| Hệ thống | `system` | Xác thực file upload, chuyển trạng thái, gửi thông báo, xóa tài liệu tạm theo chính sách, phân quyền tự động |

---

## Functional Requirements

### FR-0901: Bổ sung thông tin và tài liệu minh chứng

```
ID:            FR-0901
Name:          Bổ sung thông tin tùy chọn và tải lên tài liệu minh chứng
Description:   Hệ thống SHALL cho phép người dùng bổ sung các thông tin tùy chọn và tải lên
               tài liệu minh chứng sau khi hoàn tất đăng ký cơ bản. Tài liệu minh chứng khác
               nhau theo vai trò:
               - Organizer: Giấy quyết định thành lập CLB (bản sao có dấu đỏ), giấy giới thiệu
                 của đoàn trường, đường dẫn fanpage/website.
               - Sponsor: Mã số thuế, giấy tờ chứng thực hoạt động, giấy phép kinh doanh.
               Hệ thống SHALL xác thực định dạng và kích thước file theo BR-0901.
               Người dùng có thể bổ sung tài liệu bất kỳ lúc nào trong vòng 14 ngày (BR-0902).
Classification: SYSTEM-SUPPORTED
Actor:         Organizer hoặc Sponsor
Trigger:       Người dùng truy cập trang "Hồ sơ tổ chức" và chọn "Bổ sung thông tin"
Inputs:
  - Organizer: establishment_decision_doc (file), youth_union_letter (file),
               fanpage_url (string, optional)
  - Sponsor:   tax_code (string, optional), business_cert_doc (file),
               business_license_doc (file)
Outputs:       Tài liệu đã lưu trữ, document_id (UUID), upload_status,
               hồ sơ tổ chức đã cập nhật
Business Rules: BR-0901, BR-0902
Acceptance Criteria:
  Given   organizer đã đăng ký và chưa gửi hồ sơ xác thực
  When    organizer tải lên file PDF "giấy quyết định thành lập" kích thước 3MB
  Then    hệ thống SHALL lưu trữ file và gắn document_id duy nhất
  And     hệ thống SHALL hiển thị tài liệu đã tải lên trong danh sách minh chứng

  Given   sponsor tải lên file có kích thước 15MB (vượt giới hạn BR-0901)
  When    hệ thống kiểm tra file
  Then    hệ thống SHALL từ chối "Kích thước file vượt quá giới hạn 10MB"

  Given   sponsor tải lên file định dạng .exe
  When    hệ thống kiểm tra file
  Then    hệ thống SHALL từ chối "Chỉ chấp nhận định dạng PDF, JPEG, PNG, WebP"
Priority:      MUST
```

### FR-0902: Chỉnh sửa hồ sơ tổ chức

```
ID:            FR-0902
Name:          Chỉnh sửa thông tin hồ sơ tổ chức
Description:   Hệ thống SHALL cho phép người dùng chỉnh sửa thông tin hồ sơ tổ chức theo điều
               kiện tại BR-0903:
               - Trạng thái UNVERIFIED hoặc INFO_REQUIRED: chỉnh sửa tất cả thông tin.
               - Trạng thái PENDING_REVIEW: chỉnh sửa được nhưng hồ sơ xác thực sẽ được
                 đánh dấu cập nhật mới để kiểm duyệt viên biết.
               - Trạng thái VERIFIED: KHÔNG THỂ chỉnh sửa trực tiếp.
               - Trạng thái REJECTED: chỉnh sửa tất cả thông tin để gửi lại.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer hoặc Sponsor
Trigger:       Người dùng truy cập trang "Hồ sơ tổ chức" và nhấn "Chỉnh sửa"
Inputs:        Các trường thông tin tổ chức cần cập nhật (tùy thuộc vai trò)
Outputs:       Hồ sơ đã cập nhật, updated_at (timestamp)
Business Rules: BR-0903
Acceptance Criteria:
  Given   tài khoản có verification_status = UNVERIFIED
  When    người dùng chỉnh sửa tên tổ chức và lưu
  Then    hệ thống SHALL cập nhật thông tin thành công

  Given   tài khoản có verification_status = PENDING_REVIEW
  When    người dùng chỉnh sửa thông tin
  Then    hệ thống SHALL cập nhật và đánh dấu hồ sơ "đã cập nhật kể từ lần gửi cuối"

  Given   tài khoản có verification_status = VERIFIED
  When    người dùng cố chỉnh sửa
  Then    hệ thống SHALL từ chối "Hồ sơ đã xác thực không thể chỉnh sửa trực tiếp"
Priority:      MUST
```

### FR-0903: Gửi hồ sơ xác thực

```
ID:            FR-0903
Name:          Gửi hồ sơ tổ chức để kiểm duyệt
Description:   Hệ thống SHALL cho phép người dùng gửi hồ sơ xác thực khi đã hoàn tất đầy đủ
               thông tin bắt buộc. Hệ thống SHALL xác thực tính đầy đủ theo BR-0904 trước khi
               tiếp nhận. Khi gửi thành công, hệ thống SHALL chuyển trạng thái tài khoản sang
               PENDING_REVIEW và gửi thông báo qua email + in-app cho người dùng xác nhận đã
               tiếp nhận. Nếu đăng ký bằng Google, hệ thống vẫn yêu cầu hoàn tất thông tin
               bắt buộc trước khi cho phép gửi.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer hoặc Sponsor
Trigger:       Người dùng nhấn "Gửi hồ sơ xác thực" trên trang hồ sơ tổ chức
Inputs:        account_id (implicit từ session)
Outputs:       verification_status = PENDING_REVIEW, submitted_at (timestamp),
               notification (email + in-app), verification_request_id (UUID)
Business Rules: BR-0904, BR-0905
Acceptance Criteria:
  Given   hồ sơ tổ chức đã có đầy đủ thông tin bắt buộc và email đã xác minh
  When    người dùng nhấn "Gửi hồ sơ xác thực"
  Then    hệ thống SHALL chuyển verification_status sang PENDING_REVIEW
  And     hệ thống SHALL ghi nhận submitted_at
  And     hệ thống SHALL gửi email "Hồ sơ xác thực của bạn đã được tiếp nhận"
  And     hệ thống SHALL gửi thông báo in-app tương tự

  Given   hồ sơ thiếu thông tin bắt buộc (ví dụ: chưa nhập tên tổ chức)
  When    người dùng nhấn "Gửi hồ sơ xác thực"
  Then    hệ thống SHALL từ chối và hiển thị danh sách thông tin còn thiếu

  Given   tài khoản đăng ký bằng Google và chưa nhập đầy đủ thông tin tổ chức
  When    người dùng nhấn "Gửi hồ sơ xác thực"
  Then    hệ thống SHALL từ chối "Vui lòng hoàn tất thông tin tổ chức trước khi gửi"

  Given   tài khoản đã VERIFIED
  When    người dùng cố gửi hồ sơ xác thực lại
  Then    hệ thống SHALL từ chối "Tài khoản đã được xác thực"
Priority:      MUST
```

### FR-0904: Phân quyền theo trạng thái xác thực

```
ID:            FR-0904
Name:          Phân quyền chức năng tự động theo trạng thái xác thực
Description:   Hệ thống SHALL tự động phân quyền truy cập chức năng dựa trên vai trò và trạng
               thái xác thực của tài khoản theo BR-0906.
               - Organizer chưa xác thực: CÓ THỂ tạo hồ sơ tài trợ (SF-01) nhưng KHÔNG THỂ
                 chấp nhận lời mời tài trợ để tiến hành thương thảo.
               - Sponsor chưa xác thực: CÓ THỂ xem thông tin công khai nhưng KHÔNG THỂ
                 gửi lời mời tài trợ.
               - Sau khi VERIFIED: tất cả chức năng được mở khóa theo vai trò.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       Thay đổi verification_status của tài khoản (tự động sau phê duyệt/từ chối)
Inputs:        account_id, verification_status, role
Outputs:       Danh sách chức năng được phép truy cập (permission set)
Business Rules: BR-0906
Acceptance Criteria:
  Given   organizer có verification_status = UNVERIFIED
  When    organizer cố chấp nhận lời mời tài trợ (SF-03)
  Then    hệ thống SHALL từ chối "Tài khoản cần được xác thực để thực hiện chức năng này"

  Given   sponsor có verification_status = UNVERIFIED
  When    sponsor cố gửi lời mời tài trợ (SF-03)
  Then    hệ thống SHALL từ chối "Tài khoản cần được xác thực để gửi lời mời tài trợ"

  Given   organizer có verification_status = VERIFIED
  When    organizer chấp nhận lời mời tài trợ
  Then    hệ thống SHALL cho phép thực hiện bình thường

  Given   tài khoản vừa được phê duyệt (VERIFIED)
  When    hệ thống cập nhật verification_status
  Then    hệ thống SHALL tự động mở khóa tất cả quyền tương ứng với vai trò
Priority:      MUST
```

### FR-0905: Xóa tài liệu tạm theo chính sách bảo mật

```
ID:            FR-0905
Name:          Tự động xóa tài liệu minh chứng theo chính sách bảo mật
Description:   Hệ thống SHALL tự động xóa tài liệu minh chứng đã tải lên sau 7 ngày kể từ
               khi hồ sơ xác thực được xử lý hoàn tất (VERIFIED hoặc REJECTED lần cuối).
               Scheduled job chạy hàng ngày để kiểm tra và xóa tài liệu đã hết hạn lưu trữ
               theo BR-0907.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       Scheduled job chạy hàng ngày (daily cleanup)
Inputs:        Danh sách tài liệu minh chứng có retention_expires_at < hôm nay
Outputs:       Tài liệu đã xóa, cleanup_log entry
Business Rules: BR-0907
Acceptance Criteria:
  Given   hồ sơ organizer đã VERIFIED vào ngày 01/04/2026
  And     tài liệu minh chứng có retention_expires_at = 08/04/2026
  And     hôm nay là 09/04/2026
  When    scheduled job chạy
  Then    hệ thống SHALL xóa tài liệu và ghi nhận cleanup_log

  Given   hồ sơ đang ở trạng thái PENDING_REVIEW
  When    scheduled job chạy
  Then    hệ thống SHALL KHÔNG xóa tài liệu (chưa xử lý xong)

  Given   hồ sơ bị REJECTED và tài liệu chưa hết hạn lưu trữ
  When    scheduled job chạy
  Then    hệ thống SHALL giữ tài liệu để người dùng có thể tham khảo khi gửi lại
Priority:      SHOULD
```

---

## Business Rules

```
ID:          BR-0901
Rule:        Tài liệu minh chứng PHẢI có định dạng PDF, JPEG, PNG, hoặc WebP.
             Kích thước tối đa mỗi file là 10MB. [ASSUMED]
Source:      BP02 — Bước 3 (Bổ sung thông tin và tài liệu minh chứng)
Type:        Validation

ID:          BR-0902
Rule:        Các thông tin và tài liệu bổ sung cần được hoàn tất trong vòng 14 ngày
             kể từ ngày tạo tài khoản hoặc ngày gửi yêu cầu xác thực gần nhất.
             Hệ thống gửi nhắc nhở khi còn 3 ngày trước deadline. [ASSUMED]
Source:      BP02 — Bước 3 (Hoàn tất trong vòng 14 ngày)
Type:        Time-based

ID:          BR-0903
Rule:        Quyền chỉnh sửa hồ sơ tổ chức phụ thuộc vào verification_status:
             - UNVERIFIED: chỉnh sửa tất cả.
             - PENDING_REVIEW: chỉnh sửa được, hồ sơ đánh dấu "đã cập nhật".
             - VERIFIED: KHÔNG chỉnh sửa trực tiếp.
             - REJECTED: chỉnh sửa tất cả để gửi lại.
             - INFO_REQUIRED: chỉnh sửa tất cả để bổ sung.
Source:      BP02 — Bước 3 + Lưu ý 9 (Hồ sơ đã phê duyệt không thể chỉnh sửa trực tiếp)
Type:        Authorization

ID:          BR-0904
Rule:        Hồ sơ xác thực chỉ được gửi khi ĐẦY ĐỦ thông tin bắt buộc (BR-0807 từ SF-08)
             và email đã xác minh (email_verified = true).
Source:      BP02 — Bước 4 (Chuẩn bị đầy đủ thông tin cần thiết)
Type:        Validation

ID:          BR-0905
Rule:        Khi gửi hồ sơ xác thực, verification_status PHẢI chuyển từ
             UNVERIFIED/REJECTED/INFO_REQUIRED sang PENDING_REVIEW.
             Không cho phép gửi khi đã ở trạng thái PENDING_REVIEW hoặc VERIFIED.
Source:      BP02 — Bước 4 (Ghi nhận trạng thái đang chờ kiểm duyệt)
Type:        Routing

ID:          BR-0906
Rule:        Phân quyền chức năng theo vai trò + trạng thái xác thực:
             - Organizer + UNVERIFIED: tạo hồ sơ tài trợ (SF-01) ✓,
               chấp nhận lời mời để thương thảo ✗.
             - Sponsor + UNVERIFIED: xem thông tin công khai ✓,
               gửi lời mời tài trợ ✗.
             - Bất kỳ + VERIFIED: tất cả chức năng theo vai trò ✓.
Source:      BP02 — Bước 7.1 và 7.2 (Quyền lợi theo trạng thái xác thực)
Type:        Authorization

ID:          BR-0907
Rule:        Tài liệu minh chứng được lưu tạm thời và PHẢI được xóa sau 7 ngày
             kể từ khi hồ sơ xác thực được xử lý hoàn tất (VERIFIED hoặc REJECTED).
             Tài liệu thuộc hồ sơ đang PENDING_REVIEW hoặc INFO_REQUIRED KHÔNG bị xóa.
Source:      BP02 — Lưu ý 9 (Tài liệu minh chứng xóa sau 7 ngày theo quy định bảo mật)
Type:        Time-based
```

---

## Data Model

```
Entity:        VerificationDocument
Attributes:
  - document_id: UUID (PK)
  - org_id: UUID (FK → Organization)
  - document_type: Enum [ESTABLISHMENT_DECISION, YOUTH_UNION_LETTER, FANPAGE_URL,
                         TAX_CODE, BUSINESS_CERT, BUSINESS_LICENSE, OTHER]
  - file_url: String (nullable — null khi document_type là URL-based như FANPAGE_URL)
  - file_name: String (nullable)
  - file_size_bytes: Integer (nullable)
  - file_mime_type: String (nullable)
  - text_value: String (nullable — cho loại dữ liệu text như tax_code, fanpage_url)
  - retention_expires_at: DateTime (nullable — set khi hồ sơ được xử lý hoàn tất)
  - uploaded_at: DateTime
  - deleted_at: DateTime (nullable — soft delete timestamp)
Relationships:
  - VerificationDocument —(N:1)→ Organization

Entity:        VerificationRequest
Attributes:
  - request_id: UUID (PK)
  - org_id: UUID (FK → Organization)
  - status: Enum [PENDING, APPROVED, REJECTED, INFO_REQUIRED]
  - submitted_at: DateTime
  - reviewed_at: DateTime (nullable)
  - reviewed_by: UUID (FK → Account/Admin, nullable)
  - reviewer_notes: Text (nullable — lý do từ chối hoặc yêu cầu bổ sung)
  - info_request_details: Text (nullable — chi tiết cần bổ sung khi INFO_REQUIRED)
  - deadline_at: DateTime (submitted_at + 14 ngày)
Relationships:
  - VerificationRequest —(N:1)→ Organization
  - VerificationRequest —(N:1)→ Account (reviewer)
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 3. Bổ sung thông tin và tài liệu minh chứng | SYSTEM-SUPPORTED | FR-0901 | BR-0901, BR-0902 | VerificationDocument |
| 3/5. Chỉnh sửa hồ sơ đã gửi | SYSTEM-SUPPORTED | FR-0902 | BR-0903 | Organization |
| 4. Gửi hồ sơ xác thực | SYSTEM-SUPPORTED | FR-0903 | BR-0904, BR-0905 | VerificationRequest |
| 7. Phân quyền sau xác thực | FULLY AUTOMATED | FR-0904 | BR-0906 | Account |
| 9. Xóa tài liệu tạm theo chính sách bảo mật | FULLY AUTOMATED | FR-0905 | BR-0907 | VerificationDocument |
