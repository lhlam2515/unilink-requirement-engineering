# UC-40: Gửi hồ sơ xác thực

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) gửi hồ sơ tổ chức để Admin kiểm duyệt và xác thực. Hệ thống kiểm tra tính đầy đủ thông tin bắt buộc, chuyển trạng thái tài khoản sang PENDING_REVIEW, và gửi thông báo xác nhận tiếp nhận qua email và in-app.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người gửi hồ sơ xác thực |
| Secondary | System | Xác thực đầy đủ, chuyển trạng thái, gửi thông báo |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Email của actor đã được xác minh (email_verified = true)
- Hồ sơ tổ chức đã có đầy đủ thông tin bắt buộc
- verification_status hiện tại là UNVERIFIED, REJECTED, hoặc INFO_REQUIRED

---

**Trigger**
> Actor nhấn "Gửi hồ sơ xác thực" trên trang hồ sơ tổ chức.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Truy cập trang "Hồ sơ tổ chức" |
| 2 | System | Hiển thị hồ sơ tổ chức với nút "Gửi hồ sơ xác thực" |
| 3 | Authenticated User | Nhấn "Gửi hồ sơ xác thực" |
| 4 | System | Xác thực email đã xác minh (email_verified = true) |
| 5 | System | Xác thực tất cả thông tin bắt buộc đã đầy đủ (BR-0904) |
| 6 | System | Xác thực trạng thái cho phép gửi (BR-0905) |
| 7 | System | Tạo VerificationRequest mới với status = PENDING |
| 8 | System | Chuyển verification_status của tài khoản sang PENDING_REVIEW |
| 9 | System | Ghi nhận submitted_at và thiết lập deadline_at = submitted_at + 14 ngày |
| 10 | System | Gửi email "Hồ sơ xác thực của bạn đã được tiếp nhận" |
| 11 | System | Gửi thông báo in-app tương tự |
| 12 | System | Hiển thị xác nhận "Hồ sơ đã được gửi và đang chờ kiểm duyệt" |
| 13 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-40.a: Gửi lại hồ sơ sau khi bị REJECTED (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện verification_status = REJECTED |
| 6b | System | Cho phép gửi — tạo VerificationRequest mới (liên kết với lịch sử lần gửi trước) |
| – | – | Tiếp tục từ Step 7 |

> AF-40.b: Gửi lại hồ sơ sau khi INFO_REQUIRED (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6c | System | Phát hiện verification_status = INFO_REQUIRED |
| 6d | System | Cho phép gửi — tạo VerificationRequest mới |
| – | – | Tiếp tục từ Step 7 |

---

**Exception Flows**

> EF-40.1: Thông tin bắt buộc thiếu (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện thông tin bắt buộc chưa đầy đủ |
| 5b | System | Hiển thị danh sách thông tin còn thiếu |
| 5c | Authenticated User | Bổ sung thông tin (UC-38 hoặc UC-39) rồi thử gửi lại |

> EF-40.2: Email chưa xác minh (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện email_verified = false |
| 4b | System | Hiển thị "Vui lòng xác minh email trước khi gửi hồ sơ xác thực" |
| 4c | System | Cung cấp tùy chọn gửi lại email xác minh |

> EF-40.3: Trạng thái không cho phép gửi (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6e | System | Phát hiện verification_status = PENDING_REVIEW |
| 6f | System | Hiển thị "Hồ sơ đang chờ kiểm duyệt. Bạn không thể gửi lại lúc này." |

> EF-40.4: Tài khoản đã VERIFIED (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6g | System | Phát hiện verification_status = VERIFIED |
| 6h | System | Hiển thị "Tài khoản đã được xác thực" |

---

**Postconditions**

*Success:*

- VerificationRequest mới đã được tạo với status = PENDING
- verification_status đã chuyển sang PENDING_REVIEW
- submitted_at và deadline_at đã được ghi nhận
- Actor nhận thông báo xác nhận qua email và in-app
- Hồ sơ xuất hiện trong danh sách chờ duyệt của Admin (UC-41)

*Failure:*

- Không có VerificationRequest nào được tạo
- verification_status không thay đổi
- Actor được thông báo lý do không thể gửi

---

**Business Rules**

- BR-0904: Hồ sơ chỉ được gửi khi đầy đủ thông tin bắt buộc (BR-0807) và email đã xác minh
- BR-0905: Chỉ cho phép gửi khi status là UNVERIFIED/REJECTED/INFO_REQUIRED. Không cho phép khi PENDING_REVIEW hoặc VERIFIED

---

**Notes / Assumptions**

- Tổ chức có thể gửi hồ sơ xác thực nhiều lần (sau REJECTED hoặc INFO_REQUIRED) — mỗi lần tạo VerificationRequest mới
- Phân quyền tự động (FR-0904) được kích hoạt ngầm sau khi hồ sơ được phê duyệt — không phải hành vi trực tiếp trong UC này
- Tài liệu minh chứng sẽ được tự động xóa sau 7 ngày kể từ xử lý hoàn tất (FR-0905 — nhúng vào system behavior)
- Liên kết: UC-38 (bổ sung tài liệu), UC-39 (chỉnh sửa), UC-41 (Admin xem danh sách)
