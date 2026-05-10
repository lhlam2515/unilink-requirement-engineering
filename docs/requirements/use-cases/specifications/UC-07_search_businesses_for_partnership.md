# Use-Case Specification: UC-07 — Tìm kiếm doanh nghiệp để mời tài trợ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người tìm kiếm doanh nghiệp |
| Secondary | System | Thực hiện truy vấn, xếp hạng, phân trang |

---

### 1. Brief Description

> Organizer tìm kiếm các doanh nghiệp phù hợp để mời tài trợ cho sự kiện, dựa trên các bộ lọc như khu vực hoạt động, lĩnh vực, đối tượng khách hàng, ngân sách tài trợ, và mục tiêu tài trợ (Marketing/CSR).

---

### 2. Flow of Events

**Trigger**
> Organizer truy cập trang tìm kiếm doanh nghiệp, nhập từ khóa hoặc áp dụng bộ lọc, và nhấn "Tìm kiếm".

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Organizer | Truy cập trang tìm kiếm doanh nghiệp |
| 2 | System | Hiển thị form tìm kiếm với các bộ lọc: từ khóa, khu vực hoạt động, đối tượng khách hàng, lĩnh vực, ngân sách tài trợ, mục tiêu tài trợ |
| 3 | Organizer | Nhập tiêu chí tìm kiếm (từ khóa và/hoặc bộ lọc) |
| 4 | Organizer | Nhấn "Tìm kiếm" |
| 5 | System | Truy vấn chỉ các doanh nghiệp có hồ sơ ACTIVE |
| 6 | System | Áp dụng bộ lọc theo logic AND |
| 7 | System | Tìm kiếm toàn văn trên tên, mô tả, và lĩnh vực doanh nghiệp |
| 8 | System | Hiển thị danh sách kết quả phân trang: tên doanh nghiệp, logo, lĩnh vực, khu vực, mục tiêu tài trợ, ngân sách dự kiến |
| 9 | System | Use case kết thúc thành công — organizer xem được danh sách doanh nghiệp |

#### 2.2 Alternate Flows

##### AF-07.a: Organizer thay đổi tiêu chí sắp xếp
>
> *Triggered at Step 7 of the Basic Flow when organizer chọn tiêu chí sắp xếp khác.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | Organizer | Chọn tiêu chí sắp xếp: Mức độ phù hợp, Ngân sách, Lĩnh vực |
| 7b | System | Sắp xếp lại kết quả. Tiếp tục tại Step 8 |

#### 2.3 Exception Flows

##### EF-07.1: Không tìm thấy doanh nghiệp phù hợp
>
> *Triggered at Step 6 of the Basic Flow when không có doanh nghiệp nào phù hợp.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Không có doanh nghiệp nào phù hợp với tiêu chí |
| 6b | System | Hiển thị thông báo "Không tìm thấy doanh nghiệp phù hợp" |
| 6c | Organizer | Có thể thay đổi tiêu chí hoặc xóa bộ lọc |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-07-01 | Tìm kiếm có kết quả | Organizer áp dụng bộ lọc; danh sách doanh nghiệp ACTIVE phù hợp được trả về |
| SC-07-02 | Không có kết quả | Không có doanh nghiệp nào phù hợp tiêu chí; organizer điều chỉnh bộ lọc (EF-07.1) |

---

### 5. Preconditions

#### 5.1 Organizer đã xác thực

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`

#### 5.2 Có doanh nghiệp khả dụng

- Có ít nhất một doanh nghiệp có hồ sơ đang ACTIVE trong hệ thống

---

### 6. Postconditions

#### 6.1 Success

- Organizer nhận được danh sách doanh nghiệp phù hợp
- Kết quả chỉ bao gồm doanh nghiệp ACTIVE, được phân trang và sắp xếp

#### 6.2 Failure

- Không có kết quả — organizer được thông báo

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0201: Kết quả chỉ bao gồm doanh nghiệp có hồ sơ ACTIVE
- BR-0203: Bộ lọc bao gồm: khu vực hoạt động, đối tượng khách hàng, lĩnh vực, ngân sách tài trợ (range), mục tiêu tài trợ (MARKETING | CSR). Kết hợp AND logic

---

### 10. Additional Information

**Assumptions:**

- Hồ sơ doanh nghiệp (BusinessProfile) được quản lý bởi feature riêng ngoài phạm vi
- Organizer có thể xem chi tiết doanh nghiệp thông qua UC-09
- Organizer có thể bookmark doanh nghiệp thông qua UC-10

**Related Use Cases:**

- UC-09: Xem chi tiết hồ sơ doanh nghiệp (`<<include>>` — organizer luôn cần xem chi tiết ít nhất một doanh nghiệp)
- UC-10: Lưu hồ sơ quan tâm (sequential — sau khi xem kết quả)
- UC-11: Gửi lời mời tài trợ (sequential — sau khi xem chi tiết)
