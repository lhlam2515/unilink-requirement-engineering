# SF-10: Organization Verification & Moderation

## Overview

```
Scope:            Quản lý quy trình kiểm duyệt và xác minh hồ sơ tổ chức bởi Admin — bao gồm
                  xem danh sách hồ sơ chờ duyệt, xem chi tiết hồ sơ, phê duyệt, từ chối,
                  yêu cầu bổ sung thông tin, và gửi thông báo kết quả xử lý trong toàn bộ
                  vòng đời kiểm duyệt.
System Boundary:
  IN:             Dashboard kiểm duyệt hồ sơ; Xem chi tiết hồ sơ + tài liệu minh chứng;
                  Phê duyệt hồ sơ; Từ chối hồ sơ kèm lý do; Yêu cầu bổ sung thông tin;
                  Thông báo kết quả xử lý qua email và in-app.
  OUT:            Quản lý hồ sơ tổ chức (SF-09 — người dùng tự quản lý);
                  Đăng ký tài khoản (SF-08); Phân quyền chức năng (SF-09 — FR-0904).
Assumptions:
  - [ASSUMED] Admin là actor chung cho toàn hệ thống, bao gồm cả kiểm duyệt hồ sơ tổ chức
    (SF-10) và kiểm duyệt đánh giá vi phạm (SF-07).
  - [ASSUMED] Thời gian xử lý hồ sơ là 1-3 ngày làm việc — đây là thời gian bắt buộc
    (theo BP02 — Bước 5).
  - [ASSUMED] Việc kiểm tra MST (mã số thuế) doanh nghiệp được thực hiện THỦ CÔNG bởi Admin,
    không tích hợp API bên ngoài.
  - [ASSUMED] Admin có thể xem lịch sử tất cả các lần gửi và xử lý hồ sơ của một tổ chức.
Gaps Detected:
  - BP02 không nêu cơ chế phân công kiểm duyệt viên (auto-assign hay self-assign) → cần bổ sung.
  - BP02 không nêu chỉ số SLA cụ thể cho Admin (cảnh báo khi xử lý quá hạn) → cần bổ sung.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Nhân sự kiểm duyệt / Đội ngũ vận hành | `admin` | Xem danh sách hồ sơ chờ duyệt, xem chi tiết, phê duyệt, từ chối, yêu cầu bổ sung |
| Admin hệ thống | `admin` | Tất cả quyền trên + theo dõi quá trình kiểm duyệt để đảm bảo minh bạch |
| Hệ thống | `system` | Gửi thông báo kết quả, chuyển trạng thái, mở khóa quyền, ghi audit log |

---

## Functional Requirements

### FR-1001: Xem danh sách hồ sơ chờ kiểm duyệt

```
ID:            FR-1001
Name:          Xem danh sách hồ sơ tổ chức chờ kiểm duyệt
Description:   Hệ thống SHALL hiển thị cho Admin danh sách tất cả hồ sơ xác thực có trạng thái
               PENDING. Danh sách SHALL hiển thị thông tin tóm tắt: tên tổ chức, vai trò
               (Organizer/Sponsor), ngày gửi, thời gian chờ xử lý. Danh sách SHALL được sắp xếp
               theo thứ tự ưu tiên (submitted_at ASC — hồ sơ gửi trước xử lý trước).
               Hệ thống SHALL hỗ trợ lọc theo vai trò và tìm kiếm theo tên tổ chức.
Classification: SYSTEM-SUPPORTED
Actor:         Admin
Trigger:       Admin truy cập trang "Kiểm duyệt hồ sơ"
Inputs:        filter_role (optional: ORGANIZER | SPONSOR | ALL),
               search_query (optional: tên tổ chức), page (integer)
Outputs:       Danh sách hồ sơ chờ duyệt với phân trang,
               tổng số hồ sơ pending, thời gian chờ trung bình
Business Rules: BR-1001
Acceptance Criteria:
  Given   có 15 hồ sơ đang PENDING (10 Organizer, 5 Sponsor)
  When    Admin truy cập trang kiểm duyệt
  Then    hệ thống SHALL hiển thị 15 hồ sơ sắp xếp theo submitted_at ASC
  And     hệ thống SHALL hiển thị tổng số pending = 15

  Given   Admin lọc theo vai trò SPONSOR
  When    hệ thống áp dụng bộ lọc
  Then    hệ thống SHALL chỉ hiển thị 5 hồ sơ Sponsor

  Given   Admin tìm kiếm "CLB IT"
  When    hệ thống áp dụng tìm kiếm
  Then    hệ thống SHALL hiển thị các hồ sơ có tên tổ chức chứa "CLB IT"
Priority:      MUST
```

### FR-1002: Xem chi tiết hồ sơ xác thực

```
ID:            FR-1002
Name:          Xem chi tiết hồ sơ và tài liệu minh chứng của tổ chức
Description:   Hệ thống SHALL cho phép Admin xem toàn bộ thông tin hồ sơ xác thực của một
               tổ chức, bao gồm: thông tin cơ bản (tên, vai trò, email, địa chỉ), thông tin
               bổ sung (tùy vai trò), tài liệu minh chứng đã tải lên (có thể xem/tải về),
               và lịch sử các lần gửi/xử lý hồ sơ trước đó. Admin có thể kiểm tra MST
               doanh nghiệp thủ công bằng cách đối chiếu thông tin bên ngoài hệ thống.
Classification: SYSTEM-SUPPORTED
Actor:         Admin
Trigger:       Admin nhấn vào một hồ sơ trong danh sách chờ duyệt (FR-1001)
Inputs:        verification_request_id (UUID)
Outputs:       Chi tiết hồ sơ tổ chức, danh sách tài liệu minh chứng (có preview/download),
               lịch sử xác thực (verification history)
Business Rules: BR-1002
Acceptance Criteria:
  Given   hồ sơ Organizer "CLB IT HCMUS" đang PENDING
  When    Admin nhấn vào hồ sơ
  Then    hệ thống SHALL hiển thị: tên CLB, tên trường, email, địa chỉ
  And     hệ thống SHALL hiển thị danh sách tài liệu minh chứng với nút preview/download
  And     hệ thống SHALL hiển thị lịch sử xác thực (nếu đã gửi trước đó)

  Given   hồ sơ Sponsor "ABC Corp" có mã số thuế
  When    Admin xem chi tiết
  Then    hệ thống SHALL hiển thị mã số thuế để Admin đối chiếu thủ công

  Given   hồ sơ đã từng bị REJECTED trước đó và được gửi lại
  When    Admin xem chi tiết
  Then    hệ thống SHALL hiển thị lịch sử: lần gửi trước + lý do từ chối + lần gửi hiện tại
Priority:      MUST
```

### FR-1003: Phê duyệt hồ sơ tổ chức

```
ID:            FR-1003
Name:          Phê duyệt hồ sơ xác thực tổ chức
Description:   Hệ thống SHALL cho phép Admin phê duyệt hồ sơ xác thực. Khi phê duyệt, hệ thống
               SHALL chuyển verification_status của tài khoản sang VERIFIED, ghi nhận thời gian
               phê duyệt và Admin phê duyệt, tự động mở khóa quyền truy cập theo vai trò
               (trigger FR-0904 từ SF-09), và gửi thông báo cho người dùng qua email + in-app.
               Hệ thống SHALL thiết lập retention_expires_at cho tài liệu minh chứng
               (BR-0907 từ SF-09).
Classification: SYSTEM-SUPPORTED
Actor:         Admin (khởi tạo), System (xử lý hậu kỳ)
Trigger:       Admin nhấn "Phê duyệt" trên trang chi tiết hồ sơ
Inputs:        verification_request_id (UUID), admin_notes (text, optional)
Outputs:       verification_status = VERIFIED, reviewed_at (timestamp), reviewed_by (admin_id),
               notification (email + in-app cho người dùng), permission_update
Business Rules: BR-1003, BR-1006
Acceptance Criteria:
  Given   hồ sơ "CLB IT HCMUS" đang PENDING với đầy đủ thông tin và minh chứng
  When    Admin nhấn "Phê duyệt"
  Then    hệ thống SHALL chuyển verification_status sang VERIFIED
  And     hệ thống SHALL ghi nhận reviewed_at và reviewed_by
  And     hệ thống SHALL gửi email "Chúc mừng! Hồ sơ tổ chức của bạn đã được xác thực"
  And     hệ thống SHALL gửi thông báo in-app tương tự
  And     hệ thống SHALL tự động mở khóa tất cả chức năng theo vai trò (FR-0904)
  And     hệ thống SHALL thiết lập retention_expires_at = reviewed_at + 7 ngày cho tài liệu

  Given   hồ sơ đã ở trạng thái VERIFIED
  When    Admin cố phê duyệt lại
  Then    hệ thống SHALL từ chối "Hồ sơ đã được xác thực"
Priority:      MUST
```

### FR-1004: Từ chối hồ sơ tổ chức

```
ID:            FR-1004
Name:          Từ chối hồ sơ xác thực kèm lý do
Description:   Hệ thống SHALL cho phép Admin từ chối hồ sơ xác thực kèm theo lý do rõ ràng
               (bắt buộc). Khi từ chối, hệ thống SHALL chuyển verification_status sang REJECTED,
               ghi nhận lý do từ chối, và gửi thông báo cho người dùng. Người dùng KHÔNG CẦN
               tạo lại tài khoản — chỉ cần bổ sung/chỉnh sửa và gửi lại hồ sơ xác thực.
Classification: SYSTEM-SUPPORTED
Actor:         Admin
Trigger:       Admin nhấn "Từ chối" trên trang chi tiết hồ sơ
Inputs:        verification_request_id (UUID), rejection_reason (text, required)
Outputs:       verification_status = REJECTED, reviewed_at, reviewed_by,
               rejection_reason (ghi nhận), notification (email + in-app)
Business Rules: BR-1004, BR-1006
Acceptance Criteria:
  Given   hồ sơ "ABC Corp" đang PENDING và Admin phát hiện tài liệu không hợp lệ
  When    Admin nhấn "Từ chối" với lý do = "Giấy phép kinh doanh đã hết hạn"
  Then    hệ thống SHALL chuyển verification_status sang REJECTED
  And     hệ thống SHALL ghi nhận rejection_reason
  And     hệ thống SHALL gửi email "Hồ sơ của bạn bị từ chối. Lý do: Giấy phép kinh doanh
          đã hết hạn. Bạn có thể bổ sung và gửi lại."
  And     hệ thống SHALL gửi thông báo in-app tương tự

  Given   Admin cố từ chối mà không nhập lý do
  When    Admin submit
  Then    hệ thống SHALL từ chối "Vui lòng nhập lý do từ chối"

  Given   hồ sơ bị từ chối
  When    người dùng chỉnh sửa và gửi lại (FR-0903 từ SF-09)
  Then    hệ thống SHALL cho phép gửi lại và tạo verification_request mới
Priority:      MUST
```

### FR-1005: Yêu cầu bổ sung thông tin

```
ID:            FR-1005
Name:          Yêu cầu tổ chức bổ sung thông tin hoặc minh chứng
Description:   Hệ thống SHALL cho phép Admin yêu cầu người dùng bổ sung thông tin hoặc minh chứng
               khi hồ sơ chưa đủ rõ ràng. Khi yêu cầu bổ sung, hệ thống SHALL chuyển
               verification_status sang INFO_REQUIRED, ghi nhận chi tiết yêu cầu, và gửi thông
               báo cho người dùng. Người dùng sau khi bổ sung có thể gửi lại hồ sơ (FR-0903).
Classification: SYSTEM-SUPPORTED
Actor:         Admin
Trigger:       Admin nhấn "Yêu cầu bổ sung" trên trang chi tiết hồ sơ
Inputs:        verification_request_id (UUID), info_request_details (text, required)
Outputs:       verification_status = INFO_REQUIRED, info_request_details (ghi nhận),
               notification (email + in-app)
Business Rules: BR-1005, BR-1006
Acceptance Criteria:
  Given   hồ sơ "CLB IT" đang PENDING và Admin thấy cần thêm giấy giới thiệu đoàn trường
  When    Admin nhấn "Yêu cầu bổ sung" với nội dung = "Vui lòng bổ sung giấy giới thiệu
          của đoàn trường"
  Then    hệ thống SHALL chuyển verification_status sang INFO_REQUIRED
  And     hệ thống SHALL gửi email "Hồ sơ cần bổ sung: Vui lòng bổ sung giấy giới thiệu
          của đoàn trường"
  And     hệ thống SHALL gửi thông báo in-app tương tự

  Given   Admin cố yêu cầu bổ sung mà không nhập chi tiết
  When    Admin submit
  Then    hệ thống SHALL từ chối "Vui lòng nhập chi tiết cần bổ sung"
Priority:      MUST
```

### FR-1006: Thông báo sự kiện xác thực

```
ID:            FR-1006
Name:          Gửi thông báo tại các sự kiện quan trọng trong quy trình xác thực
Description:   Hệ thống SHALL tự động gửi thông báo qua email và in-app khi có các sự kiện
               quan trọng trong quy trình xác thực theo BR-1006. Danh sách sự kiện:
               (1) Đăng ký thành công, (2) Hồ sơ xác thực đã được gửi,
               (3) Hồ sơ cần bổ sung thông tin, (4) Hồ sơ được phê duyệt,
               (5) Hồ sơ bị từ chối, (6) Tài liệu minh chứng cần được cập nhật lại.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       Thay đổi trạng thái xác thực hoặc sự kiện liên quan
Inputs:        event_type (enum), account_id, context_data (tùy sự kiện)
Outputs:       Notification (email + in-app) gửi tới người dùng liên quan
Business Rules: BR-1006
Acceptance Criteria:
  Given   người dùng vừa đăng ký thành công
  When    hệ thống hoàn tất tạo tài khoản
  Then    hệ thống SHALL gửi email "Chào mừng bạn đến với UniLink!"
  And     hệ thống SHALL gửi thông báo in-app tương tự

  Given   hồ sơ vừa được phê duyệt
  When    Admin phê duyệt (FR-1003)
  Then    hệ thống SHALL gửi email "Hồ sơ đã được xác thực — tất cả chức năng đã mở khóa"
  And     hệ thống SHALL gửi thông báo in-app tương tự

  Given   hồ sơ bị từ chối
  When    Admin từ chối (FR-1004)
  Then    hệ thống SHALL gửi email chứa lý do từ chối và hướng dẫn gửi lại
  And     hệ thống SHALL gửi thông báo in-app tương tự
Priority:      MUST
```

---

## Business Rules

```
ID:          BR-1001
Rule:        Danh sách hồ sơ chờ kiểm duyệt PHẢI sắp xếp theo submitted_at ASC
             (hồ sơ gửi trước xử lý trước — FIFO). Hỗ trợ phân trang với 20 hồ sơ mỗi trang.
             [ASSUMED — 20 items/page]
Source:      BP02 — Bước 5 (Kiểm duyệt và xác minh)
Type:        Routing

ID:          BR-1002
Rule:        Admin PHẢI có quyền xem và tải về tất cả tài liệu minh chứng của hồ sơ
             đang kiểm duyệt. Lịch sử xác thực hiển thị TẤT CẢ các lần gửi và xử lý
             theo thứ tự thời gian.
Source:      BP02 — Bước 5 (Xem xét thông tin và tài liệu)
Type:        Authorization

ID:          BR-1003
Rule:        Khi phê duyệt hồ sơ:
             (1) verification_status PHẢI chuyển sang VERIFIED.
             (2) Tất cả quyền truy cập theo vai trò PHẢI được mở khóa tự động.
             (3) retention_expires_at của tài liệu minh chứng PHẢI được thiết lập = reviewed_at + 7 ngày.
Source:      BP02 — Bước 6 (Phê duyệt thành công, mở khóa quyền)
Type:        Routing + Authorization

ID:          BR-1004
Rule:        Khi từ chối hồ sơ:
             (1) rejection_reason là BẮT BUỘC — Admin PHẢI nhập lý do rõ ràng.
             (2) Người dùng KHÔNG cần tạo lại tài khoản — chỉ bổ sung/chỉnh sửa và gửi lại.
             (3) verification_status PHẢI chuyển sang REJECTED.
Source:      BP02 — Bước 6 (Từ chối kèm lý do rõ ràng, không cần tạo lại tài khoản)
Type:        Routing + Validation

ID:          BR-1005
Rule:        Khi yêu cầu bổ sung thông tin:
             (1) info_request_details là BẮT BUỘC — Admin PHẢI mô tả cụ thể cần bổ sung gì.
             (2) verification_status PHẢI chuyển sang INFO_REQUIRED.
             (3) Người dùng được phép chỉnh sửa và gửi lại hồ sơ.
Source:      BP02 — Bước 5 (Đội ngũ vận hành có thể yêu cầu bổ sung)
Type:        Routing + Validation

ID:          BR-1006
Rule:        Hệ thống PHẢI gửi thông báo qua EMAIL và IN-APP cho 6 sự kiện:
             (1) Đăng ký thành công
             (2) Hồ sơ xác thực đã được gửi
             (3) Hồ sơ cần bổ sung thông tin
             (4) Hồ sơ được phê duyệt
             (5) Hồ sơ bị từ chối
             (6) Tài liệu minh chứng cần được cập nhật lại
             Mỗi thông báo PHẢI có nội dung rõ ràng và hướng dẫn bước tiếp theo (nếu có).
Source:      BP02 — Bước 8 (Thông báo trong quá trình xử lý — 6 sự kiện)
Type:        Notification
```

---

## Data Model

```
Lưu ý: Các entity chính (Account, Organization, VerificationDocument, VerificationRequest)
đã được định nghĩa tại SF-08 và SF-09. SF-10 tái sử dụng các entity này.

Entity bổ sung:

Entity:        AuditLog (cho theo dõi kiểm duyệt)
Attributes:
  - log_id: UUID (PK)
  - entity_type: String (e.g., "VerificationRequest")
  - entity_id: UUID
  - action: Enum [SUBMITTED, APPROVED, REJECTED, INFO_REQUESTED, RESUBMITTED]
  - performed_by: UUID (FK → Account)
  - performed_at: DateTime
  - details: JSON (metadata bổ sung: lý do từ chối, chi tiết yêu cầu, ghi chú admin...)
Relationships:
  - AuditLog —(N:1)→ Account (performer)
  - AuditLog liên kết theo entity_type + entity_id (polymorphic)

Entity:        Notification
Attributes:
  - notification_id: UUID (PK)
  - recipient_id: UUID (FK → Account)
  - channel: Enum [EMAIL, IN_APP]
  - event_type: Enum [REGISTRATION_SUCCESS, VERIFICATION_SUBMITTED,
                       VERIFICATION_INFO_REQUIRED, VERIFICATION_APPROVED,
                       VERIFICATION_REJECTED, DOCUMENT_UPDATE_NEEDED]
  - title: String (required)
  - body: Text (required)
  - is_read: Boolean (default: false — chỉ cho IN_APP)
  - sent_at: DateTime
  - read_at: DateTime (nullable)
Relationships:
  - Notification —(N:1)→ Account (recipient)
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 5. Kiểm duyệt — Xem danh sách chờ duyệt | SYSTEM-SUPPORTED | FR-1001 | BR-1001 | VerificationRequest |
| 5. Kiểm duyệt — Xem chi tiết hồ sơ | SYSTEM-SUPPORTED | FR-1002 | BR-1002 | Organization, VerificationDocument |
| 6. Kết quả — Phê duyệt thành công | SYSTEM-SUPPORTED | FR-1003 | BR-1003, BR-1006 | VerificationRequest, Account, AuditLog |
| 6. Kết quả — Từ chối kèm lý do | SYSTEM-SUPPORTED | FR-1004 | BR-1004, BR-1006 | VerificationRequest, AuditLog |
| 5. Kiểm duyệt — Yêu cầu bổ sung | SYSTEM-SUPPORTED | FR-1005 | BR-1005, BR-1006 | VerificationRequest, AuditLog |
| 8. Thông báo trong quá trình xử lý | FULLY AUTOMATED | FR-1006 | BR-1006 | Notification |
