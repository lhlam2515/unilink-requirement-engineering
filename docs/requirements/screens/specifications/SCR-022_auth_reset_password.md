# SCR-022: Auth_ResetPassword_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-022 |
| **Screen Name** | Auth_ResetPassword_Screen |
| **Mục đích** | Guest yêu cầu đặt lại mật khẩu (nhập email) và nhập mật khẩu mới (sau khi truy cập đường dẫn từ email). Screen phục vụ 2 giai đoạn của cùng quy trình reset password |
| **Actor chính** | Guest |
| **Quy trình nghiệp vụ** | BP-02 / Bước 1 — Đăng ký và xác thực tài khoản (khôi phục truy cập) |
| **Use case liên quan** | UC-37 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Mục tiêu: khôi phục mật khẩu — khác biệt với đăng nhập và đăng ký |
| Data scope riêng | ✅ Email (giai đoạn 1), mật khẩu mới + xác nhận (giai đoạn 2) |
| Action set riêng | ✅ Gửi yêu cầu reset, nhập mật khẩu mới, đổi mật khẩu |
| Navigation boundary | ✅ Route riêng /reset-password và /reset-password?token=xxx |
| Independently testable | ✅ Mỗi giai đoạn có acceptance criteria riêng |

> **Lưu ý thiết kế**: 2 giai đoạn (yêu cầu email + nhập mật khẩu mới) là UI states trong CÙNG MỘT screen vì chúng phục vụ cùng mục tiêu "đặt lại mật khẩu". Giai đoạn 2 được kích hoạt qua token trong URL.

---

## Dữ liệu hiển thị (Read-only Data)

### Giai đoạn 1 — Yêu cầu đặt lại

- Hướng dẫn: "Nhập email đã đăng ký để nhận đường dẫn đặt lại mật khẩu"
- Thông báo thành công: "Vui lòng kiểm tra email để đặt lại mật khẩu"

### Giai đoạn 2 — Nhập mật khẩu mới

- Hướng dẫn: "Nhập mật khẩu mới cho tài khoản của bạn"
- Yêu cầu mật khẩu: hint về BR-0802 (≥8 ký tự, chữ hoa, chữ thường, chữ số)

---

## Dữ liệu nhập (Input Fields)

### Giai đoạn 1 — Yêu cầu đặt lại

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Email | Text input (email) | Bắt buộc |

### Giai đoạn 2 — Nhập mật khẩu mới

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Mật khẩu mới | Password input | Bắt buộc, validation BR-0802 |
| Xác nhận mật khẩu | Password input | Bắt buộc, phải khớp mật khẩu mới |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Gửi yêu cầu đặt lại** (Giai đoạn 1) | System gửi email reset → hiển thị xác nhận (UC-37 Main-4~8) |
| **Đổi mật khẩu** (Giai đoạn 2) | System cập nhật mật khẩu → redirect SCR-020 (UC-37 Main-13~18) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Nhấn "Quay lại đăng nhập" | Chuyển đến SCR-020 (Auth_Login_Screen) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0802: Mật khẩu tối thiểu 8 ký tự, gồm 1 chữ hoa, 1 chữ thường, 1 chữ số
- BR-0808: Token reset hiệu lực 15 phút. Yêu cầu mới vô hiệu hóa token cũ. Tối đa 3 yêu cầu/giờ/email

---

## Quy tắc xác thực (Validation Rules)

- Email: bắt buộc, định dạng email hợp lệ
- Mật khẩu mới: bắt buộc, ≥ 8 ký tự, ≥ 1 chữ hoa, ≥ 1 chữ thường, ≥ 1 chữ số
- Xác nhận mật khẩu: phải khớp mật khẩu mới
- Hệ thống KHÔNG tiết lộ email có tồn tại hay không — luôn hiển thị thông báo chung

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-020 (Login) | Nhấn "Quên mật khẩu" |
| Email reset password | Click đường dẫn trong email → Giai đoạn 2 |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Đổi mật khẩu thành công | SCR-020 với thông báo "Đổi mật khẩu thành công" |
| Nhấn "Quay lại đăng nhập" | SCR-020 |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Phase 1 — Email form | Truy cập /reset-password (không có token) |
| Phase 1 — Success | Gửi yêu cầu thành công, hiển thị thông báo kiểm tra email |
| Phase 2 — New password form | Truy cập /reset-password?token=xxx (token hợp lệ) |
| Loading State | Đang xử lý yêu cầu |
| Error State — Token hết hạn | Token quá 15 phút (EF-37.1) |
| Error State — Token đã sử dụng | Token đã dùng (EF-37.2) |
| Error State — Mật khẩu yếu | Mật khẩu không đạt BR-0802 (EF-37.3) |

## UI Components liên quan

- Email request form (Phase 1)
- New password form (Phase 2)
- Password strength indicator
- Inline validation — hiển thị lỗi realtime
- Alert/Toast notification — thông báo thành công hoặc lỗi
- Link "Quay lại đăng nhập"

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-37 | Main-1~2 | Guest nhấn "Quên mật khẩu", system hiển thị form | Navigation In | Phase 1 form |
| UC-37 | Main-3~4 | Guest nhập email, nhấn "Gửi yêu cầu" | Input + Action | Email field + CTA |
| UC-37 | Main-5~8 | System tạo token, gửi email, hiển thị xác nhận | UI State | Phase 1 Success state |
| UC-37 | Main-9~10 | Guest truy cập link, system xác thực token | Navigation In | Phase 2 (URL với token) |
| UC-37 | Main-11 | System hiển thị form nhập mật khẩu mới | Read-only | Phase 2 form |
| UC-37 | Main-12~13 | Guest nhập mật khẩu mới, nhấn "Đổi mật khẩu" | Input + Action | Password fields + CTA |
| UC-37 | Main-14~18 | System cập nhật, vô hiệu session, redirect login | Action | System xử lý → redirect SCR-020 |
| UC-37 | AF-37.a | Email không tồn tại — vẫn hiển thị thông báo chung | UI State | Phase 1 Success state (bảo mật) |
| UC-37 | EF-37.1 | Token hết hạn | UI State | Error page + link yêu cầu lại |
| UC-37 | EF-37.2 | Token đã sử dụng | UI State | Error page + link yêu cầu lại |
| UC-37 | EF-37.3 | Mật khẩu không đạt yêu cầu | UI State | Inline error trên password field |
| UC-37 | EF-37.4 | Vượt quá 3 yêu cầu/giờ | UI State | Phase 1 Success state (bảo mật) |
