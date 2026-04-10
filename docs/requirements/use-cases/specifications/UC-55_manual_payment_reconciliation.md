# UC-55: Đối soát thanh toán thủ công

**Brief Description**
> Admin thực hiện đối soát thủ công khi webhook thanh toán thất bại, không khớp, hoặc bị trùng lặp. Admin xác nhận thanh toán đã nhận được dựa trên bằng chứng ngân hàng và cập nhật trạng thái ServiceFeeTransaction.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Thực hiện đối soát, xác nhận thanh toán thủ công |
| Secondary | System | Cập nhật trạng thái, ghi audit log |

---

**Preconditions**

- Admin đã đăng nhập với quyền quản trị
- Có ít nhất một giao dịch cần đối soát (MISMATCH, UNMATCHED, hoặc không nhận webhook > 4 giờ)

---

**Trigger**
> Admin nhận cảnh báo hệ thống về giao dịch cần đối soát, hoặc truy cập trang "Đối soát thanh toán".

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Admin | Truy cập trang "Đối soát thanh toán" |
| 2 | System | Hiển thị danh sách giao dịch cần đối soát: MISMATCH, UNMATCHED, hoặc chưa nhận webhook > 4 giờ |
| 3 | Admin | Chọn một giao dịch để đối soát |
| 4 | System | Hiển thị chi tiết: transaction_reference, số tiền kỳ vọng, webhook data (nếu có), deal info |
| 5 | Admin | Xác nhận tiền đã vào tài khoản nền tảng (kiểm tra bên ngoài hệ thống) |
| 6 | Admin | Nhập bank_reference thực tế và nhấn "Xác nhận thanh toán" |
| 7 | System | Cập nhật ServiceFeeTransaction.status = PAID |
| 8 | System | Ghi audit log: "Manual reconciliation by [admin] at [time], bank_ref = [ref]" |
| 9 | System | Kích hoạt chuỗi xử lý thanh toán bình thường (cập nhật PaywallSession, thông báo, v.v.) |
| 10 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-55.a: Xác nhận giao dịch trùng lặp (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Admin | Chọn giao dịch có processing_result = DUPLICATE |
| 3b | System | Hiển thị webhook gốc đã xử lý và webhook trùng lặp |
| 3c | Admin | Xác nhận đây là trùng lặp thực sự, đánh dấu "Đã xác nhận trùng" |
| 3d | System | Ghi nhận, không có hành động thêm |

> AF-55.b: Từ chối đối soát (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Admin | Xác nhận tiền KHÔNG vào tài khoản, chọn "Từ chối đối soát" |
| 5b | Admin | Nhập lý do từ chối |
| 5c | System | Giữ nguyên trạng thái giao dịch, ghi log "Rejected reconciliation" |
| 5d | System | Tiếp tục áp dụng quy trình 48h timeout bình thường |

---

**Exception Flows**

> EF-55.1: Giao dịch đã được xử lý (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện giao dịch đã có status = PAID (webhook đến sau khi admin mở trang) |
| 3b | System | Hiển thị "Giao dịch đã được xử lý tự động" |
| 3c | Admin | Quay lại danh sách |

---

**Postconditions**

*Success (xác nhận thanh toán):*
- ServiceFeeTransaction chuyển sang PAID
- Audit log ghi nhận đối soát thủ công
- PaywallSession được cập nhật payment_count
- Nếu 2/2: kích hoạt unlock + hợp đồng tự động

*Failure:*
- Giao dịch vẫn ở trạng thái chờ đối soát

---

**Business Rules**

- BR-1405: Xử lý webhook idempotent, manual reconciliation yêu cầu audit log bắt buộc
- BR-1205: Đối soát dựa trên transaction_reference và exact match

---

**Notes / Assumptions**

- Admin xác nhận thanh toán dựa trên kiểm tra NGOÀI hệ thống (internet banking, sao kê ngân hàng)
- Hệ thống ghi cảnh báo khi không nhận webhook > 4 giờ sau khi tạo QR
- Liên kết: UC-50 (thanh toán bình thường), UC-54 (báo cáo doanh thu)
