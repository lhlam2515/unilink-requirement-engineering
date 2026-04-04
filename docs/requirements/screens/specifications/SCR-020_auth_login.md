# SCR-020: Auth_Login_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-020 |
| **Screen Name** | Auth_Login_Screen |
| **Mục đích** | Guest đăng nhập vào hệ thống UniLink bằng email/mật khẩu hoặc nhanh qua Google OAuth. Đây là entry point chính cho người dùng đã có tài khoản |
| **Actor chính** | Guest |
| **Quy trình nghiệp vụ** | BP-02 / Bước 1 — Đăng ký và xác thực tài khoản |
| **Use case liên quan** | UC-36, UC-37 (trigger) |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Mục tiêu: xác thực danh tính, tạo phiên đăng nhập |
| Data scope riêng | ✅ Form email/password, nút Google OAuth |
| Action set riêng | ✅ Đăng nhập, đăng nhập Google, chuyển đăng ký, quên mật khẩu |
| Navigation boundary | ✅ Trang độc lập, entry point chính cho unauthenticated user |
| Independently testable | ✅ Có thể viết acceptance criteria riêng |

---

## Dữ liệu hiển thị (Read-only Data)

- Logo và branding UniLink
- Thông báo lỗi đăng nhập (nếu có): "Email hoặc mật khẩu không chính xác"
- Thông báo tài khoản bị tạm khóa (nếu có): "Tài khoản tạm thời bị khóa, vui lòng thử lại sau [X] phút"
- Thông báo đăng ký thành công (nếu redirect từ SCR-021)
- Thông báo đổi mật khẩu thành công (nếu redirect từ SCR-022)

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Email | Text input (email) | Bắt buộc |
| Mật khẩu | Password input | Bắt buộc |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Đăng nhập** | System xác thực → Chuyển đến trang chính phù hợp vai trò (UC-36 Main Flow) |
| **Tiếp tục bằng Google** | Redirect Google OAuth → Đăng nhập hoặc Đăng ký mới (UC-36 AF-36.a / UC-35) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Nhấn "Đăng ký tài khoản" | Chuyển đến SCR-021 (Auth_Registration_Screen) |
| Nhấn "Quên mật khẩu" | Chuyển đến SCR-022 (Auth_ResetPassword_Screen) — UC-37 trigger |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0805: Sau 5 lần đăng nhập sai liên tiếp, tài khoản bị TẠM KHÓA 15 phút. Bộ đếm reset khi đăng nhập thành công hoặc hết thời gian khóa

---

## Quy tắc xác thực (Validation Rules)

- Email: không được bỏ trống, phải đúng định dạng email
- Mật khẩu: không được bỏ trống
- Thông báo lỗi chung "Email hoặc mật khẩu không chính xác" — KHÔNG tiết lộ email có tồn tại hay không (chống enumeration)

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| URL trực tiếp / Landing page | Truy cập /login |
| SCR-021 (Registration) | Nhấn "Đã có tài khoản? Đăng nhập" |
| SCR-022 (Reset Password) | Sau đổi mật khẩu thành công |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Đăng nhập thành công | Trang chính phù hợp vai trò (Dashboard) |
| Nhấn "Đăng ký tài khoản" | SCR-021 |
| Nhấn "Quên mật khẩu" | SCR-022 |
| Google OAuth — email chưa có tài khoản | SCR-021 (Step chọn vai trò + nhập thông tin) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang xác thực thông tin đăng nhập |
| Error State — Sai thông tin | Email/mật khẩu không chính xác (EF-36.1) |
| Error State — Tài khoản bị khóa | Tạm khóa do đăng nhập sai quá 5 lần (EF-36.2, EF-36.3) |
| Error State — Google OAuth thất bại | Google trả về lỗi hoặc user hủy (EF-36.4) |
| Success State | Thông báo từ flow khác (đăng ký thành công, đổi mật khẩu thành công) |

## UI Components liên quan

- Login form — email + password fields
- Google OAuth button — "Tiếp tục bằng Google"
- Link "Quên mật khẩu"
- Link "Đăng ký tài khoản"
- Alert/Toast notification — hiển thị lỗi hoặc thông báo thành công

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-36 | Main-1 | Guest nhập email và mật khẩu | Input | Email field + Password field |
| UC-36 | Main-2 | Guest nhấn "Đăng nhập" | Action | CTA button "Đăng nhập" |
| UC-36 | Main-3~6 | System xác thực và tạo session | Action | System xử lý → redirect |
| UC-36 | Main-7 | Chuyển đến trang chính | Navigation Out | Redirect theo vai trò |
| UC-36 | AF-36.a | Đăng nhập bằng Google OAuth | Action | Google OAuth button |
| UC-36 | EF-36.1 | Email/mật khẩu không chính xác | UI State | Error alert |
| UC-36 | EF-36.2 | Tài khoản bị tạm khóa | UI State | Error alert + countdown |
| UC-36 | EF-36.3 | Đăng nhập sai quá 5 lần | UI State | Error alert + lock notification |
| UC-36 | EF-36.4 | Google OAuth thất bại | UI State | Error alert |
| UC-36 | EF-36.5 | Email Google chưa có tài khoản | Navigation Out | Redirect → SCR-021 (UC-35) |
| UC-37 | Trigger | Guest nhấn "Quên mật khẩu" | Navigation Out | Link → SCR-022 |
