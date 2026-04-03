# UC-34: Đăng ký tài khoản bằng email

**Brief Description**
> Guest tạo tài khoản mới trên hệ thống UniLink bằng email và mật khẩu. Quy trình đăng ký bao gồm ba bước liên tiếp trong cùng phiên: (1) nhập email/mật khẩu, (2) chọn vai trò tổ chức, (3) nhập thông tin cơ bản tổ chức. Sau khi hoàn tất, hệ thống gửi email xác minh và tạo tài khoản với trạng thái UNVERIFIED.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Guest | Người dùng chưa có tài khoản trên hệ thống |
| Secondary | System | Xác thực dữ liệu, tạo tài khoản, gửi email xác minh |

---

**Preconditions**

- Guest chưa có tài khoản trên hệ thống UniLink
- Guest đang truy cập trang đăng ký của hệ thống

---

**Trigger**
> Guest chọn chức năng "Đăng ký tài khoản" trên trang đăng ký.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Guest | Nhập địa chỉ email, mật khẩu và xác nhận mật khẩu |
| 2 | Guest | Nhấn "Đăng ký" |
| 3 | System | Xác thực email chưa tồn tại trong hệ thống (BR-0801) |
| 4 | System | Xác thực mật khẩu đạt yêu cầu chính sách (BR-0802) |
| 5 | System | Xác thực mật khẩu và xác nhận mật khẩu khớp nhau |
| 6 | System | Tạo tài khoản mới với status = UNVERIFIED, auth_method = EMAIL |
| 7 | System | Gửi email xác minh chứa đường dẫn kích hoạt (BR-0803) |
| 8 | System | Chuyển sang bước chọn vai trò tổ chức |
| 9 | Guest | Chọn vai trò đại diện: "Câu lạc bộ/BTC" (Organizer) hoặc "Doanh nghiệp/Nhà tài trợ" (Sponsor) |
| 10 | System | Gán vai trò vĩnh viễn cho tài khoản (BR-0806) |
| 11 | System | Hiển thị form nhập thông tin cơ bản tổ chức tương ứng với vai trò đã chọn |
| 12 | Guest | Nhập thông tin cơ bản bắt buộc theo vai trò (BR-0807) |
| 13 | Guest | Nhấn "Hoàn tất đăng ký" |
| 14 | System | Xác thực tất cả trường bắt buộc đã được nhập |
| 15 | System | Tạo Organization entity liên kết với tài khoản |
| 16 | System | Gửi email thông báo đăng ký thành công |
| 17 | System | Chuyển Guest (nay là Authenticated User) đến trang chính với thông báo "Đăng ký thành công" |
| 18 | System | Use case kết thúc thành công — tài khoản đã được tạo với trạng thái UNVERIFIED |

---

**Alternate Flows**

> AF-34.a: Guest chọn vai trò Organizer (triggered at Step 9)

| Step | Actor / System | Action |
|------|----------------|--------|
| 9a | Guest | Chọn "Câu lạc bộ/BTC" |
| 11a | System | Hiển thị form với các trường: Tên CLB/BTC (bắt buộc), Tên trường (bắt buộc), Địa chỉ liên hệ (bắt buộc) |
| – | – | Tiếp tục từ Step 12 |

> AF-34.b: Guest chọn vai trò Sponsor (triggered at Step 9)

| Step | Actor / System | Action |
|------|----------------|--------|
| 9b | Guest | Chọn "Doanh nghiệp/Nhà tài trợ" |
| 11b | System | Hiển thị form với các trường: Tên công ty (bắt buộc), Lĩnh vực hoạt động (bắt buộc), Địa chỉ liên hệ (bắt buộc) |
| – | – | Tiếp tục từ Step 12 |

---

**Exception Flows**

> EF-34.1: Email đã tồn tại trong hệ thống (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện email đã được sử dụng bởi tài khoản khác |
| 3b | System | Hiển thị thông báo "Email này đã được sử dụng" |
| 3c | Guest | Có thể nhập email khác hoặc chuyển sang đăng nhập |

> EF-34.2: Mật khẩu không đạt yêu cầu (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện mật khẩu không đạt yêu cầu BR-0802 |
| 4b | System | Hiển thị lỗi cụ thể về yêu cầu mật khẩu chưa đạt (ví dụ: "Mật khẩu phải có ít nhất 8 ký tự, bao gồm 1 chữ hoa, 1 chữ thường, 1 chữ số") |
| 4c | Guest | Nhập lại mật khẩu đạt yêu cầu |

> EF-34.3: Mật khẩu và xác nhận không khớp (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện password và password_confirmation không khớp |
| 5b | System | Hiển thị thông báo "Mật khẩu xác nhận không khớp" |
| 5c | Guest | Nhập lại mật khẩu xác nhận |

> EF-34.4: Thiếu thông tin bắt buộc của tổ chức (triggered at Step 14)

| Step | Actor / System | Action |
|------|----------------|--------|
| 14a | System | Phát hiện một hoặc nhiều trường bắt buộc chưa được nhập |
| 14b | System | Hiển thị thông báo lỗi cụ thể cho từng trường thiếu |
| 14c | Guest | Bổ sung thông tin thiếu và thử lại |

---

**Postconditions**

*Success:*

- Tài khoản mới được tạo với status = UNVERIFIED, auth_method = EMAIL
- Vai trò tổ chức (Organizer/Sponsor) đã được gán vĩnh viễn
- Organization entity đã được tạo với thông tin cơ bản
- Email xác minh đã được gửi tới địa chỉ email đăng ký
- Guest trở thành Authenticated User và ở trang chính

*Failure:*

- Không có tài khoản nào được tạo
- Guest được thông báo về lỗi cụ thể và ở lại form đăng ký

---

**Business Rules**

- BR-0801: Mỗi email chỉ được liên kết với MỘT tài khoản duy nhất
- BR-0802: Mật khẩu tối thiểu 8 ký tự, gồm 1 chữ hoa, 1 chữ thường, 1 chữ số
- BR-0803: Gửi email xác minh, đường dẫn hiệu lực 24 giờ. Tài khoản chưa xác minh email bị giới hạn chức năng
- BR-0806: Vai trò tổ chức chọn MỘT LẦN, không thể thay đổi
- BR-0807: Thông tin bắt buộc theo vai trò (Organizer: tên CLB, tên trường, địa chỉ; Sponsor: tên công ty, lĩnh vực, địa chỉ)

---

**Notes / Assumptions**

- Quy trình đăng ký là multi-step (3 bước) nhưng diễn ra trong cùng một phiên làm việc, hướng đến cùng một mục tiêu: tạo tài khoản hoàn chỉnh
- Tài khoản được tạo với email_verified = false; xác minh email diễn ra bất đồng bộ qua link gửi tới mailbox
- Sau khi đăng ký, tài khoản cần hoàn tất thêm thông tin tùy chọn và tài liệu minh chứng (UC-38) rồi gửi hồ sơ xác thực (UC-40)
- Liên kết: UC-35 (đăng ký qua Google — luồng thay thế), UC-38, UC-40
