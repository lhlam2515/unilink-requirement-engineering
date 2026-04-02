# UC-22: Ký hợp đồng điện tử

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) ký chữ ký điện tử lên hợp đồng đã được xác nhận nội dung (CONFIRMED). Hệ thống hỗ trợ chữ ký vẽ tay hoặc gõ tên. Khi CẢ HAI bên đã ký, hợp đồng chuyển sang trạng thái SIGNED và hệ thống tự động tạo danh sách nghĩa vụ tài trợ.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Mỗi bên ký chữ ký điện tử |
| Secondary | System | Lưu chữ ký, kiểm tra song phương, tạo nghĩa vụ tự động |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hợp đồng đang ở trạng thái CONFIRMED
- Actor là một trong hai bên liên quan
- Actor chưa ký hợp đồng này

---

**Trigger**
> Actor nhấn "Ký hợp đồng" trên trang hợp đồng đã xác nhận.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Nhấn "Ký hợp đồng" trên trang hợp đồng CONFIRMED |
| 2 | System | Hiển thị giao diện ký: chọn hình thức ký (vẽ tay hoặc gõ tên) |
| 3 | Authenticated User | Vẽ chữ ký hoặc gõ tên đầy đủ |
| 4 | Authenticated User | Nhấn "Xác nhận ký" |
| 5 | System | Lưu chữ ký (signature_data), ghi nhận signed = true và signed_at |
| 6 | System | Kiểm tra cả hai bên đã ký chưa |
| 7 | System | Nếu cả hai đã ký: chuyển hợp đồng sang trạng thái SIGNED |
| 8 | System | Tự động tạo danh sách nghĩa vụ tài trợ từ điều khoản hợp đồng (FR-0601 / SF-06) |
| 9 | System | Gửi thông báo cho cả hai bên "Hợp đồng đã được ký kết thành công" |
| 10 | System | Use case kết thúc thành công — hợp đồng đã ký kết |

---

**Alternate Flows**

> AF-22.a: Chỉ một bên ký (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện chỉ một bên đã ký |
| 6b | System | Giữ hợp đồng ở CONFIRMED, chờ bên còn lại |
| 6c | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã ký hợp đồng, chờ bạn ký" |
| 6d | System | Use case kết thúc (chưa hoàn thành) |

---

**Exception Flows**

> EF-22.1: Hợp đồng chưa CONFIRMED (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hợp đồng ở trạng thái DRAFTING |
| 1b | System | Từ chối "Cần xác nhận nội dung bởi cả hai bên trước khi ký" |

> EF-22.2: Actor đã ký trước đó (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện actor đã ký hợp đồng này |
| 1b | System | Thông báo "Bạn đã ký hợp đồng này. Đang chờ đối tác ký." |

---

**Postconditions**

*Success (cả hai ký):*
- Hợp đồng chuyển sang trạng thái SIGNED với chữ ký của hai bên
- Ngày ký (signing_date) được ghi nhận
- Danh sách nghĩa vụ được tạo tự động (UC-25)
- Cả hai bên được thông báo

*Success (một bên ký):*
- Hợp đồng vẫn ở CONFIRMED, chờ bên còn lại

*Failure:*
- Hợp đồng không thay đổi

---

**Business Rules**

- BR-0505: Chữ ký điện tử chỉ thực hiện khi hợp đồng CONFIRMED. Mỗi bên ký MỘT LẦN, sau khi ký không thể rút lại
- BR-0506: Hợp đồng chuyển sang SIGNED khi VÀ CHỈ KHI cả hai bên đều đã ký

---

**Notes / Assumptions**

- Phiên bản đầu sử dụng chữ ký điện tử đơn giản (vẽ/gõ), không phải chữ ký số PKI
- Sau khi SIGNED: có thể xuất PDF (UC-23), yêu cầu hóa đơn VAT (UC-24)
- Nghĩa vụ tài trợ được tạo tự động — theo dõi qua UC-25 đến UC-28
- Liên kết: UC-21, UC-23, UC-24, UC-25, UC-33
