# Use-Case Specification: UC-41 — Xem danh sách hồ sơ chờ kiểm duyệt

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người kiểm duyệt hồ sơ tổ chức |
| Secondary | System | Truy xuất và hiển thị danh sách hồ sơ |

---

### 1. Brief Description

> Admin xem danh sách tất cả hồ sơ xác thực tổ chức đang chờ kiểm duyệt. Danh sách hiển thị thông tin tóm tắt, sắp xếp theo thứ tự ưu tiên (FIFO), và hỗ trợ lọc theo vai trò cùng tìm kiếm theo tên tổ chức.

---

### 2. Flow of Events

**Trigger**
> Admin truy cập trang "Kiểm duyệt hồ sơ" từ admin panel.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Admin | Truy cập trang "Kiểm duyệt hồ sơ" |
| 2 | System | Truy xuất danh sách hồ sơ xác thực có status = PENDING |
| 3 | System | Sắp xếp theo submitted_at ASC (hồ sơ gửi trước hiển thị trước) |
| 4 | System | Hiển thị danh sách: tên tổ chức, vai trò, ngày gửi, thời gian chờ xử lý |
| 5 | System | Hiển thị tổng số hồ sơ pending và thời gian chờ trung bình |
| 6 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-41.a: Lọc theo vai trò

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Admin | Chọn bộ lọc: Organizer, Sponsor, hoặc Tất cả |
| 4b | System | Áp dụng bộ lọc và hiển thị kết quả |

##### AF-41.b: Tìm kiếm theo tên tổ chức

| Step | Actor / System | Action |
|------|----------------|--------|
| 4c | Admin | Nhập từ khóa tìm kiếm |
| 4d | System | Hiển thị hồ sơ có tên chứa từ khóa |

##### AF-41.c: Phân trang

| Step | Actor / System | Action |
|------|----------------|--------|
| 4e | System | Hiển thị tối đa 20 hồ sơ mỗi trang (BR-1001) |
| 4f | Admin | Chuyển trang để xem thêm |

##### AF-41.d: Không có hồ sơ pending

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Hiển thị "Không có hồ sơ nào đang chờ kiểm duyệt" |

#### 2.3 Exception Flows

##### EF-41.1: Lỗi truy xuất dữ liệu

| Step | Actor / System | Action |
|------|----------------|--------|
| 2c | System | Hiển thị "Không thể tải danh sách. Vui lòng thử lại sau." |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-41-01 | Xem danh sách | Admin xem danh sách hồ sơ pending, sắp xếp FIFO |
| SC-41-02 | Lọc và tìm kiếm | Admin lọc theo vai trò và/hoặc tìm kiếm (AF-41.a, AF-41.b) |

---

### 5. Preconditions

#### 5.1 Admin đã xác thực

- Admin đã đăng nhập với vai trò `admin`

#### 5.2 Có hồ sơ pending

- Có ít nhất một hồ sơ với status = PENDING

---

### 6. Postconditions

#### 6.1 Success

- Admin xem được danh sách hồ sơ chờ duyệt
- Admin có thể chọn hồ sơ để xem chi tiết (UC-42)

#### 6.2 Failure

- Danh sách không hiển thị

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1001: Danh sách sắp xếp theo submitted_at ASC (FIFO). Phân trang 20 hồ sơ/trang

---

### 10. Additional Information

**Assumptions:**

- Danh sách chỉ hiển thị hồ sơ PENDING — hồ sơ đã xử lý xem qua bộ lọc riêng
- Admin nhấn vào hồ sơ để xem chi tiết (UC-42)

**Related Use Cases:**

- UC-40: Gửi hồ sơ xác thực (sequential — hồ sơ đã gửi)
- UC-42: Xem chi tiết hồ sơ xác thực (`<<include>>` — xem chi tiết)
