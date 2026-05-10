# Use-Case Specification: UC-50 — Thanh toán phí dịch vụ kết nối

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Xem Paywall, thanh toán phí của bên mình |
| Secondary | System | Tính phí, tạo QR, nhận webhook, mở khóa, kích hoạt hợp đồng |
| Secondary | Payment Gateway (External) | Nhận thanh toán, gửi webhook xác nhận |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) xem Bức tường thu phí (Paywall), nhập thông tin thuế (nếu là Sponsor), quét mã QR thanh toán, và theo dõi trạng thái. Khi cả hai bên hoàn tất (2/2), hệ thống tự động mở khóa thông tin liên hệ, kích hoạt tạo hợp đồng điện tử, và bắt đầu đếm ngược 72 giờ ký kết. Nếu hết 48 giờ mà chưa đủ 2/2, hệ thống tự động hoàn tiền cho bên đã nộp và hủy thương vụ.

---

### 2. Flow of Events

**Trigger**
> Actor truy cập trang deal đang ở trạng thái AWAITING_PAYMENT.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Truy cập trang deal đang ở trạng thái AWAITING_PAYMENT |
| 2 | System | Hiển thị Paywall: số tiền phí CỦA BÊN ACTOR, mã QR, đếm ngược 48 giờ, trạng thái tổng thể (0/2, 1/2, 2/2) |
| 3 | Authenticated User | Quét mã QR hoặc chuyển khoản theo thông tin trên Paywall |
| 4 | Payment Gateway | Xử lý thanh toán và gửi webhook xác nhận |
| 5 | System | Đối soát transaction_reference và số tiền chính xác |
| 6 | System | Cập nhật ServiceFeeTransaction sang PAID, ghi nhận paid_at |
| 7 | System | Cập nhật PaywallSession.payment_count (0→1 hoặc 1→2) |
| 8 | System | Gửi thông báo cho đối tác "Đối tác đã hoàn tất thanh toán phí" |
| 9 | System | Hiển thị trạng thái cập nhật trên Paywall |
| 10 | System | Nếu payment_count = 2/2: kích hoạt chuỗi sự kiện hoàn tất (xem Postconditions) |
| 11 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-50.a: Sponsor chưa nhập Mã số thuế
>
> *Triggered at Step 2 of the Basic Flow when actor là Sponsor và chưa nhập MST.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Hiển thị form nhập: MST, Tên doanh nghiệp, Địa chỉ doanh nghiệp |
| 2b | System | Vô hiệu hóa nút thanh toán cho đến khi MST hợp lệ |
| 2c | Sponsor | Nhập MST và thông tin doanh nghiệp |
| 2d | System | Xác thực MST (10 hoặc 13 chữ số), lưu thông tin thuế |
| 2e | System | Mở khóa nút thanh toán, quay lại Step 2 hiển thị Paywall đầy đủ |

##### AF-50.b: Chỉ một bên thanh toán — chờ đối tác
>
> *Triggered at Step 7 of the Basic Flow when payment_count = 1/2.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | System | Hiển thị "1/2 Đã thanh toán — Chờ đối tác" |
| 7b | System | Tiếp tục đếm ngược 48 giờ |

##### AF-50.c: Hết hạn 48 giờ — Auto-Refund
>
> *Triggered by System timer when PaywallSession hết hạn.*

| Step | Actor / System | Action |
|------|----------------|--------|
| T1 | System | Phát hiện PaywallSession hết hạn (expires_at < now) và status = ACTIVE |
| T2 | System | Nếu payment_count = 0: hủy deal (TERMINATED), thông báo cả hai |
| T3 | System | Nếu payment_count = 1: hoàn tiền 100% cho bên đã nộp, tạo RefundTransaction |
| T4 | System | Cập nhật ServiceFeeTransaction sang REFUNDED, PaywallSession sang EXPIRED |
| T5 | System | Chuyển deal sang TERMINATED (lý do: "Hết hạn thanh toán phí dịch vụ") |
| T6 | System | Thông báo bên đã nộp: "Phí dịch vụ sẽ được hoàn trong 3-5 ngày làm việc" |
| T7 | System | Thông báo đối tác: "Thương vụ bị hủy do không hoàn tất thanh toán" |

#### 2.3 Exception Flows

##### EF-50.1: Webhook không khớp số tiền
>
> *Triggered at Step 5 of the Basic Flow when số tiền không khớp.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Đánh dấu giao dịch MISMATCH, ghi log chi tiết |
| 5b | System | KHÔNG cập nhật PAID |
| 5c | System | Gửi cảnh báo admin để đối soát thủ công (→ UC-55) |

##### EF-50.2: Webhook với transaction_reference không tồn tại
>
> *Triggered at Step 5 of the Basic Flow when reference không khớp.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Ghi log UNMATCHED_WEBHOOK, trả HTTP 200 OK cho gateway |
| 5b | System | Gửi cảnh báo admin |

##### EF-50.3: MST không hợp lệ
>
> *Triggered at AF-50.a Step 2d when MST sai định dạng.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2d-a | System | Hiển thị "Mã số thuế phải là 10 hoặc 13 chữ số" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-50-01 | Thanh toán 2/2 thành công | Cả hai bên thanh toán; unlock + hợp đồng tự động |
| SC-50-02 | 1/2 chờ đối tác | Một bên thanh toán; chờ bên còn lại (AF-50.b) |
| SC-50-03 | Timeout auto-refund | Hết 48h, 1/2 → hoàn tiền + hủy deal (AF-50.c) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Deal đang AWAITING_PAYMENT

- Deal đang ở trạng thái AWAITING_PAYMENT

#### 5.3 PaywallSession tồn tại

- PaywallSession đã được tạo (từ UC-18 khi hai bên đồng thuận)

#### 5.4 Chưa thanh toán

- Actor chưa hoàn tất thanh toán phí của bên mình

---

### 6. Postconditions

#### 6.1 Success (2/2)

- Cả hai ServiceFeeTransaction có status = PAID
- PaywallSession.status = COMPLETED
- Data Masking được gỡ bỏ (SF-13 FR-1302)
- Deal chuyển sang AGREED
- Hợp đồng tự động tạo (SF-05 FR-0501)
- Đếm ngược 72 giờ ký kết bắt đầu (SF-05 FR-0507)
- Hóa đơn VAT tạo cho Sponsor (→ UC-52)
- Phí trở thành non-refundable (FR-1402)

#### 6.2 Success (1/2)

- Một ServiceFeeTransaction = PAID, một = PENDING
- PaywallSession vẫn ACTIVE

#### 6.3 Failure (hết hạn 48h)

- Bên đã nộp được hoàn tiền 100%
- Deal chuyển sang TERMINATED

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1201: Cơ cấu phí admin-configurable (CLB 1%/cap 100K, DN 2-5%/cap 3M)
- BR-1202: Paywall độc lập — mỗi bên chỉ thấy phí của mình
- BR-1203: Thời hạn 48 giờ chung cho cả hai bên
- BR-1204: Sponsor PHẢI nhập MST trước khi thanh toán
- BR-1205: Đối soát exact match (reference + amount), retry 3 lần
- BR-1206: Mở khóa CHỈ khi 2/2
- BR-1401: Auto-refund khi timeout 1/2
- BR-1402: Non-refundable sau 2/2
- BR-1403: Nhắc nhở T-24h, T-6h, T-1h cho bên chưa nộp

---

### 10. Additional Information

**Assumptions:**

- Thanh toán diễn ra NGOÀI nền tảng (external payment + webhook)
- Nhắc nhở thanh toán (FR-1403) được gửi tự động bởi system

**Related Use Cases:**

- UC-18: Xác nhận đồng thuận (prerequisite — tạo PaywallSession)
- UC-22: Ký hợp đồng điện tử (sequential — sau 2/2)
- UC-51: Xem trước chi phí (sequential — ước tính trước khi lock-in)
- UC-52: Xuất hóa đơn VAT (sequential — sau 2/2 cho Sponsor)
- UC-55: Đối soát thủ công (sequential — khi webhook thất bại)
