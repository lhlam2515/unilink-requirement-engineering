# UC-44: Từ chối hồ sơ tổ chức

**Brief Description**
> Admin từ chối hồ sơ xác thực tổ chức kèm theo lý do rõ ràng. Khi từ chối, hệ thống chuyển trạng thái sang REJECTED và gửi thông báo cho người dùng. Người dùng không cần tạo lại tài khoản — chỉ cần bổ sung/chỉnh sửa và gửi lại hồ sơ xác thực.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người từ chối hồ sơ |
| Secondary | System | Chuyển trạng thái, gửi thông báo, ghi audit log |

---

**Preconditions**

- Admin đã đăng nhập vào hệ thống với vai trò `admin`
- Admin đã xem chi tiết hồ sơ xác thực (UC-42)
- Hồ sơ đang có status = PENDING

---

**Trigger**
> Admin nhấn "Từ chối" trên trang chi tiết hồ sơ.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Admin | Nhấn "Từ chối" trên trang chi tiết hồ sơ |
| 2 | System | Hiển thị form từ chối với trường lý do (bắt buộc) |
| 3 | Admin | Nhập lý do từ chối rõ ràng |
| 4 | Admin | Xác nhận từ chối |
| 5 | System | Chuyển verification_status của tài khoản sang REJECTED |
| 6 | System | Cập nhật VerificationRequest: status = REJECTED, reviewed_at, reviewed_by, rejection_reason |
| 7 | System | Ghi nhận AuditLog: action = REJECTED, details chứa lý do |
| 8 | System | Gửi email "Hồ sơ của bạn bị từ chối. Lý do: [rejection_reason]. Bạn có thể bổ sung và gửi lại." |
| 9 | System | Gửi thông báo in-app tương tự |
| 10 | System | Hiển thị xác nhận "Hồ sơ đã bị từ chối" |
| 11 | System | Chuyển Admin quay lại danh sách hồ sơ chờ duyệt (UC-41) |
| 12 | System | Use case kết thúc thành công |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-44.1: Không nhập lý do từ chối (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện trường lý do từ chối trống |
| 4b | System | Hiển thị "Vui lòng nhập lý do từ chối" |
| 4c | Admin | Nhập lý do và thử lại |

> EF-44.2: Hồ sơ đã được xử lý trước đó (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện hồ sơ không còn ở status PENDING |
| 5b | System | Hiển thị "Hồ sơ đã được xử lý bởi admin khác" |
| 5c | System | Chuyển Admin quay lại danh sách |

---

**Postconditions**

*Success:*

- verification_status = REJECTED
- VerificationRequest.status = REJECTED với rejection_reason, reviewed_at, reviewed_by
- AuditLog đã ghi nhận hành động từ chối kèm lý do
- Người dùng nhận thông báo qua email và in-app kèm lý do và hướng dẫn gửi lại
- Người dùng có thể chỉnh sửa (UC-39) và gửi lại (UC-40)

*Failure:*

- Trạng thái không thay đổi
- Admin được yêu cầu nhập lý do

---

**Business Rules**

- BR-1004: Khi từ chối: (1) rejection_reason bắt buộc, (2) người dùng không cần tạo lại tài khoản, (3) chuyển REJECTED
- BR-1006: Gửi thông báo email + in-app khi hồ sơ bị từ chối kèm lý do và hướng dẫn gửi lại

---

**Notes / Assumptions**

- Từ chối không xóa tài khoản — người dùng có full quyền chỉnh sửa và gửi lại hồ sơ
- Lý do từ chối được lưu trong rejection_reason và hiển thị cho người dùng tham khảo khi chỉnh sửa (UC-39 AF-39.b)
- Liên kết: UC-42 (<<extend>> từ xem chi tiết), UC-39 (chỉnh sửa sau từ chối), UC-40 (gửi lại)
