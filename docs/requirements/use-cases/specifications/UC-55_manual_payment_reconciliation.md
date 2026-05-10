# Use-Case Specification: UC-55 — Đối soát thanh toán thủ công

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Thực hiện đối soát, xác nhận thanh toán thủ công |
| Secondary | System | Cập nhật trạng thái, ghi audit log |

---

### 1. Brief Description

> Admin thực hiện đối soát thủ công khi webhook thanh toán thất bại, không khớp, hoặc bị trùng lặp. Admin xác nhận thanh toán đã nhận được dựa trên bằng chứng ngân hàng và cập nhật trạng thái ServiceFeeTransaction.

---

### 2. Flow of Events

**Trigger**
> Admin nhận cảnh báo hệ thống về giao dịch cần đối soát, hoặc truy cập trang "Đối soát thanh toán".

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Admin | Truy cập trang "Đối soát thanh toán" |
| 2 | System | Hiển thị danh sách: MISMATCH, UNMATCHED, hoặc chưa nhận webhook > 4 giờ |
| 3 | Admin | Chọn một giao dịch để đối soát |
| 4 | System | Hiển thị chi tiết: transaction_reference, số tiền kỳ vọng, webhook data, deal info |
| 5 | Admin | Xác nhận tiền đã vào tài khoản (kiểm tra bên ngoài hệ thống) |
| 6 | Admin | Nhập bank_reference thực tế và nhấn "Xác nhận thanh toán" |
| 7 | System | Cập nhật ServiceFeeTransaction.status = PAID |
| 8 | System | Ghi audit log: "Manual reconciliation by [admin] at [time], bank_ref = [ref]" |
| 9 | System | Kích hoạt chuỗi xử lý thanh toán bình thường (PaywallSession, thông báo, v.v.) |
| 10 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-55.a: Xác nhận giao dịch trùng lặp
>
> *Triggered at Step 3 of the Basic Flow when processing_result = DUPLICATE.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Admin | Chọn giao dịch DUPLICATE |
| 3b | System | Hiển thị webhook gốc và webhook trùng |
| 3c | Admin | Xác nhận trùng lặp thực sự, đánh dấu "Đã xác nhận trùng" |
| 3d | System | Ghi nhận, không hành động thêm |

##### AF-55.b: Từ chối đối soát
>
> *Triggered at Step 5 of the Basic Flow when tiền KHÔNG vào tài khoản.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Admin | Chọn "Từ chối đối soát", nhập lý do |
| 5b | System | Giữ nguyên trạng thái, ghi log "Rejected reconciliation" |
| 5c | System | Tiếp tục áp dụng quy trình 48h timeout bình thường |

#### 2.3 Exception Flows

##### EF-55.1: Giao dịch đã được xử lý
>
> *Triggered at Step 3 of the Basic Flow when status đã = PAID.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Hiển thị "Giao dịch đã được xử lý tự động" |
| 3b | Admin | Quay lại danh sách |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-55-01 | Đối soát thành công | Admin xác nhận thanh toán; ServiceFeeTransaction → PAID |
| SC-55-02 | Từ chối đối soát | Tiền không vào; giữ nguyên trạng thái (AF-55.b) |
| SC-55-03 | Giao dịch trùng | Admin xác nhận trùng lặp (AF-55.a) |

---

### 5. Preconditions

#### 5.1 Admin đã xác thực

- Admin đã đăng nhập với quyền quản trị

#### 5.2 Có giao dịch cần đối soát

- Có ít nhất một giao dịch MISMATCH, UNMATCHED, hoặc không nhận webhook > 4 giờ

---

### 6. Postconditions

#### 6.1 Success (xác nhận thanh toán)

- ServiceFeeTransaction chuyển sang PAID
- Audit log ghi nhận đối soát thủ công
- PaywallSession cập nhật payment_count
- Nếu 2/2: kích hoạt unlock + hợp đồng tự động

#### 6.2 Failure

- Giao dịch vẫn ở trạng thái chờ đối soát

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1405: Xử lý webhook idempotent, manual reconciliation yêu cầu audit log bắt buộc
- BR-1205: Đối soát dựa trên transaction_reference và exact match

---

### 10. Additional Information

**Assumptions:**

- Admin xác nhận dựa trên kiểm tra NGOÀI hệ thống (internet banking, sao kê)
- Hệ thống ghi cảnh báo khi không nhận webhook > 4 giờ sau khi tạo QR

**Related Use Cases:**

- UC-50: Thanh toán phí dịch vụ (sequential — thanh toán bình thường)
- UC-54: Xem báo cáo doanh thu (sequential — ảnh hưởng doanh thu)
