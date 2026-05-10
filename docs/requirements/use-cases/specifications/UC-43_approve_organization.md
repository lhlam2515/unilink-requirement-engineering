# Use-Case Specification: UC-43 — Phê duyệt hồ sơ tổ chức

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người phê duyệt hồ sơ |
| Secondary | System | Chuyển trạng thái, mở khóa quyền, gửi thông báo, ghi audit log |

---

### 1. Brief Description

> Admin phê duyệt hồ sơ xác thực tổ chức sau khi xem xét chi tiết. Khi phê duyệt, hệ thống chuyển trạng thái sang VERIFIED, tự động mở khóa quyền truy cập theo vai trò, thiết lập thời hạn lưu trữ tài liệu, và gửi thông báo cho người dùng.

---

### 2. Flow of Events

**Trigger**
> Admin nhấn "Phê duyệt" trên trang chi tiết hồ sơ.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Admin | Nhấn "Phê duyệt" trên trang chi tiết hồ sơ |
| 2 | System | Hiển thị xác nhận kèm trường ghi chú tùy chọn |
| 3 | Admin | Nhập ghi chú (tùy chọn) và xác nhận |
| 4 | System | Chuyển verification_status sang VERIFIED |
| 5 | System | Cập nhật VerificationRequest: status = APPROVED, reviewed_at, reviewed_by |
| 6 | System | Tự động mở khóa quyền truy cập theo vai trò (FR-0904) |
| 7 | System | Thiết lập retention_expires_at = reviewed_at + 7 ngày (BR-0907) |
| 8 | System | Ghi nhận AuditLog: action = APPROVED |
| 9 | System | Gửi email + in-app "Hồ sơ tổ chức đã được xác thực" cho người dùng |
| 10 | System | Chuyển Admin quay lại danh sách (UC-41) |
| 11 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-43.1: Hồ sơ đã được xử lý trước đó

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Hiển thị "Hồ sơ đã được xử lý bởi admin khác" |

##### EF-43.2: Tài khoản đã VERIFIED

| Step | Actor / System | Action |
|------|----------------|--------|
| 4d | System | Hiển thị "Tài khoản đã được xác thực" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-43-01 | Phê duyệt thành công | Admin phê duyệt; VERIFIED, quyền mở khóa, người dùng thông báo |

---

### 5. Preconditions

#### 5.1 Admin đã xác thực

- Admin đã đăng nhập với vai trò `admin`

#### 5.2 Đã xem chi tiết

- Admin đã xem chi tiết hồ sơ (UC-42)

#### 5.3 Hồ sơ đang PENDING

- Hồ sơ đang có status = PENDING

---

### 6. Postconditions

#### 6.1 Success

- verification_status = VERIFIED
- VerificationRequest.status = APPROVED
- Quyền truy cập đã mở khóa
- retention_expires_at đã thiết lập
- AuditLog đã ghi nhận
- Người dùng nhận thông báo email + in-app

#### 6.2 Failure

- Trạng thái không thay đổi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1003: Khi phê duyệt: (1) chuyển VERIFIED, (2) mở khóa quyền tự động, (3) retention_expires_at = reviewed_at + 7 ngày
- BR-1006: Gửi thông báo email + in-app khi phê duyệt

---

### 10. Additional Information

**Assumptions:**

- Phê duyệt không thể hoàn tác (sau VERIFIED, hồ sơ không thể chỉnh sửa trực tiếp)

**Related Use Cases:**

- UC-42: Xem chi tiết hồ sơ (`<<extend>>` base — UC-43 mở rộng UC-42)
- UC-41: Xem danh sách (sequential — quay lại)
