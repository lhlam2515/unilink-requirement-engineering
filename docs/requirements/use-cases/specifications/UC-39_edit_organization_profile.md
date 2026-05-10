# Use-Case Specification: UC-39 — Chỉnh sửa hồ sơ tổ chức

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người chỉnh sửa hồ sơ |
| Secondary | System | Kiểm tra quyền chỉnh sửa, cập nhật hồ sơ |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) chỉnh sửa thông tin hồ sơ tổ chức đã nhập. Quyền chỉnh sửa phụ thuộc vào trạng thái xác thực: cho phép toàn bộ khi UNVERIFIED/REJECTED/INFO_REQUIRED, cho phép kèm đánh dấu khi PENDING_REVIEW, và không cho phép khi VERIFIED.

---

### 2. Flow of Events

**Trigger**
> Actor truy cập trang "Hồ sơ tổ chức" và nhấn "Chỉnh sửa".

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
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

#### 2.2 Alternate Flows

##### AF-39.a: Chỉnh sửa khi PENDING_REVIEW

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện verification_status = PENDING_REVIEW |
| 4b | System | Cảnh báo "Hồ sơ đang chờ kiểm duyệt. Thay đổi sẽ được ghi nhận và gửi thông báo cho kiểm duyệt viên." |
| 9a | System | Ghi nhận field-level diff, đánh dấu "đã cập nhật kể từ lần gửi cuối" |
| 9b | System | Gửi thông báo cho Admin (in-app + email) kèm danh sách trường thay đổi |

##### AF-39.b: Chỉnh sửa khi REJECTED

| Step | Actor / System | Action |
|------|----------------|--------|
| 4c | System | Hiển thị lý do từ chối từ lần kiểm duyệt gần nhất |
| 5b | System | Hiển thị form cho phép sửa tất cả. Tiếp tục từ Step 6 |

##### AF-39.c: Chỉnh sửa khi INFO_REQUIRED

| Step | Actor / System | Action |
|------|----------------|--------|
| 4e | System | Hiển thị chi tiết yêu cầu bổ sung từ Admin |
| 5c | System | Hiển thị form cho phép sửa tất cả. Tiếp tục từ Step 6 |

#### 2.3 Exception Flows

##### EF-39.1: Hồ sơ đã VERIFIED

| Step | Actor / System | Action |
|------|----------------|--------|
| 4g | System | Hiển thị "Hồ sơ đã xác thực không thể chỉnh sửa trực tiếp" |

##### EF-39.2: Thiếu trường bắt buộc sau chỉnh sửa

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | System | Hiển thị lỗi cụ thể cho từng trường thiếu |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-39-01 | Chỉnh sửa UNVERIFIED | Actor chỉnh sửa hồ sơ chưa xác thực; cập nhật thành công |
| SC-39-02 | Chỉnh sửa PENDING_REVIEW | Admin được thông báo kèm field-level diff (AF-39.a) |
| SC-39-03 | Chỉnh sửa sau REJECTED | Actor xem lý do từ chối và chỉnh sửa (AF-39.b) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hồ sơ tổ chức tồn tại

- Actor đã có hồ sơ tổ chức (Organization entity tồn tại)

---

### 6. Postconditions

#### 6.1 Success

- Hồ sơ tổ chức đã được cập nhật
- Nếu đang PENDING_REVIEW: hồ sơ đánh dấu "đã cập nhật" kèm field-level diff, Admin được thông báo

#### 6.2 Failure

- Hồ sơ không thay đổi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0903: Quyền chỉnh sửa phụ thuộc verification_status: UNVERIFIED → sửa tất cả; PENDING_REVIEW → sửa kèm đánh dấu; VERIFIED → KHÔNG sửa; REJECTED/INFO_REQUIRED → sửa tất cả

---

### 10. Additional Information

**Assumptions:**

- Hồ sơ VERIFIED cần liên hệ hỗ trợ (ngoài phạm vi hiện tại)

**Related Use Cases:**

- UC-38: Bổ sung thông tin và tài liệu (sequential)
- UC-40: Gửi hồ sơ xác thực (sequential — gửi lại sau khi chỉnh sửa)
