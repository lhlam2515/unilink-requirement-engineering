# Use-Case Specification: UC-09 — Xem chi tiết hồ sơ doanh nghiệp

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người xem hồ sơ doanh nghiệp |
| Secondary | System | Truy xuất và hiển thị dữ liệu hồ sơ |

---

### 1. Brief Description

> Organizer xem toàn bộ thông tin chi tiết của một doanh nghiệp, bao gồm tên, logo, lĩnh vực hoạt động, khu vực, đối tượng khách hàng, mục tiêu tài trợ, và ngân sách tài trợ dự kiến.

---

### 2. Flow of Events

**Trigger**
> Organizer nhấn vào một doanh nghiệp trong danh sách kết quả tìm kiếm (UC-07) hoặc truy cập bằng đường dẫn trực tiếp.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Organizer | Chọn một doanh nghiệp từ danh sách kết quả tìm kiếm |
| 2 | System | Truy xuất hồ sơ doanh nghiệp và kiểm tra trạng thái ACTIVE |
| 3 | System | Hiển thị thông tin doanh nghiệp: tên, logo, lĩnh vực hoạt động, khu vực |
| 4 | System | Hiển thị thông tin tài trợ: đối tượng khách hàng, mục tiêu tài trợ (Marketing/CSR), ngân sách tài trợ dự kiến |
| 5 | System | Use case kết thúc thành công — organizer thấy đầy đủ thông tin doanh nghiệp |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-09.1: Hồ sơ doanh nghiệp không ACTIVE
>
> *Triggered at Step 2 of the Basic Flow when hồ sơ INACTIVE hoặc không tồn tại.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ doanh nghiệp ở trạng thái INACTIVE hoặc không tồn tại |
| 2b | System | Hiển thị thông báo "Hồ sơ doanh nghiệp không tồn tại hoặc đã ngừng hoạt động" |
| 2c | Organizer | Quay lại trang tìm kiếm |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-09-01 | Xem chi tiết thành công | Organizer chọn doanh nghiệp ACTIVE và xem đầy đủ thông tin |
| SC-09-02 | Hồ sơ không khả dụng | Hồ sơ doanh nghiệp INACTIVE hoặc không tồn tại; truy cập không thành công (EF-09.1) |

---

### 5. Preconditions

#### 5.1 Organizer đã xác thực

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`

#### 5.2 Doanh nghiệp đang ACTIVE

- Doanh nghiệp có hồ sơ đang ở trạng thái ACTIVE

---

### 6. Postconditions

#### 6.1 Success

- Organizer xem được toàn bộ thông tin chi tiết doanh nghiệp

#### 6.2 Failure

- Organizer không thể xem hồ sơ và được thông báo lý do

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Sau khi organizer xem chi tiết doanh nghiệp | UC-10: Lưu hồ sơ quan tâm | Organizer muốn lưu hồ sơ doanh nghiệp vào danh sách quan tâm |
| 2 | Sau khi organizer xem chi tiết doanh nghiệp | UC-11: Gửi lời mời tài trợ | Organizer muốn gửi lời mời tài trợ đến doanh nghiệp |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0204: Actor chỉ có thể xem chi tiết hồ sơ ACTIVE. Truy cập hồ sơ INACTIVE bị từ chối

---

### 10. Additional Information

**Assumptions:**

- Hồ sơ doanh nghiệp (BusinessProfile) được quản lý bởi feature ngoài phạm vi quy trình này
- Organizer có thể bookmark doanh nghiệp từ trang chi tiết (UC-10)
- Organizer có thể gửi lời mời tài trợ từ trang chi tiết (UC-11)

**Related Use Cases:**

- UC-07: Tìm kiếm doanh nghiệp để mời tài trợ (`<<include>>` — UC-07 bao gồm UC-09)
- UC-10: Lưu hồ sơ quan tâm (`<<extend>>` — bookmark từ trang chi tiết)
- UC-11: Gửi lời mời tài trợ (`<<extend>>` — gửi lời mời từ trang chi tiết)
