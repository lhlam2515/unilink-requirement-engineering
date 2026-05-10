# Use-Case Specification: UC-47 — Xem lịch sử hồ sơ tài trợ công khai

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User | Người xem lịch sử public |
| Secondary | System | Truy xuất và hiển thị lịch sử |

---

### 1. Brief Description

> Authenticated User xem danh sách lịch sử công khai các hồ sơ tài trợ đã từng thực hiện của một tổ chức theo vai trò Organizer, để đánh giá năng lực tổ chức và mức độ hoạt động thực tế.

---

### 2. Flow of Events

**Trigger**
> Người dùng chọn tab "Lịch sử tài trợ" trên public profile của tổ chức.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Chọn tab lịch sử tài trợ |
| 2 | System | Truy xuất danh sách lịch sử tài trợ công khai |
| 3 | System | Áp dụng quy tắc lọc public (BR-1102, BR-1106) |
| 4 | System | Hiển thị tối đa 5 mục theo thứ tự mới nhất trước (BR-1103) |
| 5 | System | Hiển thị: tên sự kiện, năm, trạng thái, nhãn tóm tắt, thời điểm |
| 6 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-47.a: Không có lịch sử công khai

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Hiển thị "Chưa có lịch sử tài trợ công khai" |

##### AF-47.b: Phân trang (load more)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Authenticated User | Nhấn "Xem thêm" |
| 4b | System | Tải và hiển thị 5 mục tiếp theo |

##### AF-47.c: Áp dụng bộ lọc

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Chọn bộ lọc theo năm hoặc trạng thái |
| 1b | System | Áp dụng bộ lọc và hiển thị kết quả |

#### 2.3 Exception Flows

##### EF-47.1: Tổ chức không phải Organizer hoặc chưa VERIFIED

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
| SC-47-01 | Xem lịch sử tài trợ | Actor xem danh sách tối đa 5 mục gần nhất |
| SC-47-02 | Xem thêm | Actor load more để xem tiếp (AF-47.b) |

---

### 5. Preconditions

#### 5.1 Tổ chức đã VERIFIED

- Tổ chức có verification_status = VERIFIED (BR-1101)

#### 5.2 Vai trò Organizer

- Tổ chức có vai trò Organizer (BR-1104)

#### 5.3 Public profile đang mở

- UC-46 đã hiển thị thành công

---

### 6. Postconditions

#### 6.1 Success

- Actor xem được lịch sử tài trợ công khai

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
- BR-1102: Không hiển thị draft, thương thảo nội bộ, hoặc dữ liệu nhạy cảm
- BR-1103: Tối đa 5 mục gần nhất, page size cố định, hỗ trợ load more
- BR-1104: Chỉ hiển thị lịch sử phù hợp với vai trò Organizer
- BR-1106: Không hiển thị giá trị tài trợ, điều khoản hợp đồng, số lượng deal

---

### 10. Additional Information

**Assumptions:**

- Đây là lịch sử public, không phải danh sách đầy đủ của hệ thống nội bộ

**Related Use Cases:**

- UC-46: Xem hồ sơ tổ chức công khai (`<<extend>>` base — UC-47 mở rộng UC-46)
