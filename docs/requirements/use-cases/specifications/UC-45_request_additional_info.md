# UC-45: Yêu cầu bổ sung thông tin hồ sơ

**Brief Description**
> Admin yêu cầu tổ chức bổ sung thông tin hoặc minh chứng khi hồ sơ xác thực chưa đủ rõ ràng. Hệ thống chuyển trạng thái sang INFO_REQUIRED và gửi thông báo chi tiết cho người dùng. Sau khi bổ sung, người dùng có thể gửi lại hồ sơ.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người yêu cầu bổ sung thông tin |
| Secondary | System | Chuyển trạng thái, gửi thông báo, ghi audit log |

---

**Preconditions**

- Admin đã đăng nhập vào hệ thống với vai trò `admin`
- Admin đã xem chi tiết hồ sơ xác thực (UC-42)
- Hồ sơ đang có status = PENDING

---

**Trigger**
> Admin nhấn "Yêu cầu bổ sung" trên trang chi tiết hồ sơ.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Admin | Nhấn "Yêu cầu bổ sung" trên trang chi tiết hồ sơ |
| 2 | System | Hiển thị form yêu cầu bổ sung với trường chi tiết (bắt buộc) |
| 3 | Admin | Nhập chi tiết cần bổ sung (ví dụ: "Vui lòng bổ sung giấy giới thiệu của đoàn trường") |
| 4 | Admin | Xác nhận yêu cầu bổ sung |
| 5 | System | Chuyển verification_status của tài khoản sang INFO_REQUIRED |
| 6 | System | Cập nhật VerificationRequest: status = INFO_REQUIRED, info_request_details |
| 7 | System | Ghi nhận AuditLog: action = INFO_REQUESTED, details chứa yêu cầu |
| 8 | System | Gửi email "Hồ sơ cần bổ sung: [info_request_details]" cho người dùng |
| 9 | System | Gửi thông báo in-app tương tự |
| 10 | System | Hiển thị xác nhận "Đã gửi yêu cầu bổ sung thông tin" |
| 11 | System | Chuyển Admin quay lại danh sách hồ sơ chờ duyệt (UC-41) |
| 12 | System | Use case kết thúc thành công |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-45.1: Không nhập chi tiết yêu cầu (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện trường chi tiết yêu cầu trống |
| 4b | System | Hiển thị "Vui lòng nhập chi tiết cần bổ sung" |
| 4c | Admin | Nhập chi tiết và thử lại |

> EF-45.2: Hồ sơ đã được xử lý trước đó (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện hồ sơ không còn ở status PENDING |
| 5b | System | Hiển thị "Hồ sơ đã được xử lý bởi admin khác" |
| 5c | System | Chuyển Admin quay lại danh sách |

---

**Postconditions**

*Success:*

- verification_status = INFO_REQUIRED
- VerificationRequest.status = INFO_REQUIRED với info_request_details
- AuditLog đã ghi nhận hành động yêu cầu bổ sung
- Người dùng nhận thông báo qua email và in-app kèm chi tiết cần bổ sung
- Người dùng có thể bổ sung (UC-38), chỉnh sửa (UC-39), và gửi lại (UC-40)

*Failure:*

- Trạng thái không thay đổi
- Admin được yêu cầu nhập chi tiết

---

**Business Rules**

- BR-1005: Khi yêu cầu bổ sung: (1) info_request_details bắt buộc, (2) chuyển INFO_REQUIRED, (3) người dùng được phép chỉnh sửa và gửi lại
- BR-1006: Gửi thông báo email + in-app khi hồ sơ cần bổ sung thông tin

---

**Notes / Assumptions**

- INFO_REQUIRED là trạng thái trung gian giữa PENDING và REJECTED — không phải từ chối mà là yêu cầu làm rõ
- Chi tiết yêu cầu được hiển thị cho người dùng tham khảo khi chỉnh sửa (UC-39 AF-39.c)
- Liên kết: UC-42 (<<extend>> từ xem chi tiết), UC-38 (bổ sung tài liệu), UC-39 (chỉnh sửa), UC-40 (gửi lại)
