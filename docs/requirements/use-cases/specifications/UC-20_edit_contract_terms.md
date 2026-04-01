# UC-20: Chỉnh sửa điều khoản hợp đồng

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) chỉnh sửa các điều khoản trong bản nháp hợp đồng tài trợ được tạo tự động từ deal đã đồng thuận. Bao gồm thông tin hai bên, thời gian hiệu lực, hình thức và giá trị tài trợ, quyền lợi, cam kết, và trách nhiệm. Hệ thống lưu lịch sử thay đổi.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người chỉnh sửa nội dung hợp đồng |
| Secondary | System | Tạo bản nháp tự động, xác thực, lưu lịch sử chỉnh sửa |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đã ở trạng thái AGREED
- Hợp đồng đang ở trạng thái DRAFTING (được tạo tự động khi deal AGREED)
- Actor là một trong hai bên liên quan

---

**Trigger**
> Actor mở trang soạn thảo hợp đồng và chỉnh sửa nội dung các điều khoản.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Mở trang soạn thảo hợp đồng |
| 2 | System | Hiển thị hợp đồng đã điền sẵn thông tin từ deal context: thông tin BTC, thông tin doanh nghiệp, sự kiện, gói tài trợ |
| 3 | Authenticated User | Chỉnh sửa các mục: thông tin hai bên, ngày ký kết, thời gian hiệu lực, thời hạn giao dịch, hình thức tài trợ, giá trị tài trợ, quyền lợi, cam kết và trách nhiệm |
| 4 | Authenticated User | Nhấn "Lưu thay đổi" |
| 5 | System | Xác thực dữ liệu đã nhập |
| 6 | System | Lưu nội dung đã cập nhật |
| 7 | System | Ghi log chỉnh sửa: người chỉnh sửa, thời gian, trường thay đổi, giá trị cũ → giá trị mới |
| 8 | System | Tăng version_number của hợp đồng |
| 9 | System | Nếu đã có xác nhận nội dung trước đó (bất kỳ bên nào): reset TẤT CẢ xác nhận |
| 10 | System | Thông báo cho đối tác "Nội dung hợp đồng đã được cập nhật bởi [bên chỉnh sửa]" |
| 11 | System | Use case kết thúc thành công — hợp đồng đã cập nhật |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-20.1: Hợp đồng đã ký (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hợp đồng ở trạng thái SIGNED |
| 1b | System | Từ chối chỉnh sửa với thông báo "Hợp đồng đã ký không thể chỉnh sửa" |

> EF-20.2: Hợp đồng đã ở trạng thái CONFIRMED (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hợp đồng ở trạng thái CONFIRMED |
| 1b | System | Cảnh báo "Nội dung đã được xác nhận bởi cả hai bên. Chỉnh sửa sẽ reset toàn bộ xác nhận." |
| 1c | Authenticated User | Xác nhận tiếp tục chỉnh sửa hoặc hủy |
| 1d | System | Nếu tiếp tục: chuyển hợp đồng về DRAFTING, reset xác nhận. Tiếp tục tại Step 2 |

---

**Postconditions**

*Success:*
- Hợp đồng được cập nhật với nội dung mới
- Lịch sử chỉnh sửa được ghi lại (ai, khi nào, thay đổi gì)
- Version number tăng
- Nếu đã có xác nhận: tất cả xác nhận bị reset

*Failure:*
- Hợp đồng không thay đổi
- Actor được thông báo lỗi

---

**Business Rules**

- BR-0501: Hợp đồng chỉ tạo từ deal ở trạng thái AGREED. Mỗi deal chỉ có DUY NHẤT một hợp đồng
- BR-0502: Hợp đồng chỉ chỉnh sửa khi ở trạng thái DRAFTING. Trạng thái SIGNED không cho phép chỉnh sửa
- BR-0503: Hệ thống PHẢI lưu lịch sử mọi thay đổi: người chỉnh sửa, thời gian, trường, giá trị cũ/mới
- BR-0504: Khi chỉnh sửa sau khi có xác nhận: TẤT CẢ xác nhận bị reset

---

**Notes / Assumptions**

- Bản nháp hợp đồng được tạo tự động từ template khi deal AGREED (FR-0501)
- Cả hai bên đều có quyền chỉnh sửa nội dung
- Sau khi chỉnh sửa xong, hai bên xác nhận nội dung thông qua UC-21
- Liên kết: UC-18, UC-21, UC-22
