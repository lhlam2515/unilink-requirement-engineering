# Use-Case Specification: UC-52 — Xuất hóa đơn VAT phí dịch vụ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | System | Tự động tạo hóa đơn sau 2/2 payment |
| Secondary | Sponsor | Nhận hóa đơn qua email |

---

### 1. Brief Description

> Hệ thống tự động tạo và gửi hóa đơn VAT điện tử cho Phí quản lý chiến dịch thu từ Doanh nghiệp sau khi thanh toán 2/2 hoàn tất. Hóa đơn CHỈ áp dụng cho phí dịch vụ nền tảng, TUYỆT ĐỐI không xuất cho giá trị gói tài trợ.

---

### 2. Flow of Events

**Trigger**
> PaywallSession chuyển sang COMPLETED (2/2 thanh toán hoàn tất).

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | System | Phát hiện PaywallSession vừa COMPLETED và sponsor fee = PAID |
| 2 | System | Truy xuất thông tin thuế: MST, tên DN, địa chỉ DN |
| 3 | System | Tạo hóa đơn VAT: "Phí quản lý chiến dịch kết nối tài trợ" |
| 4 | System | Tính: Giá trị trước thuế, VAT 10%, Tổng giá trị |
| 5 | System | Gán invoice_number (auto-generated, sequential) |
| 6 | System | Tạo file PDF hóa đơn |
| 7 | System | Gửi hóa đơn PDF qua email cho Sponsor |
| 8 | System | Lưu bản ghi PlatformVATInvoice |
| 9 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-52.a: Phí CLB — không xuất hóa đơn
>
> *Triggered at Step 1 of the Basic Flow when payer_role = ORGANIZER.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Xác nhận payer_role = ORGANIZER |
| 1b | System | KHÔNG tạo hóa đơn VAT (CLB là tổ chức sinh viên) |
| 1c | System | Use case kết thúc — không có hóa đơn |

#### 2.3 Exception Flows

##### EF-52.1: Thiếu thông tin thuế
>
> *Triggered at Step 2 of the Basic Flow when MST hoặc thông tin DN thiếu.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Ghi log lỗi, gửi cảnh báo admin |
| 2b | System | Đánh dấu hóa đơn PENDING_INFO để xử lý thủ công |

##### EF-52.2: Lỗi tạo PDF hoặc gửi email
>
> *Triggered at Step 6-7 of the Basic Flow when lỗi kỹ thuật.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Ghi log lỗi, retry tối đa 3 lần |
| 6b | System | Nếu vẫn thất bại: gửi cảnh báo admin |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-52-01 | Xuất hóa đơn thành công | 2/2 hoàn tất → hóa đơn VAT gửi cho Sponsor |
| SC-52-02 | Phí CLB không xuất | payer_role = ORGANIZER → không tạo hóa đơn (AF-52.a) |

---

### 5. Preconditions

#### 5.1 PaywallSession COMPLETED

- PaywallSession đã đạt COMPLETED (2/2)

#### 5.2 Sponsor đã thanh toán

- ServiceFeeTransaction.status = PAID

#### 5.3 Thông tin thuế đã thu thập

- MST, tên DN, địa chỉ đã được thu thập (UC-50 AF-50.a)

---

### 6. Postconditions

#### 6.1 Success

- PlatformVATInvoice được tạo với đầy đủ thông tin
- Sponsor nhận email với file PDF hóa đơn
- Hóa đơn chỉ ghi "Phí quản lý chiến dịch", KHÔNG ghi giá trị tài trợ

#### 6.2 Failure

- Hóa đơn ở trạng thái PENDING_INFO, chờ admin xử lý

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1207: Hóa đơn VAT CHỈ cho phí dịch vụ, KHÔNG cho giá trị tài trợ
- BR-1204: MST bắt buộc (10 hoặc 13 chữ số) — đã thu thập trong UC-50

---

### 10. Additional Information

**Assumptions:**

- Hóa đơn được tạo tự động — không cần actor kích hoạt thủ công
- Mỗi ServiceFeeTransaction (sponsor) chỉ có tối đa MỘT hóa đơn

**Related Use Cases:**

- UC-50: Thanh toán phí dịch vụ (prerequisite — trigger hóa đơn)
