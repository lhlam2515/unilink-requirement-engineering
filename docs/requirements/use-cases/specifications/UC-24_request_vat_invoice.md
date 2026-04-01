# UC-24: Yêu cầu hóa đơn VAT

**Brief Description**
> Sponsor yêu cầu phát hành hóa đơn VAT (hóa đơn đỏ) cho giao dịch tài trợ tiền mặt từ hợp đồng đã ký. Hệ thống tạo hóa đơn với thông tin doanh nghiệp, giá trị, thuế suất VAT, và cho phép xuất dạng PDF.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Sponsor (Doanh nghiệp) | Người yêu cầu hóa đơn |
| Secondary | System | Tạo hóa đơn, tính toán thuế, gán mã hóa đơn |

---

**Preconditions**

- Sponsor đã đăng nhập vào hệ thống
- Hợp đồng đang ở trạng thái SIGNED
- Hình thức tài trợ bao gồm CASH (tiền mặt)
- Chưa có hóa đơn VAT cho hợp đồng này

---

**Trigger**
> Sponsor nhấn "Yêu cầu hóa đơn VAT" trên trang hợp đồng đã ký.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Sponsor | Nhấn "Yêu cầu hóa đơn VAT" trên trang hợp đồng |
| 2 | System | Hiển thị form nhập thông tin: tên doanh nghiệp, mã số thuế, địa chỉ doanh nghiệp, mô tả dịch vụ |
| 3 | Sponsor | Nhập mã số thuế và xác nhận thông tin doanh nghiệp |
| 4 | Sponsor | Nhập mô tả nội dung dịch vụ |
| 5 | Sponsor | Nhấn "Phát hành hóa đơn" |
| 6 | System | Xác thực: mã số thuế đúng định dạng (10 hoặc 13 chữ số), tất cả trường bắt buộc đầy đủ |
| 7 | System | Tính toán: giá trị trước thuế, thuế suất VAT (10%), tiền thuế, tổng giá trị |
| 8 | System | Gán số hóa đơn (invoice_number) tự động, tuần tự |
| 9 | System | Tạo hóa đơn VAT và ghi nhận issued_at |
| 10 | System | Cho phép xuất hóa đơn dạng PDF |
| 11 | System | Use case kết thúc thành công — hóa đơn đã được phát hành |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-24.1: Hình thức tài trợ không bao gồm tiền mặt (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hình thức tài trợ chỉ có IN_KIND |
| 1b | System | Từ chối "Hóa đơn VAT chỉ áp dụng cho giao dịch tiền mặt" |

> EF-24.2: Mã số thuế không hợp lệ (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện mã số thuế không đúng 10 hoặc 13 chữ số |
| 6b | System | Từ chối "Mã số thuế phải có 10 hoặc 13 chữ số" |
| 6c | Sponsor | Nhập lại mã số thuế |

> EF-24.3: Đã có hóa đơn cho hợp đồng (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện đã phát hành hóa đơn VAT cho hợp đồng này |
| 1b | System | Từ chối "Hợp đồng đã có hóa đơn VAT. Bạn có thể tải lại hóa đơn hiện tại." |

---

**Postconditions**

*Success:*
- Hóa đơn VAT được tạo với số hóa đơn duy nhất
- Sponsor có thể xuất hóa đơn dạng PDF
- Hóa đơn liên kết với hợp đồng

*Failure:*
- Hóa đơn không được tạo
- Sponsor được thông báo lỗi

---

**Business Rules**

- BR-0508: Hóa đơn VAT chỉ cho hợp đồng SIGNED có hình thức CASH. Mã số thuế BẮT BUỘC (10 hoặc 13 chữ số). Mỗi hợp đồng tối đa MỘT hóa đơn

---

**Notes / Assumptions**

- Thuế suất VAT mặc định: 10%
- Hóa đơn VAT theo quy định pháp luật Việt Nam (giả định)
- Liên kết: UC-22
