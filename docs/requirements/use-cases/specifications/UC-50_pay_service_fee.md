# UC-50: Thanh toán phí dịch vụ kết nối

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) xem Bức tường thu phí (Paywall), nhập thông tin thuế (nếu là Sponsor), quét mã QR thanh toán, và theo dõi trạng thái thanh toán phí dịch vụ. Khi cả hai bên hoàn tất thanh toán (2/2), hệ thống tự động mở khóa thông tin liên hệ, kích hoạt tạo hợp đồng điện tử, và bắt đầu đếm ngược 72 giờ ký kết. Nếu hết 48 giờ mà chưa đủ 2/2, hệ thống tự động hoàn tiền cho bên đã nộp và hủy thương vụ.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Xem Paywall, thanh toán phí của bên mình |
| Secondary | System | Tính phí, tạo QR, nhận webhook, mở khóa, kích hoạt hợp đồng |
| Secondary | Payment Gateway (External) | Nhận thanh toán, gửi webhook xác nhận |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái AWAITING_PAYMENT
- PaywallSession đã được tạo (từ UC-18 khi hai bên đồng thuận)
- Phí dịch vụ đã được tính tự động (FR-1201)
- Actor chưa hoàn tất thanh toán phí của bên mình

---

**Trigger**
> Actor truy cập trang deal đang ở trạng thái AWAITING_PAYMENT.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Truy cập trang deal đang ở trạng thái AWAITING_PAYMENT |
| 2 | System | Hiển thị Paywall với: số tiền phí CỦA BÊN ACTOR (không hiển thị phí đối tác), mã QR thanh toán, đếm ngược thời hạn 48 giờ, trạng thái thanh toán tổng thể (0/2, 1/2, 2/2) |
| 3 | Authenticated User | Quét mã QR hoặc chuyển khoản theo thông tin trên Paywall |
| 4 | Payment Gateway | Xử lý thanh toán và gửi webhook xác nhận về hệ thống |
| 5 | System | Đối soát transaction_reference và số tiền chính xác |
| 6 | System | Cập nhật ServiceFeeTransaction sang PAID, ghi nhận paid_at |
| 7 | System | Cập nhật PaywallSession.payment_count (0→1 hoặc 1→2) |
| 8 | System | Gửi thông báo cho đối tác "Đối tác đã hoàn tất thanh toán phí" |
| 9 | System | Hiển thị trạng thái cập nhật trên Paywall cho cả hai bên |
| 10 | System | Nếu payment_count = 2/2: kích hoạt chuỗi sự kiện hoàn tất (xem Postconditions) |
| 11 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-50.a: Sponsor chưa nhập Mã số thuế (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện actor là Sponsor và chưa nhập MST |
| 2b | System | Hiển thị form nhập: Mã số thuế (MST), Tên doanh nghiệp, Địa chỉ doanh nghiệp |
| 2c | System | Vô hiệu hóa nút thanh toán cho đến khi MST hợp lệ |
| 2d | Sponsor | Nhập MST và thông tin doanh nghiệp |
| 2e | System | Xác thực MST (10 hoặc 13 chữ số), lưu thông tin thuế |
| 2f | System | Mở khóa nút thanh toán, quay lại Step 2 hiển thị Paywall đầy đủ |

> AF-50.b: Chỉ một bên thanh toán — chờ đối tác (triggered at Step 7)

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | System | Phát hiện payment_count = 1/2 (chỉ một bên đã nộp) |
| 7b | System | Hiển thị trạng thái "1/2 Đã thanh toán — Chờ đối tác" |
| 7c | System | Tiếp tục đếm ngược 48 giờ |
| 7d | System | Use case kết thúc (chưa hoàn thành — chờ đối tác thanh toán) |

> AF-50.c: Hết hạn 48 giờ — Auto-Refund (triggered by System timer)

| Step | Actor / System | Action |
|------|----------------|--------|
| T1 | System | Phát hiện PaywallSession hết hạn (expires_at < now) và status = ACTIVE |
| T2 | System | Nếu payment_count = 0: hủy deal (TERMINATED), thông báo cả hai bên |
| T3 | System | Nếu payment_count = 1: hoàn tiền 100% cho bên đã nộp, tạo RefundTransaction |
| T4 | System | Cập nhật ServiceFeeTransaction sang REFUNDED, PaywallSession sang EXPIRED |
| T5 | System | Chuyển deal sang TERMINATED (lý do: "Hết hạn thanh toán phí dịch vụ") |
| T6 | System | Thông báo bên đã nộp: "Phí dịch vụ sẽ được hoàn trong 3-5 ngày làm việc" |
| T7 | System | Thông báo đối tác: "Thương vụ bị hủy do không hoàn tất thanh toán" |

---

**Exception Flows**

> EF-50.1: Webhook không khớp số tiền (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện số tiền trong webhook không khớp fee_amount |
| 5b | System | Đánh dấu giao dịch MISMATCH, ghi log chi tiết |
| 5c | System | KHÔNG cập nhật trạng thái PAID |
| 5d | System | Gửi cảnh báo admin để đối soát thủ công (→ UC-55) |

> EF-50.2: Webhook với transaction_reference không tồn tại (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện transaction_reference không khớp bản ghi nào |
| 5b | System | Ghi log UNMATCHED_WEBHOOK, trả HTTP 200 OK cho gateway |
| 5c | System | Gửi cảnh báo admin |

> EF-50.3: MST không hợp lệ (triggered at AF-50.a Step 2e)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2e-a | System | Phát hiện MST không đúng định dạng (không phải 10 hoặc 13 chữ số) |
| 2e-b | System | Hiển thị lỗi "Mã số thuế phải là 10 hoặc 13 chữ số" |
| 2e-c | Sponsor | Sửa lại MST và thử lại |

---

**Postconditions**

*Success (2/2 thanh toán hoàn tất):*
- Cả hai ServiceFeeTransaction có status = PAID
- PaywallSession.status = COMPLETED
- Data Masking được gỡ bỏ — thông tin liên hệ thật hiển thị (SF-13 FR-1302)
- Deal chuyển sang trạng thái AGREED
- Hợp đồng được tạo tự động (SF-05 FR-0501)
- Đếm ngược 72 giờ ký kết bắt đầu (SF-05 FR-0507)
- Hóa đơn VAT phí dịch vụ được tạo cho Sponsor (→ UC-52)
- Doanh thu nền tảng được ghi nhận (FR-1404)
- Phí trở thành non-refundable (FR-1402)

*Success (1/2 — chờ đối tác):*
- Một ServiceFeeTransaction = PAID, một = PENDING
- PaywallSession vẫn ACTIVE, đếm ngược tiếp tục

*Failure (hết hạn 48h):*
- Bên đã nộp được hoàn tiền 100%
- Deal chuyển sang TERMINATED
- PaywallSession.status = EXPIRED

---

**Business Rules**

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

**Notes / Assumptions**

- Use case này là UC trung tâm của BP03, tích hợp nhiều FR tự động
- Thanh toán diễn ra NGOÀI nền tảng (external payment + webhook)
- Nhắc nhở thanh toán (FR-1403) được nhúng vào postcondition — system gửi tự động
- Liên kết: UC-18 (đồng thuận → tạo Paywall), UC-52 (VAT invoice), UC-55 (đối soát thủ công), UC-22 (ký hợp đồng sau 2/2)
