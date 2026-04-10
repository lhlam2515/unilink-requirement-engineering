# UC-21: Xác nhận nội dung hợp đồng

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) xác nhận đồng ý với nội dung hợp đồng hiện tại. Khi CẢ HAI bên đều xác nhận, hệ thống chuyển hợp đồng sang trạng thái CONFIRMED và mở khóa tính năng ký chữ ký điện tử.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Mỗi bên xác nhận nội dung |
| Secondary | System | Kiểm tra song phương, chuyển trạng thái |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hợp đồng đang ở trạng thái DRAFTING
- Actor là một trong hai bên liên quan

---

**Trigger**
> Actor nhấn "Xác nhận nội dung hợp đồng" trên trang soạn thảo hợp đồng.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Xem nội dung hợp đồng hiện tại |
| 2 | Authenticated User | Nhấn "Xác nhận nội dung hợp đồng" |
| 3 | System | Ghi nhận xác nhận cho bên hiện tại (organizer_content_confirmed hoặc sponsor_content_confirmed = true) |
| 4 | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã xác nhận nội dung hợp đồng" |
| 5 | System | Kiểm tra cả hai bên đã xác nhận chưa |
| 6 | System | Nếu cả hai đã xác nhận: chuyển hợp đồng sang trạng thái CONFIRMED |
| 7 | System | Mở khóa nút "Ký chữ ký điện tử" |
| 8 | System | Gửi thông báo cho cả hai bên "Nội dung hợp đồng đã được xác nhận, sẵn sàng ký kết" |
| 9 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-21.a: Chỉ một bên xác nhận (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện chỉ một bên đã xác nhận |
| 5b | System | Giữ hợp đồng ở DRAFTING, chờ bên còn lại |
| 5c | System | Use case kết thúc (chưa hoàn thành) |

> AF-21.b: Đối tác chỉnh sửa sau khi actor đã xác nhận (triggered bởi UC-20)

| Step | Actor / System | Action |
|------|----------------|--------|
| – | System | Phát hiện bên đối tác chỉnh sửa nội dung hợp đồng |
| – | System | Reset TẤT CẢ xác nhận về false |
| – | System | Thông báo cho bên đã xác nhận "Nội dung đã thay đổi, cần xác nhận lại" |
| – | System | Quy trình xác nhận bắt đầu lại |

---

**Exception Flows**

Không có exception flow đặc biệt cho use case này.

---

**Postconditions**

*Success (cả hai xác nhận):*
- Hợp đồng chuyển sang trạng thái CONFIRMED
- Nút "Ký chữ ký điện tử" được mở khóa (UC-22)

*Success (một bên xác nhận):*
- Hợp đồng vẫn ở DRAFTING, chờ bên còn lại

*Nếu có chỉnh sửa sau:*
- Tất cả xác nhận bị reset, quy trình bắt đầu lại

---

**Business Rules**

- BR-0504: Khi bất kỳ bên nào chỉnh sửa sau khi có xác nhận: TẤT CẢ xác nhận PHẢI được reset

---

**Notes / Assumptions**

- Xác nhận nội dung là bước bắt buộc trước khi ký — đảm bảo hai bên đồng ý cùng phiên bản
- Nếu cần chỉnh sửa thêm: quay lại UC-20, xác nhận bị reset
- Liên kết: UC-20, UC-22
