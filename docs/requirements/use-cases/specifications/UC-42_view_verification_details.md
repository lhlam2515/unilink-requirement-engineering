# Use-Case Specification: UC-42 — Xem chi tiết hồ sơ xác thực

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người xem xét hồ sơ để đánh giá |
| Secondary | System | Truy xuất và hiển thị chi tiết hồ sơ |

---

### 1. Brief Description

> Admin xem toàn bộ thông tin chi tiết của một hồ sơ xác thực tổ chức, bao gồm thông tin cơ bản, thông tin bổ sung, tài liệu minh chứng (xem/tải về), và lịch sử các lần gửi/xử lý trước đó. Đây là bước bắt buộc trước khi phê duyệt, từ chối, hoặc yêu cầu bổ sung.

---

### 2. Flow of Events

**Trigger**
> Admin nhấn vào một hồ sơ trong danh sách chờ duyệt (UC-41).

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Admin | Nhấn vào hồ sơ cần xem chi tiết |
| 2 | System | Truy xuất thông tin đầy đủ của hồ sơ xác thực |
| 3 | System | Hiển thị thông tin cơ bản: tên tổ chức, vai trò, email, địa chỉ liên hệ |
| 4 | System | Hiển thị thông tin bổ sung theo vai trò |
| 5 | System | Hiển thị danh sách tài liệu minh chứng với nút preview/download |
| 6 | System | Hiển thị lịch sử xác thực (tất cả lần gửi và xử lý theo thứ tự thời gian) |
| 7 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-42.a: Xem trước tài liệu minh chứng

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Admin | Nhấn "Xem trước" → System hiển thị preview (PDF/image viewer) |

##### AF-42.b: Tải về tài liệu minh chứng

| Step | Actor / System | Action |
|------|----------------|--------|
| 5c | Admin | Nhấn "Tải về" → System tải file về máy Admin |

##### AF-42.c: Hồ sơ Sponsor có mã số thuế

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Hiển thị mã số thuế của doanh nghiệp |
| 4b | Admin | Đối chiếu MST thủ công với nguồn bên ngoài |

##### AF-42.d: Hồ sơ đã từng bị xử lý trước đó

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Hiển thị lịch sử: lần gửi trước + quyết định + lý do + lần gửi hiện tại |

##### AF-42.e: Hồ sơ đã được cập nhật trong khi PENDING_REVIEW

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Hiển thị badge "Hồ sơ đã được cập nhật" |
| 3b | System | Hiển thị field-level diff (giá trị cũ → giá trị mới) |

#### 2.3 Exception Flows

##### EF-42.1: Hồ sơ không tồn tại

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Hiển thị "Hồ sơ không tồn tại hoặc đã bị xóa" |
| 2b | System | Chuyển Admin quay lại danh sách (UC-41) |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-42-01 | Xem chi tiết hồ sơ | Admin xem thông tin, tài liệu, và lịch sử xác thực |
| SC-42-02 | Hồ sơ có cập nhật | Admin thấy badge "đã cập nhật" và field-level diff (AF-42.e) |

---

### 5. Preconditions

#### 5.1 Admin đã xác thực

- Admin đã đăng nhập với vai trò `admin`

#### 5.2 Hồ sơ tồn tại

- Hồ sơ xác thực tồn tại trong hệ thống

---

### 6. Postconditions

#### 6.1 Success

- Admin đã xem đầy đủ thông tin hồ sơ
- Admin sẵn sàng ra quyết định: phê duyệt (UC-43), từ chối (UC-44), hoặc yêu cầu bổ sung (UC-45)

#### 6.2 Failure

- Không hiển thị được; Admin quay lại danh sách

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Admin quyết định phê duyệt | UC-43: Phê duyệt hồ sơ tổ chức | Admin nhấn "Phê duyệt" |
| 2 | Admin quyết định từ chối | UC-44: Từ chối hồ sơ tổ chức | Admin nhấn "Từ chối" |
| 3 | Admin yêu cầu bổ sung | UC-45: Yêu cầu bổ sung thông tin | Admin nhấn "Yêu cầu bổ sung" |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1002: Admin phải có quyền xem và tải về tất cả tài liệu minh chứng. Lịch sử xác thực hiển thị TẤT CẢ lần gửi/xử lý. Nếu hồ sơ cập nhật trong PENDING_REVIEW, hiển thị field-level diff

---

### 10. Additional Information

**Assumptions:**

- Kiểm tra MST được thực hiện THỦ CÔNG bên ngoài hệ thống

**Related Use Cases:**

- UC-41: Xem danh sách hồ sơ (`<<include>>` — từ danh sách)
- UC-43: Phê duyệt hồ sơ (`<<extend>>`)
- UC-44: Từ chối hồ sơ (`<<extend>>`)
- UC-45: Yêu cầu bổ sung (`<<extend>>`)
