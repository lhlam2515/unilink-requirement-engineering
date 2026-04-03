# UC-37: Đặt lại mật khẩu

**Brief Description**
> Guest yêu cầu đặt lại mật khẩu khi quên. Hệ thống gửi email chứa đường dẫn reset password với token có thời hạn. Guest truy cập đường dẫn và nhập mật khẩu mới. Sau khi đổi thành công, tất cả phiên đăng nhập cũ bị vô hiệu hóa.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Guest | Người dùng đã có tài khoản nhưng quên mật khẩu |
| Secondary | System | Gửi email reset, xác thực token, cập nhật mật khẩu |

---

**Preconditions**

- Guest đã có tài khoản trên hệ thống (auth_method = EMAIL)
- Guest đang ở trang đăng nhập

---

**Trigger**
> Guest nhấn "Quên mật khẩu" trên trang đăng nhập.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Guest | Nhấn "Quên mật khẩu" |
| 2 | System | Hiển thị form yêu cầu nhập email |
| 3 | Guest | Nhập email đã đăng ký |
| 4 | Guest | Nhấn "Gửi yêu cầu đặt lại mật khẩu" |
| 5 | System | Tạo reset token với thời hạn 15 phút (BR-0808) |
| 6 | System | Vô hiệu hóa token cũ (nếu có) cho email này |
| 7 | System | Gửi email chứa đường dẫn reset password |
| 8 | System | Hiển thị "Vui lòng kiểm tra email để đặt lại mật khẩu" |
| 9 | Guest | Truy cập đường dẫn reset password từ email |
| 10 | System | Xác thực token hợp lệ và chưa hết hạn |
| 11 | System | Hiển thị form nhập mật khẩu mới |
| 12 | Guest | Nhập mật khẩu mới và xác nhận mật khẩu |
| 13 | Guest | Nhấn "Đổi mật khẩu" |
| 14 | System | Xác thực mật khẩu mới đạt yêu cầu (BR-0802) |
| 15 | System | Cập nhật mật khẩu mới (lưu dạng hash) |
| 16 | System | Đánh dấu token đã sử dụng (used = true) |
| 17 | System | Vô hiệu hóa tất cả session hiện tại của tài khoản |
| 18 | System | Chuyển đến trang đăng nhập với thông báo "Đổi mật khẩu thành công" |
| 19 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-37.a: Email không tồn tại trong hệ thống (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện email không tồn tại trong hệ thống |
| 5b | System | Vẫn hiển thị thông báo "Vui lòng kiểm tra email để đặt lại mật khẩu" (không tiết lộ email tồn tại hay không) |
| 5c | System | Không gửi email — use case kết thúc |

---

**Exception Flows**

> EF-37.1: Token reset đã hết hạn (triggered at Step 10)

| Step | Actor / System | Action |
|------|----------------|--------|
| 10a | System | Phát hiện token đã quá thời hạn 15 phút |
| 10b | System | Hiển thị "Đường dẫn đã hết hạn, vui lòng yêu cầu lại" |
| 10c | Guest | Có thể quay lại yêu cầu reset mới (Step 1) |

> EF-37.2: Token đã được sử dụng (triggered at Step 10)

| Step | Actor / System | Action |
|------|----------------|--------|
| 10d | System | Phát hiện token đã được sử dụng (used = true) |
| 10e | System | Hiển thị "Đường dẫn đã được sử dụng, vui lòng yêu cầu lại" |

> EF-37.3: Mật khẩu mới không đạt yêu cầu (triggered at Step 14)

| Step | Actor / System | Action |
|------|----------------|--------|
| 14a | System | Phát hiện mật khẩu mới không đạt yêu cầu BR-0802 |
| 14b | System | Hiển thị lỗi cụ thể về yêu cầu chưa đạt |
| 14c | Guest | Nhập lại mật khẩu mới đạt yêu cầu |

> EF-37.4: Vượt quá giới hạn yêu cầu reset (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5d | System | Phát hiện đã có 3 yêu cầu reset trong 1 giờ qua cho email này (BR-0808) |
| 5e | System | Vẫn hiển thị thông báo chung (không tiết lộ cụ thể) |
| 5f | System | Không gửi email — ghi nhận cảnh báo bảo mật |

---

**Postconditions**

*Success:*

- Mật khẩu mới đã được cập nhật
- Token reset đã được đánh dấu đã sử dụng
- Tất cả session cũ đã bị vô hiệu hóa
- Guest ở trang đăng nhập, sẵn sàng đăng nhập với mật khẩu mới

*Failure:*

- Mật khẩu không thay đổi
- Guest được thông báo về lỗi cụ thể (token hết hạn, mật khẩu yếu)

---

**Business Rules**

- BR-0802: Mật khẩu tối thiểu 8 ký tự, gồm 1 chữ hoa, 1 chữ thường, 1 chữ số
- BR-0808: Token reset password hiệu lực 15 phút. Yêu cầu mới vô hiệu hóa token cũ. Tối đa 3 yêu cầu/giờ/email

---

**Notes / Assumptions**

- Hệ thống KHÔNG tiết lộ email có tồn tại hay không khi yêu cầu reset — biện pháp bảo mật
- Use case này chỉ áp dụng cho tài khoản đăng ký bằng email (auth_method = EMAIL). Tài khoản Google sử dụng cơ chế khôi phục của Google
- Liên kết: UC-36 (đăng nhập)
