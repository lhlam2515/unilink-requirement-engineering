# UC-33: Hủy đồng thuận ký kết hợp đồng

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) hủy đồng thuận ký kết khi hợp đồng đang ở trạng thái DRAFTING hoặc CONFIRMED và chưa có chữ ký điện tử từ bất kỳ bên nào. Hệ thống reset cờ xác nhận nội dung, chuyển contract về DRAFTING, và đưa deal liên kết về IN_PROGRESS để hai bên quay lại thương thảo.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|k
| Primary | Authenticated User (Organizer hoặc Sponsor) | Bên muốn hủy đồng thuận ký kết |
| Secondary | System | Kiểm tra điều kiện, reset trạng thái, gửi thông báo |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hợp đồng đang ở trạng thái DRAFTING hoặc CONFIRMED
- Chưa có chữ ký điện tử từ bất kỳ bên nào (organizer_signed = false VÀ sponsor_signed = false)
- Actor là một trong hai bên liên quan trong deal/contract

---

**Trigger**
> Actor nhấn "Hủy đồng thuận ký kết" trong trang hợp đồng.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Nhấn "Hủy đồng thuận ký kết" trong trang hợp đồng |
| 2 | System | Kiểm tra hợp đồng ở trạng thái DRAFTING hoặc CONFIRMED |
| 3 | System | Kiểm tra chưa có chữ ký điện tử nào (BR-0509) |
| 4 | System | Hiển thị form nhập lý do hủy (bắt buộc, tối thiểu 10 ký tự) |
| 5 | Authenticated User | Nhập lý do hủy đồng thuận |
| 6 | Authenticated User | Nhấn "Xác nhận hủy" |
| 7 | System | Xác thực lý do (≥ 10 ký tự) |
| 8 | System | Chuyển hợp đồng về trạng thái DRAFTING |
| 9 | System | Reset cờ xác nhận: organizer_content_confirmed = false, sponsor_content_confirmed = false |
| 10 | System | Chuyển deal liên kết về trạng thái IN_PROGRESS |
| 11 | System | Ghi nhận cancellation_log_entry: người hủy, thời gian, lý do |
| 12 | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã hủy đồng thuận ký kết" kèm lý do |
| 13 | System | Use case kết thúc thành công — hai bên quay lại thương thảo |

---

**Alternate Flows**

> AF-33.a: Actor hủy thao tác (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Authenticated User | Nhấn "Hủy" hoặc quay lại trang trước |
| 4b | System | Giữ nguyên trạng thái hợp đồng và deal. Use case kết thúc |

---

**Exception Flows**

> EF-33.1: Hợp đồng đã có chữ ký điện tử (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện organizer_signed = true hoặc sponsor_signed = true |
| 3b | System | Từ chối với thông báo "Không thể hủy đồng thuận sau khi đã bắt đầu ký hợp đồng" |

> EF-33.2: Lý do hủy quá ngắn (triggered at Step 7)

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | System | Phát hiện lý do dưới 10 ký tự |
| 7b | System | Từ chối với thông báo "Lý do hủy phải có ít nhất 10 ký tự" |
| 7c | Authenticated User | Nhập lý do đầy đủ hơn và thử xác nhận lại |

---

**Postconditions**

*Success:*
- Hợp đồng chuyển về trạng thái DRAFTING
- Cờ xác nhận nội dung bị reset (cả hai bên phải xác nhận lại)
- Deal liên kết chuyển về IN_PROGRESS (hai bên có thể tiếp tục thương thảo)
- Cancellation log được ghi nhận
- Đối tác được thông báo kèm lý do

*Failure:*
- Hợp đồng và deal không thay đổi
- Actor được thông báo lý do không thể hủy

---

**Business Rules**

- BR-0509: Chỉ cho phép hủy đồng thuận ký kết khi contract ở trạng thái DRAFTING hoặc CONFIRMED và chưa có chữ ký điện tử nào. Khi hủy, hệ thống PHẢI reset cờ xác nhận nội dung và đưa deal liên kết về IN_PROGRESS. Lý do hủy BẮT BUỘC (tối thiểu 10 ký tự) và PHẢI được ghi audit log

---

**Notes / Assumptions**

- Sau khi hủy đồng thuận, hai bên có thể tiếp tục thương thảo (UC-14), đặt lịch họp (UC-15), hoặc xác nhận đồng thuận lại (UC-18)
- Nếu đã có chữ ký, không thể hủy đồng thuận — cần quy trình hủy hợp đồng riêng (chưa thuộc phạm vi hiện tại)
- Liên kết: UC-18, UC-20, UC-21, UC-22
