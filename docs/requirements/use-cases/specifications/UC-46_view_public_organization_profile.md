# Use-Case Specification: UC-46 — Xem hồ sơ tổ chức công khai

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User | Người xem public profile (đã đăng nhập) |
| Secondary | System | Truy xuất và hiển thị dữ liệu public |

---

### 1. Brief Description

> Authenticated User xem hồ sơ tổ chức công khai để nhận diện tổ chức, kiểm tra trạng thái xác thực, xem tóm tắt uy tín (có thể điều hướng sang chi tiết), và truy cập lịch sử công khai phù hợp theo vai trò duy nhất của tổ chức.

---

### 2. Flow of Events

**Trigger**
> Người dùng mở đường dẫn public profile hoặc nhấn vào liên kết công khai tới hồ sơ tổ chức.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Mở trang hồ sơ tổ chức công khai |
| 2 | System | Truy xuất hồ sơ public và kiểm tra verification_status = VERIFIED |
| 3 | System | Hiển thị tên tổ chức, logo, vai trò, trạng thái xác thực, khu vực hoạt động, mô tả ngắn |
| 4 | System | Hiển thị tóm tắt uy tín: điểm trung bình, tổng số đánh giá, liên kết "Xem chi tiết uy tín" → UC-30 |
| 5 | System | Hiển thị khu vực lịch sử công khai theo vai trò (tab tài trợ cho Organizer, tab giao dịch cho Sponsor) |
| 6 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-46.a: Tổ chức chưa có lịch sử công khai

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Hiển thị empty state phù hợp |

##### AF-46.b: Tổ chức chưa có đánh giá

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Hiển thị "Chưa có đánh giá", ẩn liên kết chi tiết |

##### AF-46.c: Xem chi tiết uy tín

| Step | Actor / System | Action |
|------|----------------|--------|
| 4c | Authenticated User | Nhấn "Xem chi tiết uy tín" |
| 4d | System | Điều hướng sang SCR-018 — kích hoạt UC-30 (`<<extend>>`) |

#### 2.3 Exception Flows

##### EF-46.1: Hồ sơ không tồn tại hoặc chưa VERIFIED

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Hiển thị trang 404 hoặc thông báo không khả dụng |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-46-01 | Xem public profile | Actor xem thông tin, uy tín, và lịch sử công khai |
| SC-46-02 | Xem chi tiết uy tín | Actor nhấn "Xem chi tiết uy tín" → UC-30 (AF-46.c) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Tổ chức đã VERIFIED

- Tổ chức có verification_status = VERIFIED (BR-1101)

---

### 6. Postconditions

#### 6.1 Success

- Actor xem được hồ sơ công khai, tóm tắt uy tín, và lịch sử

#### 6.2 Failure

- Actor không thể xem và được thông báo lý do

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Xem chi tiết uy tín | UC-30: Xem điểm uy tín đối tác | Actor nhấn "Xem chi tiết uy tín" |
| 2 | Xem lịch sử tài trợ (Organizer) | UC-47: Xem lịch sử hồ sơ tài trợ công khai | Tổ chức là Organizer |
| 3 | Xem lịch sử giao dịch (Sponsor) | UC-48: Xem lịch sử giao dịch tài trợ công khai | Tổ chức là Sponsor |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1101: Chỉ tổ chức VERIFIED mới được hiển thị hồ sơ công khai
- BR-1102: Chỉ dữ liệu công khai mới được phép hiển thị
- BR-1104: Mỗi tổ chức chỉ có một vai trò — chỉ hiển thị loại lịch sử tương ứng
- BR-1105: Tóm tắt uy tín + liên kết sang SCR-018 (UC-30)

---

### 10. Additional Information

**Assumptions:**

- Mỗi tổ chức chỉ có một vai trò (BR-1104), do đó chỉ có một loại lịch sử
- Chỉ Authenticated User mới truy cập được public profile

**Related Use Cases:**

- UC-30: Xem điểm uy tín (`<<extend>>` — chi tiết uy tín)
- UC-47: Xem lịch sử tài trợ (`<<extend>>` — Organizer)
- UC-48: Xem lịch sử giao dịch (`<<extend>>` — Sponsor)
