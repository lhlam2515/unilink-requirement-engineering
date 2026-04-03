# SF-08: Account Registration & Authentication

## Overview

```
Scope:            Quản lý toàn bộ quy trình đăng ký tài khoản, đăng nhập và xác thực danh tính
                  người dùng trên nền tảng UniLink — bao gồm đăng ký bằng email/mật khẩu,
                  đăng ký/đăng nhập nhanh qua Google OAuth, chọn vai trò tổ chức, và nhập thông tin
                  cơ bản bắt buộc của tổ chức.
System Boundary:
  IN:             Tạo tài khoản mới (email + password); Đăng ký/Đăng nhập qua Google OAuth;
                  Đăng nhập hệ thống; Chọn vai trò tổ chức (Organizer / Sponsor);
                  Nhập thông tin cơ bản tổ chức theo vai trò; Đặt lại mật khẩu.
  OUT:            Bổ sung tài liệu minh chứng và gửi hồ sơ xác thực (SF-09);
                  Kiểm duyệt hồ sơ tổ chức bởi Admin (SF-10);
                  Phân quyền theo trạng thái xác thực (SF-09).
Assumptions:
  - [ASSUMED] Mỗi tài khoản đại diện cho MỘT tổ chức duy nhất; hệ thống không quản lý
    thành viên nội bộ, tài khoản con, hoặc phân quyền nhiều người dùng trong cùng tổ chức.
  - [ASSUMED] Vai trò (Organizer / Sponsor) được chọn khi đăng ký và KHÔNG THỂ thay đổi
    sau khi tài khoản được tạo.
  - [ASSUMED] Mỗi email chỉ được liên kết với MỘT tài khoản duy nhất trên hệ thống.
  - [ASSUMED] Google OAuth sử dụng chuẩn OAuth 2.0 / OpenID Connect.
  - [ASSUMED] Mật khẩu được lưu trữ dưới dạng hash (bcrypt hoặc tương đương), không lưu plaintext.
  - [ASSUMED] Reset password sử dụng token gửi qua email, có thời hạn hiệu lực.
Gaps Detected:
  - BP02 không nêu chính sách mật khẩu (độ dài tối thiểu, ký tự đặc biệt) → cần bổ sung BR.
  - BP02 không đề cập cơ chế khóa tài khoản khi đăng nhập sai nhiều lần → cần đánh giá.
  - BP02 không nêu cơ chế xác minh email (email verification) khi đăng ký → cần bổ sung.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Người đăng ký mới (chưa có tài khoản) | `guest` | Tạo tài khoản mới, đăng ký qua Google |
| Người dùng đã có tài khoản | `authenticated_user` | Đăng nhập, đặt lại mật khẩu |
| Hệ thống | `system` | Xác thực danh tính, gửi email xác minh, gửi token reset password, ghi log |

---

## Functional Requirements

### FR-0801: Đăng ký tài khoản bằng email

```
ID:            FR-0801
Name:          Đăng ký tài khoản mới bằng email và mật khẩu
Description:   Hệ thống SHALL cho phép người dùng mới tạo tài khoản bằng cách cung cấp
               địa chỉ email và mật khẩu. Hệ thống SHALL xác thực email chưa tồn tại trong
               hệ thống (BR-0801), mật khẩu đạt yêu cầu chính sách (BR-0802), và gửi email
               xác minh địa chỉ email (BR-0803). Tài khoản có trạng thái ban đầu là UNVERIFIED.
Classification: SYSTEM-SUPPORTED
Actor:         Guest (khởi tạo)
Trigger:       Người dùng chọn "Đăng ký tài khoản" trên trang đăng ký
Inputs:        email (string), password (string), password_confirmation (string)
Outputs:       account_id (UUID), status = UNVERIFIED, created_at (timestamp),
               verification_email (gửi tới email đăng ký)
Business Rules: BR-0801, BR-0802, BR-0803
Acceptance Criteria:
  Given   người dùng truy cập trang đăng ký
  When    người dùng nhập email = "btc@example.com", password đạt yêu cầu BR-0802
  Then    hệ thống SHALL tạo tài khoản mới với status = UNVERIFIED
  And     hệ thống SHALL gửi email xác minh tới "btc@example.com"
  And     hệ thống SHALL chuyển người dùng sang bước chọn vai trò (FR-0804)

  Given   email "btc@example.com" đã tồn tại trong hệ thống
  When    người dùng cố đăng ký với email này
  Then    hệ thống SHALL từ chối với thông báo "Email này đã được sử dụng"

  Given   người dùng nhập password không đạt yêu cầu BR-0802
  When    người dùng submit form đăng ký
  Then    hệ thống SHALL hiển thị lỗi cụ thể về yêu cầu mật khẩu chưa đạt
Priority:      MUST
```

### FR-0802: Đăng ký và Đăng nhập bằng Google OAuth

```
ID:            FR-0802
Name:          Đăng ký và đăng nhập nhanh bằng tài khoản Google
Description:   Hệ thống SHALL cho phép người dùng đăng ký hoặc đăng nhập nhanh bằng tài khoản
               Google thông qua OAuth 2.0 / OpenID Connect. Nếu email Google chưa tồn tại trên
               hệ thống, hệ thống SHALL tạo tài khoản mới và yêu cầu hoàn tất thông tin tổ chức
               (FR-0804, FR-0805) trước khi sử dụng đầy đủ. Nếu email đã liên kết với tài khoản
               hiện có, hệ thống SHALL đăng nhập trực tiếp.
Classification: SYSTEM-SUPPORTED
Actor:         Guest (đăng ký mới) hoặc Authenticated User (đăng nhập)
Trigger:       Người dùng nhấn "Tiếp tục bằng Google" trên trang đăng ký/đăng nhập
Inputs:        Google OAuth token (từ Google Identity Provider)
Outputs:       account_id (UUID), email (từ Google profile), display_name (từ Google profile),
               auth_method = GOOGLE, session_token
Business Rules: BR-0801, BR-0804
Acceptance Criteria:
  Given   người dùng chưa có tài khoản trên UniLink
  When    người dùng nhấn "Tiếp tục bằng Google" và xác thực thành công với Google
  Then    hệ thống SHALL tạo tài khoản mới với email từ Google profile
  And     hệ thống SHALL đánh dấu email đã xác minh (email_verified = true)
  And     hệ thống SHALL chuyển người dùng sang bước chọn vai trò (FR-0804)

  Given   người dùng đã có tài khoản liên kết với Google email
  When    người dùng nhấn "Tiếp tục bằng Google"
  Then    hệ thống SHALL đăng nhập và tạo session_token
  And     hệ thống SHALL chuyển đến trang chính phù hợp với vai trò

  Given   Google OAuth trả về lỗi hoặc người dùng hủy xác thực
  When    hệ thống nhận phản hồi lỗi
  Then    hệ thống SHALL hiển thị thông báo "Đăng nhập bằng Google không thành công, vui lòng thử lại"
Priority:      MUST
```

### FR-0803: Đăng nhập hệ thống

```
ID:            FR-0803
Name:          Đăng nhập bằng email và mật khẩu
Description:   Hệ thống SHALL cho phép người dùng đã có tài khoản đăng nhập bằng email và mật
               khẩu. Hệ thống SHALL xác thực thông tin đăng nhập, tạo session token khi thành
               công, và ghi nhận thời gian đăng nhập. Hệ thống SHALL áp dụng cơ chế bảo vệ
               chống brute-force theo BR-0805.
Classification: SYSTEM-SUPPORTED
Actor:         Guest (khởi tạo)
Trigger:       Người dùng submit form đăng nhập
Inputs:        email (string), password (string)
Outputs:       session_token, account_id, role, verification_status, last_login_at (timestamp)
Business Rules: BR-0805
Acceptance Criteria:
  Given   tài khoản "btc@example.com" tồn tại với mật khẩu chính xác
  When    người dùng nhập email và mật khẩu đúng
  Then    hệ thống SHALL tạo session_token và chuyển đến trang chính
  And     hệ thống SHALL cập nhật last_login_at

  Given   người dùng nhập sai mật khẩu
  When    người dùng submit form đăng nhập
  Then    hệ thống SHALL từ chối với thông báo "Email hoặc mật khẩu không chính xác"
  And     hệ thống SHALL KHÔNG tiết lộ email có tồn tại hay không

  Given   tài khoản đã bị khóa do đăng nhập sai quá số lần cho phép (BR-0805)
  When    người dùng cố đăng nhập
  Then    hệ thống SHALL từ chối và thông báo "Tài khoản tạm thời bị khóa, vui lòng thử lại sau [X] phút"
Priority:      MUST
```

### FR-0804: Chọn vai trò tổ chức

```
ID:            FR-0804
Name:          Lựa chọn vai trò đại diện tổ chức
Description:   Hệ thống SHALL yêu cầu người dùng chọn vai trò đại diện cho tổ chức ngay sau khi
               tạo tài khoản: Câu lạc bộ/BTC (Organizer) hoặc Doanh nghiệp/Nhà tài trợ (Sponsor).
               Vai trò này SHALL được gán vĩnh viễn và KHÔNG THỂ thay đổi sau khi xác nhận
               (BR-0806). Hệ thống SHALL điều hướng đến form nhập thông tin tổ chức tương ứng
               (FR-0805) sau khi chọn vai trò.
Classification: SYSTEM-SUPPORTED
Actor:         Guest (vừa tạo tài khoản)
Trigger:       Hoàn tất đăng ký (FR-0801) hoặc đăng ký qua Google lần đầu (FR-0802)
Inputs:        role (enum: ORGANIZER | SPONSOR)
Outputs:       account đã cập nhật role, redirect đến form thông tin tổ chức
Business Rules: BR-0806
Acceptance Criteria:
  Given   người dùng vừa tạo tài khoản và chưa chọn vai trò
  When    người dùng chọn "Câu lạc bộ/BTC"
  Then    hệ thống SHALL gán role = ORGANIZER cho tài khoản
  And     hệ thống SHALL chuyển đến form "Thông tin Câu lạc bộ/BTC" (FR-0805)

  Given   người dùng vừa tạo tài khoản và chưa chọn vai trò
  When    người dùng chọn "Doanh nghiệp/Nhà tài trợ"
  Then    hệ thống SHALL gán role = SPONSOR cho tài khoản
  And     hệ thống SHALL chuyển đến form "Thông tin Doanh nghiệp" (FR-0805)

  Given   tài khoản đã có role = ORGANIZER
  When    người dùng cố thay đổi vai trò sang SPONSOR
  Then    hệ thống SHALL từ chối "Vai trò tổ chức không thể thay đổi sau khi đăng ký"
Priority:      MUST
```

### FR-0805: Nhập thông tin cơ bản tổ chức

```
ID:            FR-0805
Name:          Nhập thông tin cơ bản của tổ chức theo vai trò
Description:   Hệ thống SHALL hiển thị form nhập thông tin cơ bản tương ứng với vai trò đã chọn.
               Đối với Organizer (CLB/BTC): tên câu lạc bộ/BTC, tên trường, địa chỉ liên hệ.
               Đối với Sponsor (Doanh nghiệp): tên công ty, lĩnh vực hoạt động, địa chỉ liên hệ.
               Hệ thống SHALL xác thực tất cả trường bắt buộc theo BR-0807 trước khi cho phép
               hoàn tất đăng ký.
Classification: SYSTEM-SUPPORTED
Actor:         Guest (đang trong quy trình đăng ký)
Trigger:       Hoàn tất chọn vai trò (FR-0804)
Inputs:
  - Organizer: org_name (string), university_name (string), contact_address (string)
  - Sponsor:   company_name (string), business_sector (string), contact_address (string)
Outputs:       Organization entity đã tạo, tài khoản hoàn tất đăng ký cơ bản
Business Rules: BR-0807
Acceptance Criteria:
  Given   người dùng đã chọn vai trò ORGANIZER
  When    người dùng nhập tên CLB = "CLB IT HCMUS", trường = "ĐH KHTN TPHCM",
          địa chỉ = "227 Nguyễn Văn Cừ, Q5"
  Then    hệ thống SHALL tạo Organization entity liên kết với tài khoản
  And     hệ thống SHALL chuyển đến trang chính với thông báo "Đăng ký thành công"
  And     hệ thống SHALL gửi email thông báo đăng ký thành công

  Given   người dùng đã chọn vai trò SPONSOR
  When    người dùng nhập tên công ty = "ABC Corp", lĩnh vực = "Công nghệ",
          địa chỉ = "123 Nguyễn Huệ, Q1"
  Then    hệ thống SHALL tạo Organization entity liên kết với tài khoản
  And     hệ thống SHALL chuyển đến trang chính

  Given   người dùng bỏ trống trường bắt buộc (ví dụ: tên tổ chức)
  When    người dùng submit form
  Then    hệ thống SHALL từ chối với thông báo lỗi cụ thể cho từng trường thiếu
Priority:      MUST
```

### FR-0806: Đặt lại mật khẩu

```
ID:            FR-0806
Name:          Đặt lại mật khẩu qua email
Description:   Hệ thống SHALL cho phép người dùng yêu cầu đặt lại mật khẩu khi quên. Hệ thống
               SHALL gửi email chứa đường dẫn reset password với token có thời hạn theo BR-0808.
               Khi người dùng truy cập đường dẫn hợp lệ, hệ thống SHALL cho phép nhập mật khẩu
               mới và cập nhật mật khẩu. Hệ thống SHALL vô hiệu hóa tất cả session hiện tại
               sau khi đổi mật khẩu thành công.
Classification: SYSTEM-SUPPORTED
Actor:         Guest (chưa đăng nhập được)
Trigger:       Người dùng nhấn "Quên mật khẩu" trên trang đăng nhập
Inputs:        email (string) — cho bước yêu cầu;
               reset_token (string), new_password (string), confirm_password (string) — cho bước đổi
Outputs:       Reset email (gửi tới email đăng ký), mật khẩu đã cập nhật,
               tất cả session cũ bị vô hiệu hóa
Business Rules: BR-0802, BR-0808
Acceptance Criteria:
  Given   tài khoản "btc@example.com" tồn tại
  When    người dùng nhập email và yêu cầu đặt lại mật khẩu
  Then    hệ thống SHALL gửi email chứa đường dẫn reset password
  And     hệ thống SHALL hiển thị "Vui lòng kiểm tra email để đặt lại mật khẩu"

  Given   email "unknown@example.com" không tồn tại
  When    người dùng yêu cầu reset password
  Then    hệ thống SHALL vẫn hiển thị thông báo tương tự (không tiết lộ email tồn tại hay không)

  Given   người dùng truy cập đường dẫn reset hợp lệ (token chưa hết hạn)
  When    người dùng nhập mật khẩu mới đạt yêu cầu BR-0802
  Then    hệ thống SHALL cập nhật mật khẩu thành công
  And     hệ thống SHALL vô hiệu hóa tất cả session cũ
  And     hệ thống SHALL chuyển đến trang đăng nhập với thông báo "Đổi mật khẩu thành công"

  Given   token reset password đã hết hạn (quá 24 giờ)
  When    người dùng truy cập đường dẫn reset
  Then    hệ thống SHALL từ chối "Đường dẫn đã hết hạn, vui lòng yêu cầu lại"
Priority:      MUST
```

---

## Business Rules

```
ID:          BR-0801
Rule:        Mỗi email chỉ được liên kết với MỘT tài khoản duy nhất trên hệ thống.
             UNIQUE(email). Kiểm tra khi đăng ký bằng email và khi đăng ký qua Google OAuth.
Source:      BP02 — Bước 1 (Tạo tài khoản)
Type:        Validation

ID:          BR-0802
Rule:        Mật khẩu PHẢI đạt yêu cầu tối thiểu: ít nhất 8 ký tự, bao gồm ít nhất
             1 chữ hoa, 1 chữ thường, và 1 chữ số. [ASSUMED — BP02 không nêu chính sách cụ thể]
Source:      [INFERRED — chính sách bảo mật tiêu chuẩn]
Type:        Validation

ID:          BR-0803
Rule:        Sau khi đăng ký bằng email, hệ thống PHẢI gửi email xác minh chứa đường dẫn
             kích hoạt. Đường dẫn có hiệu lực trong 24 giờ. Tài khoản chưa xác minh email
             vẫn có thể đăng nhập nhưng bị giới hạn chức năng (không thể gửi hồ sơ xác thực).
             [ASSUMED — BP02 không đề cập nhưng cần thiết cho bảo mật]
Source:      [INFERRED — xác minh email tiêu chuẩn]
Type:        Validation + Time-based

ID:          BR-0804
Rule:        Khi đăng ký qua Google OAuth, email từ Google profile được coi là ĐÃ XÁC MINH
             (email_verified = true) mà không cần xác minh lại qua email.
Source:      BP02 — Bước 1 (Đăng ký bằng Google)
Type:        Validation

ID:          BR-0805
Rule:        Sau 5 lần đăng nhập sai liên tiếp, tài khoản bị TẠM KHÓA trong 15 phút.
             Bộ đếm reset khi đăng nhập thành công hoặc sau thời gian khóa.
             [ASSUMED — BP02 không đề cập nhưng cần thiết cho bảo mật]
Source:      [INFERRED — chống brute-force tiêu chuẩn]
Type:        Validation + Time-based

ID:          BR-0806
Rule:        Vai trò tổ chức (ORGANIZER | SPONSOR) được chọn MỘT LẦN khi đăng ký và
             KHÔNG THỂ thay đổi sau khi xác nhận. Hệ thống KHÔNG cung cấp cơ chế chuyển đổi vai trò.
Source:      BP02 — Bước 2 (Sau khi tài khoản được tạo, vai trò này không được phép thay đổi)
Type:        Authorization

ID:          BR-0807
Rule:        Thông tin cơ bản bắt buộc theo vai trò:
             - Organizer: tên CLB/BTC (required), tên trường (required), địa chỉ liên hệ (required)
             - Sponsor: tên công ty (required), lĩnh vực hoạt động (required), địa chỉ liên hệ (required)
             Tất cả trường bắt buộc PHẢI được nhập trước khi hoàn tất đăng ký.
Source:      BP02 — Bước 2.1 và 2.2 (Thông tin bắt buộc)
Type:        Validation

ID:          BR-0808
Rule:        Token reset password có hiệu lực trong 15 phút kể từ khi tạo.
             Mỗi yêu cầu reset mới sẽ VÔ HIỆU HÓA token cũ (nếu có).
             Tối đa 3 yêu cầu reset password trong 1 giờ cho mỗi email. [ASSUMED]
Source:      [INFERRED — bảo mật tiêu chuẩn cho reset password]
Type:        Validation + Time-based
```

---

## Data Model

```
Entity:        Account
Attributes:
  - account_id: UUID (PK)
  - email: String (required, unique)
  - password_hash: String (nullable — null khi đăng ký qua Google)
  - auth_method: Enum [EMAIL, GOOGLE] (required)
  - google_sub: String (nullable — Google subject ID, unique khi có giá trị)
  - role: Enum [ORGANIZER, SPONSOR] (required sau khi chọn vai trò)
  - email_verified: Boolean (default: false)
  - verification_status: Enum [UNVERIFIED, PENDING_REVIEW, VERIFIED, REJECTED, INFO_REQUIRED]
                         (default: UNVERIFIED)
  - failed_login_attempts: Integer (default: 0)
  - locked_until: DateTime (nullable)
  - last_login_at: DateTime (nullable)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - Account —(1:1)→ Organization
Constraints:
  - UNIQUE(email)
  - UNIQUE(google_sub) WHERE google_sub IS NOT NULL

Entity:        Organization
Attributes:
  - org_id: UUID (PK)
  - account_id: UUID (FK → Account, unique)
  - org_type: Enum [CLUB_BTC, BUSINESS] (mapped from Account.role)
  - org_name: String (required)
  - university_name: String (required for CLUB_BTC, nullable for BUSINESS)
  - business_sector: String (required for BUSINESS, nullable for CLUB_BTC)
  - contact_address: String (required)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - Organization —(1:1)→ Account
  - Organization —(1:N)→ VerificationDocument (SF-09)
  - Organization —(1:N)→ SponsorshipProposal (SF-01, chỉ cho CLUB_BTC)

Entity:        PasswordResetToken
Attributes:
  - token_id: UUID (PK)
  - account_id: UUID (FK → Account)
  - token_hash: String (required)
  - expires_at: DateTime (required)
  - used: Boolean (default: false)
  - created_at: DateTime
Relationships:
  - PasswordResetToken —(N:1)→ Account

Entity:        EmailVerificationToken
Attributes:
  - token_id: UUID (PK)
  - account_id: UUID (FK → Account)
  - token_hash: String (required)
  - expires_at: DateTime (required)
  - used: Boolean (default: false)
  - created_at: DateTime
Relationships:
  - EmailVerificationToken —(N:1)→ Account
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 1. Tạo tài khoản bằng email | SYSTEM-SUPPORTED | FR-0801 | BR-0801, BR-0802, BR-0803 | Account, EmailVerificationToken |
| 1. Đăng ký/Đăng nhập bằng Google | SYSTEM-SUPPORTED | FR-0802 | BR-0801, BR-0804 | Account |
| 1. Đăng nhập hệ thống | SYSTEM-SUPPORTED | FR-0803 | BR-0805 | Account |
| 2. Chọn vai trò tổ chức | SYSTEM-SUPPORTED | FR-0804 | BR-0806 | Account |
| 2. Nhập thông tin cơ bản tổ chức | SYSTEM-SUPPORTED | FR-0805 | BR-0807 | Organization |
| [INFERRED] Đặt lại mật khẩu | SYSTEM-SUPPORTED | FR-0806 | BR-0802, BR-0808 | Account, PasswordResetToken |
