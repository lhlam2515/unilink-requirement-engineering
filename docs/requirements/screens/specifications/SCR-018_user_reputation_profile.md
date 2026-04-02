# SCR-018: User_ReputationProfile_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-018 |
| **Screen Name** | User_ReputationProfile_Screen |
| **Mục đích** | Authenticated User xem điểm uy tín tổng hợp và danh sách đánh giá gần đây của một đối tác, hỗ trợ quyết định hợp tác; đồng thời có thể báo cáo đánh giá vi phạm |
| **Actor chính** | Authenticated User (bất kỳ) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 5 — Thực hiện nghĩa vụ tài trợ (đánh giá sau hợp đồng) |
| **Use case liên quan** | UC-30, UC-31 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Xem uy tín đối tác — context review/decision |
| Data scope riêng | ✅ Reputation score + reviews list — khác profile chính |
| Action set riêng | ✅ Xem thêm, báo cáo vi phạm |
| Navigation boundary | ✅ Section trên profile công khai, có thể standalone |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Điểm uy tín trung bình (average_reputation_score): X.X/5 ⭐
- Điểm chất lượng hợp tác trung bình (average_quality_score)
- Tổng số đánh giá (total_reviews)
- Danh sách tối đa 5 đánh giá gần nhất (APPROVED), mỗi đánh giá gồm:
  - Điểm uy tín (1-5)
  - Điểm chất lượng (1-5)
  - Nhận xét văn bản (nếu có)
  - Thời gian gửi (submitted_at)
  - Tên sự kiện liên quan

---

## Dữ liệu nhập (Input Fields)

### Modal: Báo cáo đánh giá vi phạm (UC-31)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Lý do báo cáo | Textarea | Bắt buộc | report_reason |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Xem thêm đánh giá** | Hiển thị danh sách phân trang (UC-30 AF-30.a) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Báo cáo đánh giá | Mở modal báo cáo (UC-31 Main) → report PENDING |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0706: Tối đa 5 đánh giá gần nhất. Phân trang cho thêm. Chỉ APPROVED
- BR-0707: FLAGGED cần Admin review 48 giờ. REMOVED không hiển thị

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-005 (Proposal Detail) | Section "Uy tín BTC" |
| SCR-006 (Business Detail) | Section "Uy tín doanh nghiệp" |
| Hồ sơ công khai | Truy cập trực tiếp |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại | SCR-005 hoặc SCR-006 (tùy context) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải uy tín |
| No Reviews State | Chưa có đánh giá (UC-30 AF-30.b) |
| Report Modal | Form báo cáo vi phạm |
| Report Success Toast | Báo cáo gửi thành công (UC-31 Main-8) |
| Already Reported | Đã báo cáo trước đó (UC-31 EF-31.1) |

## UI Components liên quan

- Star rating display — điểm trung bình
- Review cards — danh sách đánh giá
- Pagination — xem thêm
- Report button — trên mỗi đánh giá
- Modal dialog — form báo cáo vi phạm
- Toast notification
- Empty state — chưa có đánh giá

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-30 | Main-1 | Truy cập trang uy tín | Screen entry | Page load |
| UC-30 | Main-2~3 | Hiển thị điểm trung bình | Read-only data | Star rating |
| UC-30 | Main-4 | Hiển thị 5 đánh giá gần nhất | Read-only data | Review cards |
| UC-30 | Main-5 | Mỗi đánh giá: điểm, nhận xét, thời gian, sự kiện | Read-only data | Card content |
| UC-30 | AF-30.a | Xem thêm | Component | Pagination |
| UC-30 | AF-30.b | Chưa có đánh giá | UI State | Empty state |
| UC-31 | Main-1 | Nhấn "Báo cáo" trên đánh giá | Action | Report button |
| UC-31 | Main-2~4 | Nhập lý do + gửi | Input + Action | Modal form |
| UC-31 | Main-5~8 | Tạo report PENDING | Action | System processing |
| UC-31 | EF-31.1 | Đã báo cáo trước đó | UI State | Error message |
| UC-31 | AF-31.a~b | Admin xử lý (background) | — | Không trên screen này |
