# Use-Case Specification: UC-30 — Xem điểm uy tín đối tác

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (bất kỳ) | Người xem điểm uy tín |
| Secondary | System | Truy xuất và hiển thị dữ liệu uy tín |

---

### 1. Brief Description

> Authenticated User xem điểm uy tín tổng hợp và danh sách đánh giá gần đây trên trang hồ sơ công khai của BTC hoặc doanh nghiệp, giúp đánh giá mức độ tin cậy trước khi hợp tác tài trợ. Use case này cũng được tái sử dụng từ public organization profile (SCR-026) như một điểm vào chi tiết.

---

### 2. Flow of Events

**Trigger**
> Actor truy cập trang hồ sơ công khai của BTC hoặc doanh nghiệp.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Truy cập trang hồ sơ công khai của đối tác |
| 2 | System | Truy xuất điểm uy tín tổng hợp từ ReputationScore |
| 3 | System | Hiển thị điểm uy tín trung bình (X.X/5 ⭐), điểm chất lượng trung bình, tổng số đánh giá |
| 4 | System | Hiển thị tối đa 5 đánh giá gần nhất có moderation_status = APPROVED |
| 5 | System | Mỗi đánh giá hiển thị: điểm, nhận xét, thời gian, tên sự kiện liên quan |
| 6 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-30.a: Xem thêm đánh giá
>
> *Triggered at Step 4 of the Basic Flow when actor muốn xem thêm.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Authenticated User | Nhấn "Xem thêm đánh giá" |
| 4b | System | Hiển thị danh sách đánh giá phân trang |

##### AF-30.b: Đối tác chưa có đánh giá
>
> *Triggered at Step 2 of the Basic Flow when đối tác chưa nhận đánh giá nào.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện đối tác chưa nhận đánh giá nào |
| 2b | System | Hiển thị "Chưa có đánh giá" |

#### 2.3 Exception Flows

Không có exception flow đặc biệt cho use case này.

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-30-01 | Xem điểm uy tín | Actor xem điểm tổng hợp và đánh giá gần đây |
| SC-30-02 | Xem thêm đánh giá | Actor xem danh sách đánh giá phân trang (AF-30.a) |
| SC-30-03 | Chưa có đánh giá | Đối tác chưa nhận đánh giá; hiển thị thông báo (AF-30.b) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hồ sơ đối tác tồn tại

- Hồ sơ đối tác (BTC hoặc doanh nghiệp) tồn tại trên hệ thống

---

### 6. Postconditions

#### 6.1 Success

- Actor xem được điểm uy tín và đánh giá của đối tác
- Thông tin giúp actor quyết định hợp tác

#### 6.2 Failure

- Không áp dụng (use case chỉ đọc dữ liệu)

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Khi actor xem đánh giá trên trang uy tín | UC-31: Báo cáo đánh giá vi phạm | Actor phát hiện đánh giá vi phạm |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0706: Trang hồ sơ hiển thị TỐI ĐA 5 đánh giá gần nhất. Có thể xem thêm qua phân trang. Chỉ hiển thị đánh giá APPROVED

---

### 10. Additional Information

**Assumptions:**

- Điểm uy tín được tính tự động và cập nhật real-time khi có đánh giá mới (FR-0703)
- Đánh giá FLAGGED hoặc REMOVED không hiển thị
- Có thể được truy cập từ SCR-005, SCR-006, và SCR-026 (cho Authenticated User)
- SCR-026 (public profile) hiển thị tóm tắt uy tín và cung cấp liên kết "Xem chi tiết uy tín" dẫn sang UC-30/SCR-018 (FR-1104, BR-1105). Chỉ AU truy cập SCR-026.

**Related Use Cases:**

- UC-08: Xem chi tiết hồ sơ tài trợ (sequential — xem uy tín trước khi liên hệ)
- UC-09: Xem chi tiết hồ sơ doanh nghiệp (sequential — xem uy tín trước khi liên hệ)
- UC-29: Gửi đánh giá đối tác (sequential — đánh giá tạo dữ liệu cho UC-30)
- UC-31: Báo cáo đánh giá vi phạm (`<<extend>>` — báo cáo từ trang uy tín)
- UC-46: Xem hồ sơ tổ chức công khai (`<<extend>>` base — UC-30 mở rộng UC-46)
