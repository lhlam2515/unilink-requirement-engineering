# UC-39: Chỉnh sửa hồ sơ tổ chức

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) chỉnh sửa thông tin hồ sơ tổ chức đã nhập. Quyền chỉnh sửa phụ thuộc vào trạng thái xác thực hiện tại của tài khoản: cho phép chỉnh sửa toàn bộ khi UNVERIFIED/REJECTED/INFO_REQUIRED, cho phép kèm đánh dấu khi PENDING_REVIEW, và không cho phép khi VERIFIED.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người chỉnh sửa hồ sơ |
| Secondary | System | Kiểm tra quyền chỉnh sửa, cập nhật hồ sơ |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Actor đã có hồ sơ tổ chức (Organization entity tồn tại)

---

**Trigger**
> Actor truy cập trang "Hồ sơ tổ chức" và nhấn "Chỉnh sửa".

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Truy cập trang "Hồ sơ tổ chức" |
| 2 | System | Hiển thị thông tin hiện tại của hồ sơ tổ chức |
| 3 | Authenticated User | Nhấn "Chỉnh sửa" |
| 4 | System | Kiểm tra quyền chỉnh sửa dựa trên verification_status (BR-0903) |
| 5 | System | Hiển thị form chỉnh sửa với dữ liệu hiện tại |
| 6 | Authenticated User | Cập nhật thông tin cần chỉnh sửa |
| 7 | Authenticated User | Nhấn "Lưu thay đổi" |
| 8 | System | Xác thực các trường bắt buộc vẫn đầy đủ |
| 9 | System | Cập nhật hồ sơ tổ chức và ghi nhận updated_at |
| 10 | System | Hiển thị thông báo "Cập nhật hồ sơ thành công" |
| 11 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-39.a: Chỉnh sửa khi PENDING_REVIEW (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện verification_status = PENDING_REVIEW |
| 4b | System | Hiển thị cảnh báo "Hồ sơ đang chờ kiểm duyệt. Thay đổi sẽ được đánh dấu để kiểm duyệt viên biết." |
| 5a | System | Hiển thị form chỉnh sửa bình thường |
| – | – | Tiếp tục từ Step 6 |
| 9a | System | Cập nhật hồ sơ và đánh dấu "đã cập nhật kể từ lần gửi cuối" |

> AF-39.b: Chỉnh sửa khi REJECTED (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4c | System | Phát hiện verification_status = REJECTED |
| 4d | System | Hiển thị lý do từ chối từ lần kiểm duyệt gần nhất để actor tham khảo |
| 5b | System | Hiển thị form chỉnh sửa cho phép sửa tất cả thông tin |
| – | – | Tiếp tục từ Step 6 |

> AF-39.c: Chỉnh sửa khi INFO_REQUIRED (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4e | System | Phát hiện verification_status = INFO_REQUIRED |
| 4f | System | Hiển thị chi tiết yêu cầu bổ sung từ Admin |
| 5c | System | Hiển thị form chỉnh sửa cho phép sửa tất cả thông tin |
| – | – | Tiếp tục từ Step 6 |

---

**Exception Flows**

> EF-39.1: Hồ sơ đã VERIFIED — không thể chỉnh sửa (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4g | System | Phát hiện verification_status = VERIFIED |
| 4h | System | Hiển thị "Hồ sơ đã xác thực không thể chỉnh sửa trực tiếp" |
| 4i | Authenticated User | Có thể liên hệ hỗ trợ nếu cần thay đổi |

> EF-39.2: Thiếu trường bắt buộc sau chỉnh sửa (triggered at Step 8)

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | System | Phát hiện trường bắt buộc bị bỏ trống sau khi chỉnh sửa |
| 8b | System | Hiển thị lỗi cụ thể cho từng trường thiếu |
| 8c | Authenticated User | Bổ sung thông tin thiếu |

---

**Postconditions**

*Success:*

- Hồ sơ tổ chức đã được cập nhật với thông tin mới
- updated_at đã được ghi nhận
- Nếu đang PENDING_REVIEW: hồ sơ được đánh dấu "đã cập nhật"

*Failure:*

- Hồ sơ không thay đổi
- Actor được thông báo lý do không thể chỉnh sửa

---

**Business Rules**

- BR-0903: Quyền chỉnh sửa phụ thuộc verification_status:
  - UNVERIFIED: chỉnh sửa tất cả
  - PENDING_REVIEW: chỉnh sửa kèm đánh dấu
  - VERIFIED: KHÔNG chỉnh sửa
  - REJECTED / INFO_REQUIRED: chỉnh sửa tất cả

---

**Notes / Assumptions**

- Hồ sơ VERIFIED cần liên hệ hỗ trợ hoặc tạo yêu cầu thay đổi riêng (ngoài phạm vi hiện tại)
- Chỉnh sửa khi REJECTED/INFO_REQUIRED giúp actor chuẩn bị trước khi gửi lại hồ sơ xác thực (UC-40)
- Liên kết: UC-38 (bổ sung tài liệu), UC-40 (gửi xác thực)
