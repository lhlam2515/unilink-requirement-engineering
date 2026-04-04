# UC-30: Xem điểm uy tín đối tác

**Brief Description**
> Authenticated User xem điểm uy tín tổng hợp và danh sách đánh giá gần đây trên trang hồ sơ công khai của BTC hoặc doanh nghiệp, giúp đánh giá mức độ tin cậy trước khi hợp tác tài trợ. Use case này cũng được tái sử dụng từ public organization profile (SCR-026) như một điểm vào chi tiết.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (bất kỳ) | Người xem điểm uy tín |
| Secondary | System | Truy xuất và hiển thị dữ liệu uy tín |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hồ sơ đối tác (BTC hoặc doanh nghiệp) tồn tại trên hệ thống

---

**Trigger**
> Actor truy cập trang hồ sơ công khai của BTC hoặc doanh nghiệp.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Truy cập trang hồ sơ công khai của đối tác |
| 2 | System | Truy xuất điểm uy tín tổng hợp từ ReputationScore |
| 3 | System | Hiển thị điểm uy tín trung bình (X.X/5 ⭐), điểm chất lượng trung bình, tổng số đánh giá |
| 4 | System | Hiển thị tối đa 5 đánh giá gần nhất có moderation_status = APPROVED |
| 5 | System | Mỗi đánh giá hiển thị: điểm, nhận xét, thời gian, tên sự kiện liên quan |
| 6 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-30.a: Xem thêm đánh giá (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Authenticated User | Nhấn "Xem thêm đánh giá" |
| 4b | System | Hiển thị danh sách đánh giá phân trang |

> AF-30.b: Đối tác chưa có đánh giá (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện đối tác chưa nhận đánh giá nào |
| 2b | System | Hiển thị "Chưa có đánh giá" |

---

**Exception Flows**

Không có exception flow đặc biệt cho use case này.

---

**Postconditions**

*Success:*

- Actor xem được điểm uy tín và đánh giá của đối tác
- Thông tin giúp actor quyết định hợp tác

*Failure:*

- Không áp dụng (use case chỉ đọc dữ liệu)

---

**Business Rules**

- BR-0706: Trang hồ sơ hiển thị TỐI ĐA 5 đánh giá gần nhất. Có thể xem thêm qua phân trang. Chỉ hiển thị đánh giá APPROVED

---

**Notes / Assumptions**

- Điểm uy tín được tính tự động và cập nhật real-time khi có đánh giá mới (FR-0703)
- Đánh giá FLAGGED hoặc REMOVED không hiển thị
- Có thể được truy cập từ SCR-005, SCR-006, và SCR-026 (cho Authenticated User)
- SCR-026 (public profile) hiển thị tóm tắt uy tín và cung cấp liên kết "Xem chi tiết uy tín" dẫn sang UC-30/SCR-018 (FR-1104, BR-1105). Chỉ AU truy cập SCR-026.
- Liên kết: UC-08, UC-09, UC-29, UC-46 (<<extend>>)
