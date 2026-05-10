# Use-Case Specification: UC-40 — Gửi hồ sơ xác thực

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người gửi hồ sơ xác thực |
| Secondary | System | Xác thực đầy đủ, chuyển trạng thái, gửi thông báo |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) gửi hồ sơ tổ chức để Admin kiểm duyệt và xác thực. Hệ thống kiểm tra tính đầy đủ thông tin bắt buộc, chuyển trạng thái tài khoản sang PENDING_REVIEW, và gửi thông báo xác nhận tiếp nhận qua email và in-app.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Gửi hồ sơ xác thực" trên trang hồ sơ tổ chức.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Truy cập trang "Hồ sơ tổ chức" |
| 2 | System | Hiển thị hồ sơ tổ chức với nút "Gửi hồ sơ xác thực" |
| 3 | Authenticated User | Nhấn "Gửi hồ sơ xác thực" |
| 4 | System | Xác thực email đã xác minh (email_verified = true) |
| 5 | System | Xác thực tất cả thông tin bắt buộc đã đầy đủ (BR-0904) |
| 6 | System | Xác thực trạng thái cho phép gửi (BR-0905) |
| 7 | System | Tạo VerificationRequest mới với status = PENDING |
| 8 | System | Chuyển verification_status sang PENDING_REVIEW |
| 9 | System | Ghi nhận submitted_at và thiết lập deadline_at = submitted_at + 14 ngày |
| 10 | System | Gửi email + in-app "Hồ sơ xác thực đã được tiếp nhận" cho người dùng |
| 11 | System | Gửi thông báo cho Admin (in-app + email): "Có hồ sơ xác thực mới từ [tên tổ chức]" |
| 12 | System | Hiển thị xác nhận "Hồ sơ đã được gửi và đang chờ kiểm duyệt" |
| 13 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-40.a: Gửi lại sau REJECTED

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện verification_status = REJECTED. Cho phép gửi — tạo VerificationRequest mới liên kết lịch sử |
| – | – | Tiếp tục từ Step 7 |

##### AF-40.b: Gửi lại sau INFO_REQUIRED

| Step | Actor / System | Action |
|------|----------------|--------|
| 6c | System | Phát hiện verification_status = INFO_REQUIRED. Cho phép gửi |
| – | – | Tiếp tục từ Step 7 |

#### 2.3 Exception Flows

##### EF-40.1: Thông tin bắt buộc thiếu

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Hiển thị danh sách thông tin còn thiếu |
| 5b | Authenticated User | Bổ sung thông tin (UC-38/UC-39) rồi thử gửi lại |

##### EF-40.2: Email chưa xác minh

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Hiển thị "Vui lòng xác minh email trước khi gửi hồ sơ xác thực" |

##### EF-40.3: Trạng thái không cho phép gửi

| Step | Actor / System | Action |
|------|----------------|--------|
| 6e | System | verification_status = PENDING_REVIEW → "Hồ sơ đang chờ kiểm duyệt" |

##### EF-40.4: Tài khoản đã VERIFIED

| Step | Actor / System | Action |
|------|----------------|--------|
| 6g | System | Hiển thị "Tài khoản đã được xác thực" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-40-01 | Gửi lần đầu thành công | Actor gửi hồ sơ đầy đủ; chuyển sang PENDING_REVIEW |
| SC-40-02 | Gửi lại sau REJECTED | Actor chỉnh sửa và gửi lại; tạo VerificationRequest mới (AF-40.a) |
| SC-40-03 | Thiếu thông tin | Hồ sơ chưa đủ; hiển thị danh sách thiếu (EF-40.1) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Email đã xác minh

- email_verified = true

#### 5.3 Hồ sơ đầy đủ

- Hồ sơ tổ chức đã có đầy đủ thông tin bắt buộc

#### 5.4 Trạng thái cho phép gửi

- verification_status là UNVERIFIED, REJECTED, hoặc INFO_REQUIRED

---

### 6. Postconditions

#### 6.1 Success

- VerificationRequest mới với status = PENDING
- verification_status = PENDING_REVIEW
- Actor nhận thông báo email + in-app
- Admin nhận thông báo (in-app + email)

#### 6.2 Failure

- Không có VerificationRequest nào được tạo
- verification_status không thay đổi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0904: Hồ sơ chỉ được gửi khi đầy đủ thông tin bắt buộc (BR-0807) và email đã xác minh
- BR-0905: Chỉ cho phép gửi khi status là UNVERIFIED/REJECTED/INFO_REQUIRED. Không cho phép khi PENDING_REVIEW hoặc VERIFIED

---

### 10. Additional Information

**Assumptions:**

- Tổ chức có thể gửi hồ sơ nhiều lần (sau REJECTED hoặc INFO_REQUIRED) — KHÔNG giới hạn số lần
- Tài liệu minh chứng sẽ được xóa tự động sau 7 ngày kể từ xử lý hoàn tất (FR-0905)

**Related Use Cases:**

- UC-38: Bổ sung thông tin (prerequisite — thông tin đầy đủ trước khi gửi)
- UC-39: Chỉnh sửa hồ sơ (sequential — sửa trước khi gửi lại)
- UC-41: Xem danh sách hồ sơ chờ duyệt (sequential — Admin xem)
