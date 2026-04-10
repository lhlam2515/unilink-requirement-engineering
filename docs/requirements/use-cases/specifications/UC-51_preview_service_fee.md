# UC-51: Xem trước chi phí dịch vụ

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) xem ước tính phí dịch vụ kết nối dựa trên giá trị tài trợ đang thương thảo, trước khi đồng thuận lock-in. Giúp tránh bất ngờ về chi phí.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Xem ước tính phí |
| Secondary | System | Tính phí dựa trên cấu hình hiện tại |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái IN_PROGRESS
- Actor là một trong hai bên liên quan trong deal

---

**Trigger**
> Actor nhấn "Xem ước tính phí dịch vụ" trong trang thương thảo hoặc trang tạo thỏa thuận nháp.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Nhấn "Xem ước tính phí dịch vụ" |
| 2 | System | Hiển thị form nhập: Hình thức tài trợ (CASH / IN_KIND / COMBINED), Giá trị tiền mặt (nếu CASH/COMBINED) |
| 3 | Authenticated User | Nhập hình thức và giá trị tài trợ ước tính |
| 4 | System | Tính phí ước tính cho cả hai bên dựa trên FeeConfiguration hiện tại |
| 5 | System | Hiển thị kết quả: Phí CLB = [số tiền], Phí DN = [số tiền], Tổng phí nền tảng = [tổng] |
| 6 | System | Hiển thị ghi chú: "Đây là ước tính, phí thực tế sẽ được xác nhận khi lock-in" |
| 7 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-51.a: Tài trợ hoàn toàn hiện vật (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Chọn hình thức IN_KIND |
| 3b | System | Hiển thị phí cố định: CLB = 50.000 VNĐ, DN = 500.000 VNĐ |
| 3c | System | Không yêu cầu nhập giá trị tiền mặt |

---

**Exception Flows**

> EF-51.1: Chưa nhập giá trị tiền mặt cho CASH/COMBINED (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện hình thức CASH/COMBINED nhưng chưa nhập giá trị |
| 3b | System | Hiển thị lỗi "Vui lòng nhập giá trị tài trợ tiền mặt để ước tính phí" |

---

**Postconditions**

*Success:*
- Actor nhìn thấy phí ước tính cho cả hai bên
- Không có thay đổi dữ liệu — chỉ hiển thị thông tin

*Failure:*
- Không hiển thị ước tính do thiếu dữ liệu đầu vào

---

**Business Rules**

- BR-1201: Cơ cấu phí admin-configurable
- BR-1208: Cấu hình phí chỉ admin thay đổi, áp dụng cho deal mới

---

**Notes / Assumptions**

- Đây là tính năng read-only, không ảnh hưởng đến deal hay thanh toán
- Phí ước tính có thể thay đổi nếu admin cập nhật cấu hình trước khi lock-in
- Liên kết: UC-56 (tạo thỏa thuận nháp), UC-50 (thanh toán thực tế)
