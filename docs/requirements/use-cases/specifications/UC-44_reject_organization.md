# Use-Case Specification: UC-44 — Từ chối hồ sơ tổ chức

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người từ chối hồ sơ |
| Secondary | System | Chuyển trạng thái, gửi thông báo, ghi audit log |

---

### 1. Brief Description

> Admin từ chối hồ sơ xác thực tổ chức kèm theo lý do rõ ràng. Khi từ chối, hệ thống chuyển trạng thái sang REJECTED và gửi thông báo cho người dùng. Người dùng không cần tạo lại tài khoản — chỉ cần bổ sung/chỉnh sửa và gửi lại hồ sơ.

---

### 2. Flow of Events

**Trigger**
> Admin nhấn "Từ chối" trên trang chi tiết hồ sơ.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Admin | Nhấn "Từ chối" trên trang chi tiết hồ sơ |
| 2 | System | Hiển thị form từ chối với trường lý do (bắt buộc) |
| 3 | Admin | Nhập lý do từ chối rõ ràng |
| 4 | Admin | Xác nhận từ chối |
| 5 | System | Chuyển verification_status sang REJECTED |
| 6 | System | Cập nhật VerificationRequest: status = REJECTED, rejection_reason, reviewed_at, reviewed_by |
| 7 | System | Ghi nhận AuditLog: action = REJECTED, details chứa lý do |
| 8 | System | Gửi email + in-app "Hồ sơ bị từ chối. Lý do: [rejection_reason]. Bạn có thể gửi lại." |
| 9 | System | Chuyển Admin quay lại danh sách (UC-41) |
| 10 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-44.1: Không nhập lý do từ chối

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Hiển thị "Vui lòng nhập lý do từ chối" |

##### EF-44.2: Hồ sơ đã được xử lý trước đó

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Hiển thị "Hồ sơ đã được xử lý bởi admin khác" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-44-01 | Từ chối thành công | Admin từ chối kèm lý do; người dùng thông báo, có thể gửi lại |

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

- verification_status = REJECTED
- VerificationRequest.status = REJECTED với rejection_reason
- AuditLog đã ghi nhận
- Người dùng nhận thông báo email + in-app kèm lý do và hướng dẫn gửi lại
- Người dùng có thể chỉnh sửa (UC-39) và gửi lại (UC-40)

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

- BR-1004: Khi từ chối: (1) rejection_reason bắt buộc, (2) người dùng không cần tạo lại tài khoản, (3) chuyển REJECTED
- BR-1006: Gửi thông báo email + in-app khi từ chối kèm lý do và hướng dẫn gửi lại

---

### 10. Additional Information

**Assumptions:**

- Từ chối không xóa tài khoản — người dùng có full quyền chỉnh sửa và gửi lại
- Lý do từ chối hiển thị cho người dùng khi chỉnh sửa (UC-39 AF-39.b)

**Related Use Cases:**

- UC-42: Xem chi tiết hồ sơ (`<<extend>>` base — UC-44 mở rộng UC-42)
- UC-39: Chỉnh sửa hồ sơ (sequential — sau từ chối)
- UC-40: Gửi lại hồ sơ (sequential — sau khi sửa)
