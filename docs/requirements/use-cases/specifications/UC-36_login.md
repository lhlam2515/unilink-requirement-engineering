# UC-36: Đăng nhập hệ thống

**Brief Description**
> Guest đăng nhập vào hệ thống UniLink bằng email/mật khẩu hoặc qua Google OAuth. Hệ thống xác thực thông tin đăng nhập, tạo phiên làm việc (session), và chuyển đến trang chính phù hợp với vai trò. Hệ thống áp dụng cơ chế bảo vệ chống brute-force khi đăng nhập sai nhiều lần.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Guest | Người dùng đã có tài khoản, chưa đăng nhập |
| Secondary | System | Xác thực danh tính, tạo session, ghi log |
| Secondary | Google Identity Provider | Cung cấp xác thực OAuth (cho luồng đăng nhập Google) |

---

**Preconditions**

- Guest đã có tài khoản trên hệ thống UniLink
- Guest chưa đăng nhập (chưa có session hợp lệ)

---

**Trigger**
> Guest truy cập trang đăng nhập và nhấn "Đăng nhập".

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Guest | Nhập email và mật khẩu trên form đăng nhập |
| 2 | Guest | Nhấn "Đăng nhập" |
| 3 | System | Xác thực email và mật khẩu với thông tin trong hệ thống |
| 4 | System | Kiểm tra tài khoản không bị tạm khóa (BR-0805) |
| 5 | System | Tạo session_token cho phiên làm việc |
| 6 | System | Cập nhật last_login_at và reset failed_login_attempts = 0 |
| 7 | System | Chuyển Guest (nay là Authenticated User) đến trang chính phù hợp với vai trò |
| 8 | System | Use case kết thúc thành công — phiên đăng nhập đã được tạo |

---

**Alternate Flows**

> AF-36.a: Đăng nhập bằng Google OAuth (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Guest | Nhấn "Tiếp tục bằng Google" thay vì nhập email/mật khẩu |
| 1b | System | Chuyển hướng đến trang xác thực Google OAuth |
| 1c | Guest | Xác thực tài khoản Google và cấp quyền |
| 1d | System | Nhận Google OAuth token và trích xuất email |
| 1e | System | Xác thực email Google đã liên kết với tài khoản trên hệ thống |
| 1f | System | Tạo session_token cho phiên làm việc |
| 1g | System | Chuyển đến trang chính phù hợp với vai trò — use case kết thúc |

---

**Exception Flows**

> EF-36.1: Email hoặc mật khẩu không chính xác (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện email không tồn tại hoặc mật khẩu không khớp |
| 3b | System | Tăng failed_login_attempts (nếu tài khoản tồn tại) |
| 3c | System | Hiển thị thông báo "Email hoặc mật khẩu không chính xác" (không tiết lộ email có tồn tại hay không) |
| 3d | Guest | Có thể nhập lại hoặc chọn "Quên mật khẩu" (UC-37) |

> EF-36.2: Tài khoản bị tạm khóa do đăng nhập sai (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện tài khoản đang bị tạm khóa (locked_until > thời gian hiện tại) |
| 4b | System | Hiển thị "Tài khoản tạm thời bị khóa, vui lòng thử lại sau [X] phút" |
| 4c | Guest | Chờ hết thời gian khóa hoặc sử dụng "Quên mật khẩu" (UC-37) |

> EF-36.3: Đăng nhập sai quá số lần cho phép (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3e | System | Phát hiện failed_login_attempts đạt 5 lần (BR-0805) |
| 3f | System | Khóa tài khoản trong 15 phút (set locked_until) |
| 3g | System | Hiển thị "Tài khoản đã bị khóa tạm thời do đăng nhập sai quá nhiều lần" |

> EF-36.4: Google OAuth thất bại (triggered at AF-36.a, Step 1d)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1d-a | System | Nhận phản hồi lỗi từ Google hoặc Guest hủy xác thực |
| 1d-b | System | Hiển thị "Đăng nhập bằng Google không thành công, vui lòng thử lại" |

> EF-36.5: Email Google không liên kết với tài khoản (triggered at AF-36.a, Step 1e)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1e-a | System | Phát hiện email Google chưa có tài khoản trên hệ thống |
| 1e-b | System | Chuyển sang quy trình đăng ký mới (UC-35) — tạo tài khoản với Google |

---

**Postconditions**

*Success:*

- Phiên đăng nhập (session_token) đã được tạo
- last_login_at đã được cập nhật
- failed_login_attempts đã được reset về 0
- Authenticated User ở trang chính phù hợp với vai trò

*Failure:*

- Không có phiên đăng nhập nào được tạo
- Guest được thông báo về lỗi (không tiết lộ thông tin nhạy cảm)
- Nếu đăng nhập sai quá 5 lần: tài khoản bị tạm khóa 15 phút

---

**Business Rules**

- BR-0805: Sau 5 lần đăng nhập sai liên tiếp, tài khoản bị TẠM KHÓA 15 phút. Bộ đếm reset khi đăng nhập thành công hoặc hết thời gian khóa

---

**Notes / Assumptions**

- Hệ thống KHÔNG tiết lộ email có tồn tại hay không khi đăng nhập sai — biện pháp bảo mật chống enumeration
- Đăng nhập qua Google (AF-36.a) được xử lý như alternate flow, không phải UC riêng vì cùng mục tiêu "tạo phiên đăng nhập"
- Nếu email Google chưa có tài khoản, hệ thống tự động chuyển sang luồng đăng ký (UC-35)
- Liên kết: UC-34, UC-35, UC-37
