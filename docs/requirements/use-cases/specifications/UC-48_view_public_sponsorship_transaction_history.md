# Use-Case Specification: UC-48 — Xem lịch sử giao dịch tài trợ công khai

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User | Người xem lịch sử public |
| Secondary | System | Truy xuất và hiển thị lịch sử |

---

### 1. Brief Description

> Authenticated User xem danh sách lịch sử công khai các giao dịch tài trợ đã hoàn tất của một tổ chức theo vai trò Sponsor, nhằm đánh giá mức độ tham gia tài trợ và độ tin cậy công khai của tổ chức.

---

### 2. Flow of Events

**Trigger**
> Người dùng chọn tab "Lịch sử giao dịch" trên public profile của tổ chức.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Chọn tab lịch sử giao dịch |
| 2 | System | Truy xuất danh sách lịch sử giao dịch tài trợ công khai |
| 3 | System | Áp dụng quy tắc lọc public (BR-1102, BR-1106) |
| 4 | System | Hiển thị tối đa 5 mục theo thứ tự mới nhất trước (BR-1103) |
| 5 | System | Hiển thị: tên sự kiện, năm, hình thức tài trợ, trạng thái hoàn tất, thời điểm |
| 6 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-48.a: Không có lịch sử công khai

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Hiển thị "Chưa có lịch sử giao dịch công khai" |

##### AF-48.b: Phân trang (load more)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Authenticated User | Nhấn "Xem thêm" |
| 4b | System | Tải và hiển thị 5 mục tiếp theo |

##### AF-48.c: Áp dụng bộ lọc

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Chọn bộ lọc theo năm hoặc hình thức tài trợ |
| 1b | System | Áp dụng bộ lọc và hiển thị kết quả |

#### 2.3 Exception Flows

##### EF-48.1: Tổ chức không phải Sponsor hoặc chưa VERIFIED

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Hiển thị lỗi 404 (chỉ xảy ra khi truy cập trực tiếp URL) |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-48-01 | Xem lịch sử giao dịch | Actor xem danh sách tối đa 5 mục gần nhất |
| SC-48-02 | Xem thêm | Actor load more để xem tiếp (AF-48.b) |

---

### 5. Preconditions

#### 5.1 Tổ chức đã VERIFIED

- Tổ chức có verification_status = VERIFIED (BR-1101)

#### 5.2 Vai trò Sponsor

- Tổ chức có vai trò Sponsor (BR-1104)

#### 5.3 Public profile đang mở

- UC-46 đã hiển thị thành công

---

### 6. Postconditions

#### 6.1 Success

- Actor xem được lịch sử giao dịch tài trợ công khai

#### 6.2 Failure

- Actor không xem được và nhận thông báo

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1101: Chỉ tổ chức VERIFIED mới hiển thị
- BR-1102: Không hiển thị draft, thương thảo nội bộ, điều khoản hợp đồng, hoặc dữ liệu nhạy cảm
- BR-1103: Tối đa 5 mục gần nhất, page size cố định, hỗ trợ load more
- BR-1104: Chỉ hiển thị lịch sử phù hợp với vai trò Sponsor
- BR-1106: Không hiển thị giá trị tài trợ, điều khoản hợp đồng, số lượng deal

---

### 10. Additional Information

**Assumptions:**

- Đây là lịch sử public, không phải dữ liệu giao dịch nội bộ đầy đủ

**Related Use Cases:**

- UC-46: Xem hồ sơ tổ chức công khai (`<<extend>>` base — UC-48 mở rộng UC-46)
