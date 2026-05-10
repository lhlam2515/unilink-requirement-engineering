# Use-Case Specification: UC-24 — Yêu cầu hóa đơn VAT phí dịch vụ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Sponsor (Doanh nghiệp) | Người yêu cầu hóa đơn |
| Secondary | System | Tạo hóa đơn, tính toán thuế, gán mã hóa đơn |

---

### 1. Brief Description

> Sponsor yêu cầu phát hành hóa đơn VAT (hóa đơn đỏ) cho khoản phí dịch vụ kết nối đã thanh toán qua Paywall. Hóa đơn CHỈ áp dụng cho phí dịch vụ nền tảng, TUYỆT ĐỐI KHÔNG xuất cho giá trị gói tài trợ. Hệ thống tạo hóa đơn với thông tin doanh nghiệp, giá trị phí dịch vụ, thuế suất VAT, và cho phép tải về dạng PDF.

---

### 2. Flow of Events

**Trigger**
> Sponsor nhấn "Yêu cầu hóa đơn VAT" trên trang hợp đồng đã ký.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Sponsor | Nhấn "Yêu cầu hóa đơn VAT" trên trang hợp đồng |
| 2 | System | Truy xuất thông tin thuế đã nhập từ Paywall (UC-50 AF-50.a): MST, tên DN, địa chỉ DN |
| 3 | System | Hiển thị form xác nhận thông tin thuế với dữ liệu đã có (cho phép chỉnh sửa nếu cần) |
| 4 | Sponsor | Xác nhận hoặc chỉnh sửa thông tin thuế |
| 5 | Sponsor | Nhấn "Phát hành hóa đơn" |
| 6 | System | Xác thực: MST đúng định dạng (10 hoặc 13 chữ số), tất cả trường bắt buộc đầy đủ |
| 7 | System | Tính toán: giá trị phí dịch vụ trước thuế, thuế suất VAT (10%), tiền thuế, tổng giá trị |
| 8 | System | Gán invoice_number tự động (sequential) |
| 9 | System | Tạo hóa đơn VAT với nội dung: "Phí dịch vụ kết nối tài trợ" |
| 10 | System | Tạo file PDF hóa đơn và cho phép tải về |
| 11 | System | Lưu bản ghi PlatformVATInvoice liên kết với ServiceFeeTransaction |
| 12 | System | Use case kết thúc thành công — hóa đơn đã được phát hành |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-24.1: Sponsor chưa thanh toán phí dịch vụ
>
> *Triggered at Step 1 of the Basic Flow when ServiceFeeTransaction chưa PAID.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện Sponsor chưa thanh toán phí dịch vụ |
| 1b | System | Từ chối "Hóa đơn VAT chỉ khả dụng sau khi thanh toán phí dịch vụ hoàn tất" |

##### EF-24.2: Mã số thuế không hợp lệ
>
> *Triggered at Step 6 of the Basic Flow when MST sai định dạng.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện MST không đúng 10 hoặc 13 chữ số |
| 6b | System | Hiển thị "Mã số thuế phải có 10 hoặc 13 chữ số" |
| 6c | Sponsor | Nhập lại MST |

##### EF-24.3: Đã có hóa đơn cho giao dịch phí này
>
> *Triggered at Step 1 of the Basic Flow when đã phát hành hóa đơn.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện đã phát hành hóa đơn VAT cho ServiceFeeTransaction này |
| 1b | System | Hiển thị "Đã có hóa đơn VAT. Bạn có thể tải lại hóa đơn hiện tại." kèm nút tải PDF |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-24-01 | Phát hành hóa đơn thành công | Sponsor xác nhận thông tin thuế; hóa đơn VAT phí dịch vụ được tạo |
| SC-24-02 | Đã có hóa đơn | Sponsor tải lại hóa đơn đã phát hành (EF-24.3) |

---

### 5. Preconditions

#### 5.1 Sponsor đã xác thực

- Sponsor đã đăng nhập vào hệ thống

#### 5.2 Hợp đồng đã SIGNED

- Hợp đồng đang ở trạng thái SIGNED

#### 5.3 Phí dịch vụ đã thanh toán

- ServiceFeeTransaction của Sponsor có status = PAID

#### 5.4 Chưa có hóa đơn

- Chưa có hóa đơn VAT cho ServiceFeeTransaction này

---

### 6. Postconditions

#### 6.1 Success

- Hóa đơn VAT phí dịch vụ được tạo với invoice_number duy nhất
- Hóa đơn chỉ ghi "Phí dịch vụ kết nối tài trợ", KHÔNG ghi giá trị tài trợ
- Sponsor có thể tải hóa đơn dạng PDF
- PlatformVATInvoice liên kết với ServiceFeeTransaction

#### 6.2 Failure

- Hóa đơn không được tạo
- Sponsor được thông báo lỗi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1207: Hóa đơn VAT CHỈ cho phí dịch vụ nền tảng, TUYỆT ĐỐI KHÔNG cho giá trị gói tài trợ
- BR-1204: MST bắt buộc (10 hoặc 13 chữ số)
- Mỗi ServiceFeeTransaction (sponsor) chỉ có tối đa MỘT hóa đơn

---

### 10. Additional Information

**Assumptions:**

- Thuế suất VAT mặc định: 10%
- Thông tin thuế có thể đã được thu thập tại bước thanh toán Paywall (UC-50 AF-50.a) — Sponsor chỉ cần xác nhận lại
- Hóa đơn phí dịch vụ là tách biệt hoàn toàn với hợp đồng tài trợ
- UC-52 xử lý hóa đơn tự động khi 2/2 payment hoàn tất; UC-24 là luồng yêu cầu thủ công bổ sung khi Sponsor cần hóa đơn sau đó

**Related Use Cases:**

- UC-22: Ký hợp đồng điện tử (`<<extend>>` base — UC-24 mở rộng UC-22 sau khi SIGNED)
- UC-50: Thanh toán phí dịch vụ (prerequisite — phí phải đã thanh toán)
- UC-52: Xuất hóa đơn VAT tự động (parallel — system tự động tạo khi 2/2)
