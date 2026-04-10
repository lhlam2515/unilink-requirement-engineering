# UC-52: Xuất hóa đơn VAT phí dịch vụ

**Brief Description**
> Hệ thống tự động tạo và gửi hóa đơn VAT điện tử cho Phí quản lý chiến dịch thu từ Doanh nghiệp sau khi thanh toán 2/2 hoàn tất. Hóa đơn CHỈ áp dụng cho phí dịch vụ nền tảng, TUYỆT ĐỐI không xuất cho giá trị gói tài trợ.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | System | Tự động tạo hóa đơn sau 2/2 payment |
| Secondary | Sponsor | Nhận hóa đơn qua email |

---

**Preconditions**

- PaywallSession đã đạt COMPLETED (2/2)
- Sponsor đã thanh toán phí dịch vụ (ServiceFeeTransaction.status = PAID)
- Thông tin thuế (MST, tên DN, địa chỉ) đã được thu thập (UC-50 AF-50.a)

---

**Trigger**
> PaywallSession chuyển sang COMPLETED (2/2 thanh toán hoàn tất).

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | System | Phát hiện PaywallSession vừa COMPLETED và sponsor fee = PAID |
| 2 | System | Truy xuất thông tin thuế: MST, tên DN, địa chỉ DN từ ServiceFeeTransaction |
| 3 | System | Tạo hóa đơn VAT với nội dung: "Phí quản lý chiến dịch kết nối tài trợ" |
| 4 | System | Tính: Giá trị trước thuế, VAT 10%, Tổng giá trị |
| 5 | System | Gán invoice_number (auto-generated, sequential) |
| 6 | System | Tạo file PDF hóa đơn |
| 7 | System | Gửi hóa đơn PDF qua email cho Sponsor |
| 8 | System | Lưu bản ghi PlatformVATInvoice |
| 9 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-52.a: Phí CLB — không xuất hóa đơn (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Xác nhận ServiceFeeTransaction là phí CLB (payer_role = ORGANIZER) |
| 1b | System | KHÔNG tạo hóa đơn VAT (CLB là tổ chức sinh viên, không cần hóa đơn) |
| 1c | System | Use case kết thúc — không có hóa đơn |

---

**Exception Flows**

> EF-52.1: Thiếu thông tin thuế (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện MST hoặc thông tin DN bị thiếu/trống |
| 2b | System | Ghi log lỗi, gửi cảnh báo admin |
| 2c | System | Đánh dấu hóa đơn PENDING_INFO để xử lý thủ công |

> EF-52.2: Lỗi tạo PDF hoặc gửi email (triggered at Step 6-7)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện lỗi kỹ thuật khi tạo PDF hoặc gửi email |
| 6b | System | Ghi log lỗi, retry tối đa 3 lần |
| 6c | System | Nếu vẫn thất bại: gửi cảnh báo admin để xử lý thủ công |

---

**Postconditions**

*Success:*
- PlatformVATInvoice được tạo với đầy đủ thông tin
- Sponsor nhận được email với file PDF hóa đơn
- Hóa đơn chỉ ghi "Phí quản lý chiến dịch", KHÔNG ghi giá trị tài trợ

*Failure:*
- Hóa đơn ở trạng thái PENDING_INFO, chờ admin xử lý
- Sponsor chưa nhận hóa đơn

---

**Business Rules**

- BR-1207: Hóa đơn VAT CHỈ cho phí dịch vụ, KHÔNG cho giá trị tài trợ
- BR-1204: MST bắt buộc (10 hoặc 13 chữ số) — đã thu thập trong UC-50

---

**Notes / Assumptions**

- Hóa đơn được tạo tự động — không cần actor kích hoạt thủ công
- Mỗi ServiceFeeTransaction (sponsor) chỉ có tối đa MỘT hóa đơn
- Liên kết: UC-50 (thanh toán → trigger hóa đơn)
