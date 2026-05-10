# Use-Case Specification: UC-45 — Yêu cầu bổ sung thông tin hồ sơ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người yêu cầu bổ sung thông tin |
| Secondary | System | Chuyển trạng thái, gửi thông báo, ghi audit log |

---

### 1. Brief Description

> Admin yêu cầu tổ chức bổ sung thông tin hoặc minh chứng khi hồ sơ xác thực chưa đủ rõ ràng. Hệ thống chuyển trạng thái sang INFO_REQUIRED và gửi thông báo chi tiết cho người dùng.

---

### 2. Flow of Events

**Trigger**
> Admin nhấn "Yêu cầu bổ sung" trên trang chi tiết hồ sơ.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Admin | Nhấn "Yêu cầu bổ sung" |
| 2 | System | Hiển thị form với trường chi tiết (bắt buộc) |
| 3 | Admin | Nhập chi tiết cần bổ sung |
| 4 | Admin | Xác nhận yêu cầu |
| 5 | System | Chuyển verification_status sang INFO_REQUIRED |
| 6 | System | Cập nhật VerificationRequest: status = INFO_REQUIRED, info_request_details |
| 7 | System | Ghi nhận AuditLog: action = INFO_REQUESTED |
| 8 | System | Gửi email + in-app "Hồ sơ cần bổ sung: [info_request_details]" |
| 9 | System | Chuyển Admin quay lại danh sách (UC-41) |
| 10 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-45.1: Không nhập chi tiết yêu cầu

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Hiển thị "Vui lòng nhập chi tiết cần bổ sung" |

##### EF-45.2: Hồ sơ đã được xử lý trước đó

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
| SC-45-01 | Yêu cầu bổ sung thành công | Admin yêu cầu bổ sung kèm chi tiết; người dùng thông báo |

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

- verification_status = INFO_REQUIRED
- VerificationRequest.status = INFO_REQUIRED với info_request_details
- AuditLog đã ghi nhận
- Người dùng nhận thông báo email + in-app kèm chi tiết
- Người dùng có thể bổ sung (UC-38), chỉnh sửa (UC-39), và gửi lại (UC-40)

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

- BR-1005: Khi yêu cầu bổ sung: (1) info_request_details bắt buộc, (2) chuyển INFO_REQUIRED, (3) người dùng được phép chỉnh sửa và gửi lại
- BR-1006: Gửi thông báo email + in-app khi hồ sơ cần bổ sung

---

### 10. Additional Information

**Assumptions:**

- INFO_REQUIRED là trạng thái trung gian — không phải từ chối mà là yêu cầu làm rõ
- Chi tiết yêu cầu hiển thị cho người dùng khi chỉnh sửa (UC-39 AF-39.c)

**Related Use Cases:**

- UC-42: Xem chi tiết hồ sơ (`<<extend>>` base — UC-45 mở rộng UC-42)
- UC-38: Bổ sung tài liệu (sequential — người dùng bổ sung)
- UC-39: Chỉnh sửa hồ sơ (sequential)
- UC-40: Gửi lại hồ sơ (sequential)
