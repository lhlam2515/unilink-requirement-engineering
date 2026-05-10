# Use-Case Specification: UC-06 — Tìm kiếm sự kiện để tài trợ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Sponsor (Doanh nghiệp) | Người tìm kiếm sự kiện |
| Secondary | System | Thực hiện truy vấn tìm kiếm, xếp hạng kết quả, phân trang |

---

### 1. Brief Description

> Sponsor tìm kiếm các hồ sơ tài trợ sự kiện đã phát hành phù hợp với nhu cầu tài trợ của doanh nghiệp, dựa trên các bộ lọc như địa điểm, quy mô, ngân sách, đối tượng khán giả, và hình thức tài trợ.

---

### 2. Flow of Events

**Trigger**
> Sponsor truy cập trang tìm kiếm sự kiện, nhập từ khóa hoặc áp dụng bộ lọc, và nhấn "Tìm kiếm".

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Sponsor | Truy cập trang tìm kiếm sự kiện |
| 2 | System | Hiển thị form tìm kiếm với các bộ lọc: từ khóa, địa điểm, quy mô, ngân sách, đối tượng khán giả, hình thức tài trợ |
| 3 | Sponsor | Nhập tiêu chí tìm kiếm (từ khóa và/hoặc bộ lọc) |
| 4 | Sponsor | Nhấn "Tìm kiếm" hoặc áp dụng bộ lọc |
| 5 | System | Truy vấn chỉ các hồ sơ có trạng thái PUBLISHED |
| 6 | System | Áp dụng bộ lọc theo logic AND (kết hợp tất cả điều kiện) |
| 7 | System | Sắp xếp kết quả theo mức độ phù hợp (hoặc tiêu chí sắp xếp do sponsor chọn) |
| 8 | System | Hiển thị danh sách kết quả phân trang: tên sự kiện, loại hình, thumbnail, địa điểm, thời gian, quy mô, ngân sách, hình thức tài trợ |
| 9 | System | Use case kết thúc thành công — sponsor xem được danh sách sự kiện |

#### 2.2 Alternate Flows

##### AF-06.a: Sponsor thay đổi tiêu chí sắp xếp
>
> *Triggered at Step 7 of the Basic Flow when sponsor chọn tiêu chí sắp xếp khác.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | Sponsor | Chọn tiêu chí sắp xếp: Mức độ phù hợp, Ngày tổ chức, Quy mô, Ngân sách |
| 7b | System | Sắp xếp lại kết quả theo tiêu chí mới. Tiếp tục tại Step 8 |

##### AF-06.b: Sponsor điều hướng phân trang
>
> *Triggered at Step 8 of the Basic Flow when sponsor chuyển trang.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | Sponsor | Chuyển trang hoặc thay đổi số lượng kết quả trên mỗi trang |
| 8b | System | Tải và hiển thị trang kết quả tương ứng |

#### 2.3 Exception Flows

##### EF-06.1: Không tìm thấy sự kiện phù hợp
>
> *Triggered at Step 6 of the Basic Flow when không có hồ sơ nào phù hợp.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Không có hồ sơ nào phù hợp với tiêu chí tìm kiếm |
| 6b | System | Hiển thị thông báo "Không tìm thấy sự kiện phù hợp" |
| 6c | Sponsor | Có thể thay đổi tiêu chí tìm kiếm hoặc xóa bộ lọc |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-06-01 | Tìm kiếm có kết quả | Sponsor áp dụng bộ lọc; danh sách hồ sơ PUBLISHED phù hợp được trả về |
| SC-06-02 | Không có kết quả | Không có hồ sơ nào phù hợp tiêu chí; sponsor điều chỉnh bộ lọc (EF-06.1) |
| SC-06-03 | Thay đổi sắp xếp | Sponsor thay đổi tiêu chí sắp xếp kết quả (AF-06.a) |

---

### 5. Preconditions

#### 5.1 Sponsor đã xác thực

- Sponsor đã đăng nhập vào hệ thống với vai trò `sponsor`

#### 5.2 Có hồ sơ khả dụng

- Có ít nhất một hồ sơ tài trợ đang ở trạng thái PUBLISHED trong hệ thống

---

### 6. Postconditions

#### 6.1 Success

- Sponsor nhận được danh sách hồ sơ tài trợ sự kiện phù hợp
- Kết quả chỉ bao gồm hồ sơ PUBLISHED
- Kết quả được phân trang và sắp xếp

#### 6.2 Failure

- Không có kết quả — sponsor được thông báo và có thể điều chỉnh tiêu chí

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0201: Kết quả tìm kiếm chỉ bao gồm hồ sơ ở trạng thái PUBLISHED
- BR-0202: Bộ lọc bao gồm: địa điểm, quy mô (range), đối tượng khán giả (keywords), ngân sách (range), hình thức tài trợ (multi-select). Các bộ lọc kết hợp AND logic

---

### 10. Additional Information

**Assumptions:**

- Hệ thống sử dụng tìm kiếm toàn văn (full-text search) cho trường văn bản
- Sponsor có thể xem chi tiết hồ sơ thông qua UC-08
- Sponsor có thể bookmark hồ sơ quan tâm thông qua UC-10

**Related Use Cases:**

- UC-08: Xem chi tiết hồ sơ tài trợ sự kiện (`<<include>>` — sponsor luôn cần xem chi tiết ít nhất một hồ sơ)
- UC-10: Lưu hồ sơ quan tâm (sequential — sau khi xem kết quả)
- UC-11: Gửi lời mời tài trợ (sequential — sau khi xem chi tiết)
