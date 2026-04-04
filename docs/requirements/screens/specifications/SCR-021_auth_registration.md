# SCR-021: Auth_Registration_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-021 |
| **Screen Name** | Auth_Registration_Screen |
| **Mục đích** | Guest tạo tài khoản mới trên UniLink — quy trình multi-step gồm 3 bước: (1) nhập email/mật khẩu hoặc Google OAuth, (2) chọn vai trò tổ chức, (3) nhập thông tin cơ bản tổ chức. Tất cả 3 bước diễn ra trong cùng phiên, cùng mục tiêu "tạo tài khoản hoàn chỉnh" |
| **Actor chính** | Guest |
| **Quy trình nghiệp vụ** | BP-02 / Bước 1–2 — Đăng ký tài khoản |
| **Use case liên quan** | UC-34, UC-35 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Mục tiêu: tạo tài khoản hoàn chỉnh — khác biệt hoàn toàn với đăng nhập |
| Data scope riêng | ✅ Email, mật khẩu, vai trò, thông tin tổ chức — data set lớn hơn login |
| Action set riêng | ✅ Multi-step form: đăng ký → chọn vai trò → nhập thông tin → hoàn tất |
| Navigation boundary | ✅ Trang riêng /register, navigation rõ ràng từ/về SCR-020 |
| Independently testable | ✅ Mỗi bước có acceptance criteria riêng |

> **Lưu ý thiết kế**: 3 bước (credentials → role → org info) là UI states/steps trong CÙNG MỘT screen (multi-step wizard), KHÔNG phải 3 screens riêng. Lý do: cùng mục tiêu, cùng phiên, không có back-navigation có ý nghĩa giữa các bước, và người dùng không bao giờ quay lại screen khác giữa chừng.

---

## Dữ liệu hiển thị (Read-only Data)

- Logo và branding UniLink
- Progress indicator (Step 1/3, 2/3, 3/3)
- Mô tả vai trò khi chọn (Step 2): giải thích ngắn gọn Organizer vs Sponsor
- Label các trường thông tin tổ chức tương ứng vai trò đã chọn (Step 3)

---

## Dữ liệu nhập (Input Fields)

### Step 1 — Thông tin đăng nhập

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Email | Text input (email) | Bắt buộc, unique (BR-0801) |
| Mật khẩu | Password input | Bắt buộc, validation BR-0802 |
| Xác nhận mật khẩu | Password input | Bắt buộc, phải khớp mật khẩu |

> Hoặc: nhấn "Tiếp tục bằng Google" → bỏ qua Step 1, chuyển thẳng sang Step 2

### Step 2 — Chọn vai trò tổ chức

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Vai trò | Radio/Card select | "Câu lạc bộ/BTC" (Organizer) hoặc "Doanh nghiệp/Nhà tài trợ" (Sponsor). Chọn MỘT LẦN, không thể thay đổi (BR-0806) |

### Step 3 — Thông tin cơ bản tổ chức

#### Nếu Organizer

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Tên CLB/BTC | Text input | Bắt buộc (BR-0807) |
| Tên trường | Text input | Bắt buộc (BR-0807) |
| Địa chỉ liên hệ | Text input | Bắt buộc (BR-0807) |

#### Nếu Sponsor

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Tên công ty | Text input | Bắt buộc (BR-0807) |
| Lĩnh vực hoạt động | Text input | Bắt buộc (BR-0807) |
| Địa chỉ liên hệ | Text input | Bắt buộc (BR-0807) |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Đăng ký** (Step 1) | System tạo tài khoản UNVERIFIED → chuyển sang Step 2 (UC-34 Main-2~8) |
| **Tiếp tục bằng Google** (Step 1) | Redirect Google OAuth → tạo tài khoản → chuyển Step 2 (UC-35 Main-1~7) |
| **Hoàn tất đăng ký** (Step 3) | System tạo Organization entity → redirect trang chính (UC-34 Main-13~17) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Nhấn "Đã có tài khoản? Đăng nhập" | Chuyển đến SCR-020 (Auth_Login_Screen) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0801: Mỗi email chỉ được liên kết với MỘT tài khoản duy nhất
- BR-0802: Mật khẩu tối thiểu 8 ký tự, gồm 1 chữ hoa, 1 chữ thường, 1 chữ số
- BR-0803: Gửi email xác minh, đường dẫn hiệu lực 24 giờ (chỉ cho đăng ký email)
- BR-0804: Email từ Google profile được coi là đã xác minh
- BR-0806: Vai trò tổ chức chọn MỘT LẦN, không thể thay đổi
- BR-0807: Thông tin bắt buộc theo vai trò

---

## Quy tắc xác thực (Validation Rules)

- Email: bắt buộc, định dạng email hợp lệ, chưa tồn tại trong hệ thống
- Mật khẩu: bắt buộc, ≥ 8 ký tự, ≥ 1 chữ hoa, ≥ 1 chữ thường, ≥ 1 chữ số
- Xác nhận mật khẩu: phải khớp với mật khẩu
- Vai trò: bắt buộc chọn 1 trong 2 option
- Tên tổ chức, trường/lĩnh vực, địa chỉ: bắt buộc, không được bỏ trống

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-020 (Login) | Nhấn "Đăng ký tài khoản" |
| URL trực tiếp | Truy cập /register |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Hoàn tất đăng ký thành công | Trang chính (Dashboard) với thông báo "Đăng ký thành công" |
| Nhấn "Đã có tài khoản? Đăng nhập" | SCR-020 |
| Google OAuth — email đã có tài khoản | SCR-020 → auto login (UC-35 AF-35.c) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Step 1 Active | Người dùng đang nhập email/mật khẩu |
| Step 2 Active | Tài khoản đã tạo, đang chọn vai trò |
| Step 3 Active | Vai trò đã chọn, đang nhập thông tin tổ chức |
| Loading State | Đang xử lý đăng ký / Google OAuth |
| Error State — Email đã tồn tại | Email trùng (EF-34.1) |
| Error State — Mật khẩu yếu | Mật khẩu không đạt BR-0802 (EF-34.2) |
| Error State — Mật khẩu không khớp | Xác nhận mật khẩu sai (EF-34.3) |
| Error State — Google OAuth thất bại | Google trả về lỗi (EF-35.1) |
| Error State — Thiếu thông tin tổ chức | Trường bắt buộc bỏ trống (EF-34.4, EF-35.2) |

## UI Components liên quan

- Multi-step wizard / Stepper component — hiển thị progress 3 bước
- Registration form (Step 1) — email + password fields
- Google OAuth button — "Tiếp tục bằng Google"
- Role selection cards (Step 2) — 2 card Organizer / Sponsor với mô tả
- Organization info form (Step 3) — dynamic fields theo vai trò
- Inline validation — hiển thị lỗi realtime từng trường
- Alert/Toast notification — hiển thị lỗi hệ thống

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-34 | Main-1 | Guest nhập email, mật khẩu, xác nhận mật khẩu | Input | Step 1 form fields |
| UC-34 | Main-2 | Guest nhấn "Đăng ký" | Action | CTA button "Đăng ký" |
| UC-34 | Main-3~7 | System tạo tài khoản + gửi email xác minh | Action | System xử lý → chuyển Step 2 |
| UC-34 | Main-8~9 | Chuyển sang chọn vai trò, Guest chọn | Input | Step 2 role selection |
| UC-34 | Main-10~11 | Gán vai trò, hiển thị form thông tin tổ chức | Action | Step 3 form hiển thị |
| UC-34 | Main-12~13 | Guest nhập thông tin và nhấn "Hoàn tất" | Input + Action | Step 3 form + CTA |
| UC-34 | Main-14~17 | System xác thực, tạo Organization, redirect | Action | System xử lý → redirect dashboard |
| UC-34 | EF-34.1 | Email đã tồn tại | UI State | Inline error trên email field |
| UC-34 | EF-34.2 | Mật khẩu yếu | UI State | Inline error trên password field |
| UC-34 | EF-34.3 | Mật khẩu không khớp | UI State | Inline error trên confirm password |
| UC-34 | EF-34.4 | Thiếu thông tin tổ chức | UI State | Inline errors trên Step 3 fields |
| UC-35 | Main-1 | Guest nhấn "Tiếp tục bằng Google" | Action | Google OAuth button |
| UC-35 | Main-2~6 | Google OAuth → tạo tài khoản | Action | System xử lý → chuyển Step 2 |
| UC-35 | Main-7~9 | Chọn vai trò tổ chức | Input | Step 2 role selection |
| UC-35 | Main-10~12 | Nhập thông tin tổ chức, nhấn "Hoàn tất" | Input + Action | Step 3 form + CTA |
| UC-35 | Main-13~16 | System xác thực, tạo Organization, redirect | Action | System xử lý → redirect dashboard |
| UC-35 | AF-35.c | Email Google đã có tài khoản → auto login | Navigation Out | Redirect → Dashboard |
| UC-35 | EF-35.1 | Google OAuth thất bại | UI State | Error alert |
| UC-35 | EF-35.2 | Thiếu thông tin tổ chức | UI State | Inline errors trên Step 3 fields |
