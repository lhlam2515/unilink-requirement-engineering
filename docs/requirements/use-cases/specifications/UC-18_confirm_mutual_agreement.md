# UC-18: Xác nhận đồng thuận ký kết

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) xác nhận sẵn sàng tiến đến ký kết hợp đồng. Hệ thống yêu cầu CẢ HAI bên đều xác nhận trước khi chuyển deal sang trạng thái AGREED và mở khóa tính năng soạn thảo hợp đồng.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Mỗi bên xác nhận đồng thuận |
| Secondary | System | Kiểm tra song phương, chuyển trạng thái, tạo hợp đồng |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái IN_PROGRESS
- Actor là một trong hai bên liên quan trong deal

---

**Trigger**
> Actor nhấn "Xác nhận đồng thuận" trong trang thương thảo.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Nhấn "Xác nhận đồng thuận" trong trang thương thảo |
| 2 | System | Ghi nhận xác nhận cho bên hiện tại (organizer_confirmed hoặc sponsor_confirmed = true) |
| 3 | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã xác nhận sẵn sàng ký kết" |
| 4 | System | Kiểm tra cả hai bên đã xác nhận chưa |
| 5 | System | Nếu cả hai đã xác nhận: chuyển deal sang trạng thái AGREED |
| 6 | System | Mở khóa tính năng "Soạn thảo hợp đồng" |
| 7 | System | Tự động tạo bản nháp hợp đồng từ deal context (FR-0501) |
| 8 | System | Gửi thông báo cho cả hai bên "Thương thảo hoàn tất, sẵn sàng ký kết" |
| 9 | System | Use case kết thúc thành công — deal chuyển sang AGREED |

---

**Alternate Flows**

> AF-18.a: Chỉ một bên xác nhận (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện chỉ có một bên đã xác nhận |
| 4b | System | Giữ deal ở trạng thái IN_PROGRESS, chờ bên còn lại xác nhận |
| 4c | System | Use case kết thúc (chưa hoàn thành — chờ đối tác) |

> AF-18.b: Rút lại xác nhận (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Nhấn "Rút lại xác nhận" (nếu đã xác nhận trước đó và bên còn lại CHƯA xác nhận) |
| 1b | System | Ghi nhận organizer_confirmed hoặc sponsor_confirmed = false |
| 1c | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã rút lại xác nhận" |
| 1d | System | Use case kết thúc |

---

**Exception Flows**

Không có exception flow đặc biệt cho use case này.

---

**Postconditions**

*Success (cả hai xác nhận):*
- Deal chuyển sang trạng thái AGREED
- Bản nháp hợp đồng được tạo tự động (UC-20)
- Tính năng soạn thảo hợp đồng được mở khóa

*Success (một bên xác nhận):*
- Deal vẫn ở IN_PROGRESS, chờ bên còn lại

*Success (rút lại):*
- Xác nhận được reset, deal vẫn ở IN_PROGRESS

---

**Business Rules**

- BR-0405: Deal chuyển từ IN_PROGRESS sang AGREED khi CẢ HAI bên xác nhận. Xác nhận có thể rút lại trước khi bên còn lại xác nhận

---

**Notes / Assumptions**

- Sau khi deal AGREED, không thể hủy bỏ thương thảo qua giao diện
- Bản nháp hợp đồng được tạo tự động với thông tin từ deal context
- Liên kết: UC-14, UC-19, UC-20
