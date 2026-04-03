# UC-43: Phê duyệt hồ sơ tổ chức

**Brief Description**
> Admin phê duyệt hồ sơ xác thực tổ chức sau khi xem xét chi tiết. Khi phê duyệt, hệ thống chuyển trạng thái sang VERIFIED, tự động mở khóa quyền truy cập theo vai trò, thiết lập thời hạn lưu trữ tài liệu, và gửi thông báo cho người dùng.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người phê duyệt hồ sơ |
| Secondary | System | Chuyển trạng thái, mở khóa quyền, gửi thông báo, ghi audit log |

---

**Preconditions**

- Admin đã đăng nhập vào hệ thống với vai trò `admin`
- Admin đã xem chi tiết hồ sơ xác thực (UC-42)
- Hồ sơ đang có status = PENDING

---

**Trigger**
> Admin nhấn "Phê duyệt" trên trang chi tiết hồ sơ.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Admin | Nhấn "Phê duyệt" trên trang chi tiết hồ sơ |
| 2 | System | Hiển thị xác nhận: "Bạn có chắc chắn muốn phê duyệt hồ sơ này?" với trường ghi chú tùy chọn |
| 3 | Admin | Nhập ghi chú (tùy chọn) và xác nhận phê duyệt |
| 4 | System | Chuyển verification_status của tài khoản sang VERIFIED |
| 5 | System | Cập nhật VerificationRequest: status = APPROVED, reviewed_at, reviewed_by |
| 6 | System | Tự động mở khóa tất cả quyền truy cập theo vai trò (FR-0904) |
| 7 | System | Thiết lập retention_expires_at = reviewed_at + 7 ngày cho tài liệu minh chứng (BR-0907 từ SF-09) |
| 8 | System | Ghi nhận AuditLog: action = APPROVED |
| 9 | System | Gửi email "Chúc mừng! Hồ sơ tổ chức của bạn đã được xác thực" cho người dùng |
| 10 | System | Gửi thông báo in-app tương tự |
| 11 | System | Hiển thị xác nhận "Hồ sơ đã được phê duyệt thành công" |
| 12 | System | Chuyển Admin quay lại danh sách hồ sơ chờ duyệt (UC-41) |
| 13 | System | Use case kết thúc thành công |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-43.1: Hồ sơ đã được xử lý trước đó (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện hồ sơ không còn ở status PENDING (đã bị Admin khác xử lý) |
| 4b | System | Hiển thị "Hồ sơ đã được xử lý bởi admin khác" |
| 4c | System | Chuyển Admin quay lại danh sách |

> EF-43.2: Tài khoản đã VERIFIED (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4d | System | Phát hiện verification_status = VERIFIED |
| 4e | System | Hiển thị "Tài khoản đã được xác thực" |

---

**Postconditions**

*Success:*

- verification_status = VERIFIED
- VerificationRequest.status = APPROVED với reviewed_at, reviewed_by
- Tất cả quyền truy cập theo vai trò đã được mở khóa (FR-0904)
- retention_expires_at đã được thiết lập cho tài liệu minh chứng
- AuditLog đã ghi nhận hành động phê duyệt
- Người dùng nhận thông báo qua email và in-app

*Failure:*

- Trạng thái không thay đổi
- Admin được thông báo lý do không thể phê duyệt

---

**Business Rules**

- BR-1003: Khi phê duyệt: (1) chuyển VERIFIED, (2) mở khóa quyền tự động, (3) thiết lập retention_expires_at = reviewed_at + 7 ngày
- BR-1006: Gửi thông báo email + in-app khi hồ sơ được phê duyệt

---

**Notes / Assumptions**

- Phê duyệt là hành vi không thể hoàn tác (sau khi VERIFIED, hồ sơ không thể chỉnh sửa trực tiếp)
- Mở khóa quyền (FR-0904) và xóa tài liệu tạm (FR-0905) là hành vi system tự động, nhúng vào postconditions
- Liên kết: UC-42 (<<extend>> từ xem chi tiết), UC-41 (quay lại danh sách)
