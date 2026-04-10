# SCR-027: User_ServiceFeePaywall_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-027 |
| **Screen Name** | User_ServiceFeePaywall_Screen |
| **Mục đích** | Authenticated User xem Paywall, nhập thông tin thuế (nếu Sponsor), quét QR thanh toán phí dịch vụ, theo dõi trạng thái thanh toán 2 bên, và chờ mở khóa thông tin liên hệ. Nếu quá hạn 72h ký kết, user có thể tố cáo đối tác vi phạm |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-03 / Bước 2 — Thanh toán phí dịch vụ |
| **Use case liên quan** | UC-50, UC-49 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Thanh toán — goal hoàn toàn khác thương thảo/soạn HĐ |
| Data scope riêng | ✅ PaywallSession, ServiceFeeTransaction, QR code, countdown 48h |
| Action set riêng | ✅ Quét QR, nhập MST, theo dõi payment_count, tố cáo vi phạm |
| Navigation boundary | ✅ Context switch rõ ràng từ deal negotiation sang payment |
| Independently testable | ✅ |

**Lý do tạo screen riêng:** Paywall tạo ra **transaction boundary** rõ ràng — user chuyển từ context thương thảo sang context thanh toán với data set hoàn toàn khác (QR code, fee amount, countdown timer, payment status). Đây là pattern **Browse → Transaction**. UC-49 (tố cáo vi phạm) gộp vào screen này vì trigger từ cùng Paywall context sau khi 72h ký kết quá hạn — form report đơn giản không cần screen riêng.

---

## Dữ liệu hiển thị (Read-only Data)

### Header / Deal Context

- Tên đối tác (partner_name)
- Tên sự kiện (event_name)
- Trạng thái deal: AWAITING_PAYMENT
- Thông tin thỏa thuận nháp: hình thức tài trợ, giá trị

### Panel: Paywall — Thanh toán phí (UC-50)

- Số tiền phí **CỦA BÊN ACTOR** (không hiển thị phí đối tác) — BR-1202
- Mã QR thanh toán
- Thông tin chuyển khoản: số tài khoản, nội dung CK (transaction_reference)
- Đếm ngược thời hạn 48 giờ (expires_at)
- Trạng thái thanh toán tổng thể: 0/2, 1/2, 2/2
- Trạng thái bên actor: PENDING / PAID
- Nhắc nhở thanh toán đối tác (nếu 1/2)

### Panel: Kết quả (sau 2/2)

- Thông báo mở khóa thành công
- Link chuyển sang soạn thảo hợp đồng (SCR-012)
- Đếm ngược 72 giờ ký kết (signing_deadline_at)

---

## Dữ liệu nhập (Input Fields)

### Form Thông tin thuế — chỉ Sponsor (UC-50 AF-50.a)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Mã số thuế (MST) | Text input | Bắt buộc (Sponsor), 10 hoặc 13 chữ số | tax_code |
| Tên doanh nghiệp | Text input | Bắt buộc (Sponsor) | business_name |
| Địa chỉ doanh nghiệp | Text input | Bắt buộc (Sponsor) | business_address |

### Form Tố cáo vi phạm (UC-49 Main-3~4) — conditional

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Lý do tố cáo | Textarea | Bắt buộc | report_reason |
| Bằng chứng | File upload | Tùy chọn | evidence_files |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Thanh toán (quét QR / chuyển khoản)** | Webhook → ServiceFeeTransaction = PAID (UC-50 Main) | AWAITING_PAYMENT + chưa PAID |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Tố cáo đối tác | Mở form report (UC-49 Main) | Deal quá hạn 72h ký kết + actor đã hoàn thành nghĩa vụ |
| Chuyển đến soạn thảo HĐ | SCR-012 (Contract Edit) | 2/2 thanh toán hoàn tất |
| Quay lại thương thảo | SCR-011 (read-only nếu AWAITING_PAYMENT) | — |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-1201: Cơ cấu phí admin-configurable (CLB 1%/cap 100K, DN 2-5%/cap 3M)
- BR-1202: Paywall độc lập — mỗi bên chỉ thấy phí của mình
- BR-1203: Thời hạn 48h chung cho cả hai bên
- BR-1204: Sponsor PHẢI nhập MST trước khi thanh toán
- BR-1205: Đối soát exact match (reference + amount), retry 3 lần
- BR-1206: Mở khóa CHỈ khi 2/2
- BR-1401: Auto-refund khi timeout 1/2
- BR-1402: Non-refundable sau 2/2
- BR-1403: Nhắc nhở T-24h, T-6h, T-1h cho bên chưa nộp
- BR-1406: Chỉ bên đã hoàn tất nghĩa vụ mới được tố cáo khi quá hạn 72h
- BR-1407: Sau report hợp lệ → cấp thêm 24h ân hạn
- BR-1408: Hết ân hạn mà chưa đủ 2 chữ ký → đóng thương vụ + chế tài

---

## Quy tắc xác thực (Validation Rules)

| Trường | Quy tắc |
|--------|---------|
| tax_code | 10 hoặc 13 chữ số (Sponsor only) |
| business_name | Bắt buộc, không rỗng (Sponsor only) |
| business_address | Bắt buộc, không rỗng (Sponsor only) |
| report_reason | Bắt buộc, không rỗng |

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-011 (Deal Negotiation) | Tự động redirect khi deal chuyển AWAITING_PAYMENT (UC-18 Main-11) |
| SCR-010 (Deal List) | Nhấn deal ở trạng thái AWAITING_PAYMENT |
| SCR-013 (Contract Sign) | Quá hạn 72h → hiển thị nút tố cáo → redirect |
| In-app notification | Thông báo nhắc nhở thanh toán (BR-1403) |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| 2/2 thanh toán hoàn tất | SCR-012 (Contract Edit) — tự động redirect |
| Quay lại | SCR-011 (Deal Negotiation) hoặc SCR-010 (Deal List) |
| Hết hạn 48h / Deal TERMINATED | SCR-010 (Deal List) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải PaywallSession |
| Tax Info Required Banner | Sponsor chưa nhập MST (UC-50 AF-50.a) |
| Tax Info Form | Form nhập MST, tên DN, địa chỉ (UC-50 AF-50.a) |
| Tax Validation Error | MST không đúng định dạng (UC-50 EF-50.3) |
| QR Active Display | QR code thanh toán đang active |
| Payment Processing | Đang xử lý webhook |
| My Payment Done | Actor đã thanh toán xong |
| One Party Paid (1/2) | Một bên thanh toán xong, chờ đối tác (UC-50 AF-50.b) |
| Both Paid Success (2/2) | Cả hai hoàn tất → success banner + mở khóa + redirect tự động (UC-50 Main-10) |
| Countdown Timer (48h) | Đếm ngược 48h chung |
| Countdown Warning | T-6h, T-1h cảnh báo |
| Expired State | Hết 48h → hiển thị thông báo auto-refund/hủy (UC-50 AF-50.c) |
| Mismatch Alert | Webhook không khớp số tiền — admin sẽ đối soát (UC-50 EF-50.1) |
| Breach Report Form | Form tố cáo đối tác (UC-49 Main-3~4) — conditional |
| Breach Submitted Toast | Report tố cáo đã gửi thành công |
| Breach Not Yet Error | Chưa quá hạn 72h (UC-49 EF-49.1) |
| Grace Period Timer (24h) | Đếm ngược 24h ân hạn sau report (UC-49 AF-49.a) |

## UI Components liên quan

- **QR code display** — mã QR thanh toán
- **Fee summary card** — thông tin phí tóm tắt (số tiền, loại phí)
- **Tax info form** — form nhập MST/tên/địa chỉ DN (Sponsor only)
- **Payment status tracker** — progress indicator 0/2, 1/2, 2/2
- **Countdown timer** — đếm ngược 48h (và 24h ân hạn nếu có)
- **Transfer info card** — thông tin chuyển khoản (STK, nội dung CK)
- **Notification toast** — thanh toán thành công / lỗi / cảnh báo
- **Breach report modal** — form tố cáo đối tác (conditional)
- **Success banner** — mở khóa thành công + link SCR-012

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-50 | Main-1 | Truy cập deal AWAITING_PAYMENT | Screen entry | Page load |
| UC-50 | Main-2 | Hiển thị Paywall: phí + QR + countdown | Read-only data | Fee card + QR + timer |
| UC-50 | AF-50.a-2b~2f | Sponsor nhập MST | Input + Action | Tax info form |
| UC-50 | EF-50.3 | MST không hợp lệ | UI State | Tax validation error |
| UC-50 | Main-3~4 | Quét QR + webhook | Action (external) | QR display |
| UC-50 | Main-5~6 | Đối soát + cập nhật PAID | Action | Payment processing state |
| UC-50 | Main-7~9 | Cập nhật count + thông báo | Action + State | Payment status tracker |
| UC-50 | Main-10 | 2/2 → mở khóa + redirect | Action + Redirect | Success banner → SCR-012 |
| UC-50 | AF-50.b | 1/2 chờ đối tác | UI State | One Party Paid display |
| UC-50 | AF-50.c-T1~T7 | Hết 48h auto-refund | UI State | Expired State |
| UC-50 | EF-50.1 | Webhook mismatch | UI State | Mismatch Alert |
| UC-50 | EF-50.2 | Transaction reference không tồn tại | Action (system) | No UI impact |
| UC-49 | Main-1 | Nhấn "Tố cáo đối tác" | Action | CTA button (conditional) |
| UC-49 | Main-2 | Kiểm tra quá hạn 72h | Action | System validation |
| UC-49 | Main-3~4 | Nhập lý do + gửi report | Input + Action | Breach report modal |
| UC-49 | Main-5~9 | Tạo breach case + đóng băng + 24h | Action + State | Breach submitted + grace timer |
| UC-49 | AF-49.a | Bên vi phạm ký trong 24h | Action + State | Grace Period Timer → success |
| UC-49 | EF-49.1 | Chưa quá hạn 72h | UI State | Breach Not Yet Error |
| UC-49 | EF-49.2 | Thiếu lý do tố cáo | UI State | Validation error |
