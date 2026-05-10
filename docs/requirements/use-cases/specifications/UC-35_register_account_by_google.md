# Use-Case Specification: UC-35 — Đăng ký tài khoản bằng Google

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Guest | Người dùng chưa có tài khoản, sử dụng Google để đăng ký |
| Secondary | System | Xác thực OAuth, tạo tài khoản |
| Secondary | Google Identity Provider | Cung cấp xác thực OAuth 2.0 / OpenID Connect |

---

### 1. Brief Description

> Guest tạo tài khoản mới trên hệ thống UniLink thông qua xác thực Google OAuth 2.0. Quy trình đăng ký bao gồm: (1) xác thực với Google, (2) chọn vai trò tổ chức, (3) nhập thông tin cơ bản tổ chức. Email từ Google được coi là đã xác minh, không cần gửi email verification riêng.

---

### 2. Flow of Events

**Trigger**
> Guest nhấn "Tiếp tục bằng Google" trên trang đăng ký.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Guest | Nhấn "Tiếp tục bằng Google" trên trang đăng ký |
| 2 | System | Chuyển hướng đến trang xác thực Google OAuth |
| 3 | Guest | Xác thực tài khoản Google và cấp quyền truy cập thông tin cơ bản |
| 4 | System | Nhận Google OAuth token và trích xuất thông tin: email, display_name |
| 5 | System | Xác thực email từ Google chưa tồn tại trong hệ thống (BR-0801) |
| 6 | System | Tạo tài khoản mới với auth_method = GOOGLE, email_verified = true (BR-0804) |
| 7 | System | Chuyển sang bước chọn vai trò tổ chức |
| 8 | Guest | Chọn vai trò đại diện: "Câu lạc bộ/BTC" (Organizer) hoặc "Doanh nghiệp/Nhà tài trợ" (Sponsor) |
| 9 | System | Gán vai trò vĩnh viễn cho tài khoản (BR-0806) |
| 10 | System | Hiển thị form nhập thông tin cơ bản tổ chức tương ứng với vai trò đã chọn |
| 11 | Guest | Nhập thông tin cơ bản bắt buộc theo vai trò (BR-0807) |
| 12 | Guest | Nhấn "Hoàn tất đăng ký" |
| 13 | System | Xác thực tất cả trường bắt buộc đã được nhập |
| 14 | System | Tạo Organization entity liên kết với tài khoản |
| 15 | System | Gửi email thông báo đăng ký thành công |
| 16 | System | Chuyển Guest (nay là Authenticated User) đến trang chính với thông báo "Đăng ký thành công" |
| 17 | System | Use case kết thúc thành công — tài khoản đã được tạo với email đã xác minh |

#### 2.2 Alternate Flows

##### AF-35.a: Guest chọn vai trò Organizer
>
> *Triggered at Step 8 of the Basic Flow when guest chọn "Câu lạc bộ/BTC".*

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | Guest | Chọn "Câu lạc bộ/BTC" |
| 10a | System | Hiển thị form với các trường: Tên CLB/BTC (bắt buộc), Tên trường (bắt buộc), Địa chỉ liên hệ (bắt buộc) |
| – | – | Tiếp tục từ Step 11 |

##### AF-35.b: Guest chọn vai trò Sponsor
>
> *Triggered at Step 8 of the Basic Flow when guest chọn "Doanh nghiệp/Nhà tài trợ".*

| Step | Actor / System | Action |
|------|----------------|--------|
| 8b | Guest | Chọn "Doanh nghiệp/Nhà tài trợ" |
| 10b | System | Hiển thị form với các trường: Tên công ty (bắt buộc), Lĩnh vực hoạt động (bắt buộc), Địa chỉ liên hệ (bắt buộc) |
| – | – | Tiếp tục từ Step 11 |

##### AF-35.c: Email Google đã liên kết với tài khoản hiện có
>
> *Triggered at Step 5 of the Basic Flow when email đã tồn tại trên hệ thống.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện email từ Google đã liên kết với tài khoản trên hệ thống |
| 5b | System | Đăng nhập trực tiếp và tạo session_token (chuyển thành luồng đăng nhập) |
| 5c | System | Chuyển đến trang chính phù hợp với vai trò — use case kết thúc |

#### 2.3 Exception Flows

##### EF-35.1: Google OAuth thất bại
>
> *Triggered at Step 4 of the Basic Flow when nhận phản hồi lỗi từ Google.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Nhận phản hồi lỗi từ Google hoặc Guest hủy xác thực |
| 4b | System | Hiển thị thông báo "Đăng nhập bằng Google không thành công, vui lòng thử lại" |
| 4c | Guest | Có thể thử lại hoặc chọn đăng ký bằng email (UC-34) |

##### EF-35.2: Thiếu thông tin bắt buộc của tổ chức
>
> *Triggered at Step 13 of the Basic Flow when trường bắt buộc chưa được nhập.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 13a | System | Phát hiện một hoặc nhiều trường bắt buộc chưa được nhập |
| 13b | System | Hiển thị thông báo lỗi cụ thể cho từng trường thiếu |
| 13c | Guest | Bổ sung thông tin thiếu và thử lại |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-35-01 | Đăng ký Google Organizer | Guest đăng ký qua Google, chọn Organizer, nhập thông tin CLB/BTC |
| SC-35-02 | Đăng ký Google Sponsor | Guest đăng ký qua Google, chọn Sponsor, nhập thông tin doanh nghiệp |
| SC-35-03 | Tài khoản đã tồn tại | Email Google đã liên kết; tự động chuyển sang đăng nhập (AF-35.c) |

---

### 5. Preconditions

#### 5.1 Chưa có tài khoản

- Guest chưa có tài khoản trên hệ thống UniLink

#### 5.2 Có tài khoản Google

- Guest có tài khoản Google hợp lệ

#### 5.3 Đang ở trang đăng ký

- Guest đang truy cập trang đăng ký của hệ thống

---

### 6. Postconditions

#### 6.1 Success

- Tài khoản mới được tạo với auth_method = GOOGLE, email_verified = true
- Vai trò tổ chức (Organizer/Sponsor) đã được gán vĩnh viễn
- Organization entity đã được tạo với thông tin cơ bản
- Guest trở thành Authenticated User và ở trang chính

#### 6.2 Failure

- Không có tài khoản nào được tạo
- Guest được thông báo về lỗi và ở lại trang đăng ký

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0801: Mỗi email chỉ được liên kết với MỘT tài khoản duy nhất
- BR-0804: Email từ Google profile được coi là ĐÃ XÁC MINH (email_verified = true)
- BR-0806: Vai trò tổ chức chọn MỘT LẦN, không thể thay đổi
- BR-0807: Thông tin bắt buộc theo vai trò (Organizer: tên CLB, tên trường, địa chỉ; Sponsor: tên công ty, lĩnh vực, địa chỉ)

---

### 10. Additional Information

**Assumptions:**

- Khác với UC-34 (email), đăng ký qua Google không cần gửi email verification riêng — email đã được Google xác minh
- AF-35.c xử lý trường hợp tài khoản đã tồn tại: hệ thống tự động chuyển sang luồng đăng nhập (UC-36)
- Sau khi đăng ký, tài khoản cần hoàn tất thêm thông tin tùy chọn (UC-38) và gửi hồ sơ xác thực (UC-40)

**Related Use Cases:**

- UC-34: Đăng ký tài khoản bằng email (luồng thay thế cùng mục tiêu)
- UC-36: Đăng nhập hệ thống (sequential — AF-35.c chuyển sang đăng nhập nếu email đã tồn tại)
- UC-38: Bổ sung thông tin tổ chức (sequential — hoàn thiện hồ sơ)
- UC-40: Gửi hồ sơ xác thực (sequential — sau khi hoàn thiện hồ sơ)
