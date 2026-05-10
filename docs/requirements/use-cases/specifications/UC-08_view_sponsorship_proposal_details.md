# Use-Case Specification: UC-08 — Xem chi tiết hồ sơ tài trợ sự kiện

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Sponsor (Doanh nghiệp) | Người xem hồ sơ |
| Secondary | System | Truy xuất và hiển thị dữ liệu hồ sơ |

---

### 1. Brief Description

> Sponsor xem toàn bộ thông tin chi tiết của một hồ sơ tài trợ sự kiện đã phát hành, bao gồm thông tin sự kiện, gói tài trợ, quyền lợi nhà tài trợ, và hình ảnh nhận diện.

---

### 2. Flow of Events

**Trigger**
> Sponsor nhấn vào một sự kiện trong danh sách kết quả tìm kiếm (UC-06) hoặc truy cập bằng đường dẫn trực tiếp.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Sponsor | Chọn một sự kiện từ danh sách kết quả tìm kiếm |
| 2 | System | Truy xuất hồ sơ tài trợ và kiểm tra trạng thái PUBLISHED |
| 3 | System | Hiển thị thông tin cơ bản: tên sự kiện, loại hình, thời gian, địa điểm |
| 4 | System | Hiển thị thông tin chi tiết: quy mô, ngân sách, đối tượng khán giả, nội dung chương trình |
| 5 | System | Hiển thị hình ảnh nhận diện sự kiện (banner, thumbnail) |
| 6 | System | Hiển thị hình thức tài trợ được chấp nhận |
| 7 | System | Hiển thị danh sách gói tài trợ với quyền lợi tương ứng |
| 8 | System | Use case kết thúc thành công — sponsor thấy đầy đủ thông tin hồ sơ |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-08.1: Hồ sơ không ở trạng thái PUBLISHED
>
> *Triggered at Step 2 of the Basic Flow when hồ sơ ở trạng thái DRAFT hoặc không tồn tại.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ ở trạng thái DRAFT hoặc không tồn tại |
| 2b | System | Hiển thị thông báo "Hồ sơ không tồn tại hoặc chưa được phát hành" (HTTP 404) |
| 2c | Sponsor | Quay lại trang tìm kiếm |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-08-01 | Xem chi tiết thành công | Sponsor chọn hồ sơ PUBLISHED và xem đầy đủ thông tin sự kiện, gói tài trợ, quyền lợi |
| SC-08-02 | Hồ sơ không khả dụng | Hồ sơ ở trạng thái DRAFT hoặc không tồn tại; truy cập không thành công (EF-08.1) |

---

### 5. Preconditions

#### 5.1 Sponsor đã xác thực

- Sponsor đã đăng nhập vào hệ thống với vai trò `sponsor`

#### 5.2 Hồ sơ đang PUBLISHED

- Hồ sơ tài trợ đang ở trạng thái PUBLISHED

---

### 6. Postconditions

#### 6.1 Success

- Sponsor xem được toàn bộ thông tin chi tiết hồ sơ tài trợ

#### 6.2 Failure

- Sponsor không thể xem hồ sơ và được thông báo lý do

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Sau khi sponsor xem chi tiết hồ sơ tài trợ | UC-10: Lưu hồ sơ quan tâm | Sponsor muốn lưu hồ sơ vào danh sách quan tâm |
| 2 | Sau khi sponsor xem chi tiết hồ sơ tài trợ | UC-11: Gửi lời mời tài trợ | Sponsor muốn gửi lời mời tài trợ đến BTC |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0204: Actor chỉ có thể xem chi tiết hồ sơ đã phát hành. Truy cập hồ sơ DRAFT bằng đường dẫn trực tiếp bị từ chối

---

### 10. Additional Information

**Assumptions:**

- Sponsor có thể bookmark hồ sơ từ trang chi tiết (UC-10)
- Sponsor có thể gửi lời mời tài trợ từ trang chi tiết (UC-11)

**Related Use Cases:**

- UC-06: Tìm kiếm sự kiện để tài trợ (`<<include>>` — UC-06 bao gồm UC-08)
- UC-10: Lưu hồ sơ quan tâm (`<<extend>>` — bookmark từ trang chi tiết)
- UC-11: Gửi lời mời tài trợ (`<<extend>>` — gửi lời mời từ trang chi tiết)
