# SCR-019: User_PartnerReview_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-019 |
| **Screen Name** | User_PartnerReview_Screen |
| **Mục đích** | Authenticated User gửi đánh giá về đối tác sau khi hợp đồng tài trợ kết thúc, bao gồm điểm uy tín, điểm chất lượng hợp tác, và nhận xét văn bản |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 5 — Thực hiện nghĩa vụ tài trợ (đánh giá sau hợp đồng) |
| **Use case liên quan** | UC-29 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Gửi đánh giá — form multi-criteria, mục tiêu rõ ràng |
| Data scope riêng | ✅ Rating scores + review text — data nhập mới |
| Action set riêng | ✅ Chấm điểm, nhập nhận xét, submit |
| Navigation boundary | ✅ Context riêng từ contract view |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Thông tin đối tác (tên, vai trò)
- Thông tin hợp đồng (tên sự kiện, thời gian HĐ, giá trị tài trợ)
- Tóm tắt nghĩa vụ đã hoàn thành (reference)

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Điểm uy tín | Star rating (1-5) | Bắt buộc, số nguyên 1-5 | reputation_score |
| Điểm chất lượng hợp tác | Star rating (1-5) | Bắt buộc, số nguyên 1-5 | quality_score |
| Nhận xét văn bản | Textarea | Tùy chọn, ≤ 1000 ký tự | review_comment |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Gửi đánh giá** | Lưu + kiểm duyệt + cập nhật điểm tổng hợp (UC-29 Main) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Quay lại | SCR-014 (Contract View) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0701: Đánh giá chỉ khi HĐ kết thúc (validity_end < hôm nay HOẶC tất cả nghĩa vụ CONFIRMED)
- BR-0702: Mỗi bên chỉ 1 đánh giá/hợp đồng
- BR-0703: Điểm 1-5 (số nguyên), nhận xét ≤ 1000 ký tự
- BR-0705: Điểm tổng hợp = trung bình APPROVED. Cập nhật real-time
- BR-0707: Kiểm duyệt tự động. FLAGGED → Admin review 48 giờ

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-014 (Contract View) | Link "Đánh giá đối tác" |
| In-app notification | Thông báo nhắc đánh giá |
| Email | Link nhắc đánh giá |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Gửi thành công | SCR-014 (Contract View) hoặc SCR-018 (Reputation) |
| Quay lại | SCR-014 |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải |
| Contract Not Ended Error | HĐ chưa kết thúc (UC-29 EF-29.1) |
| Already Reviewed Error | Đã đánh giá rồi (UC-29 EF-29.2) |
| Invalid Score Error | Điểm ngoài 1-5 (UC-29 EF-29.3) |
| Submit Success Toast | Đánh giá gửi thành công |
| Flagged Info | Nội dung bị đánh dấu → chờ Admin (UC-29 AF-29.a) |

## UI Components liên quan

- Partner info card — thông tin đối tác + hợp đồng
- Star rating inputs — 2 tiêu chí: uy tín + chất lượng
- Textarea — nhận xét (với character counter)
- Submit button — "Gửi đánh giá"
- Toast notification

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-29 | Main-1 | Nhấn "Đánh giá đối tác" | Screen entry | Page load |
| UC-29 | Main-2 | Hiển thị form đánh giá | Screen content | Rating + Textarea |
| UC-29 | Main-3 | Chấm điểm uy tín | Input | Star rating 1 |
| UC-29 | Main-4 | Chấm điểm chất lượng | Input | Star rating 2 |
| UC-29 | Main-5 | Nhập nhận xét | Input | Textarea |
| UC-29 | Main-6 | Nhấn "Gửi" | Action | Submit button |
| UC-29 | Main-7 | Validate | Action | System check |
| UC-29 | Main-8~12 | Lưu + kiểm duyệt + tính điểm + thông báo | Action | System processing |
| UC-29 | AF-29.a | Nội dung FLAGGED | UI State | Info: chờ Admin |
| UC-29 | EF-29.1 | HĐ chưa kết thúc | UI State | Error block |
| UC-29 | EF-29.2 | Đã đánh giá | UI State | Error block |
| UC-29 | EF-29.3 | Điểm ngoài 1-5 | UI State | Validation error |
