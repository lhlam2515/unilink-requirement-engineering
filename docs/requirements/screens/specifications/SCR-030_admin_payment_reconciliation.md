# SCR-030: Admin_PaymentReconciliation_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-030 |
| **Screen Name** | Admin_PaymentReconciliation_Screen |
| **Mục đích** | Admin thực hiện đối soát thủ công khi webhook thanh toán thất bại, không khớp, hoặc bị trùng lặp. Admin xác nhận thanh toán dựa trên bằng chứng ngân hàng và cập nhật trạng thái ServiceFeeTransaction |
| **Actor chính** | Admin |
| **Quy trình nghiệp vụ** | BP-03 / Đối soát thanh toán (SF-14) |
| **Use case liên quan** | UC-55 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Đối soát giao dịch — operational admin task |
| Data scope riêng | ✅ ServiceFeeTransaction MISMATCH/UNMATCHED, webhook data |
| Action set riêng | ✅ Xác nhận thanh toán manual, từ chối, xác nhận trùng |
| Navigation boundary | ✅ Admin panel → reconciliation |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

### Danh sách giao dịch cần đối soát

Mỗi giao dịch hiển thị:

- Transaction reference
- Deal info (deal ID, đối tác, sự kiện)
- Bên nộp (payer_role): Organizer / Sponsor
- Số tiền kỳ vọng (fee_amount)
- Loại vấn đề: MISMATCH / UNMATCHED / NO_WEBHOOK
- Thời gian tạo giao dịch
- Trạng thái xử lý: chờ đối soát / đã xử lý

### Chi tiết giao dịch (Detail panel)

- Transaction reference
- Số tiền kỳ vọng (fee_amount)
- Webhook data (nếu có): số tiền nhận, reference, thời gian
- Deal info: tên sự kiện, tên đối tác
- PaywallSession info: expires_at, payment_count
- Processing result: MISMATCH / DUPLICATE / UNMATCHED
- Audit log (nếu đã xử lý trước đó)

---

## Dữ liệu nhập (Input Fields)

### Xác nhận thanh toán (UC-55 Main-6)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Bank reference thực tế | Text input | Bắt buộc | bank_reference |

### Từ chối đối soát (UC-55 AF-55.b)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Lý do từ chối | Textarea | Bắt buộc | rejection_reason |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Xác nhận thanh toán** | ServiceFeeTransaction → PAID + audit log (UC-55 Main-7~10) | MISMATCH / UNMATCHED / NO_WEBHOOK |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Từ chối đối soát | Giữ nguyên trạng thái, ghi log rejected (UC-55 AF-55.b) | — |
| Xác nhận trùng lặp | Đánh dấu DUPLICATE confirmed (UC-55 AF-55.a) | DUPLICATE |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-1405: Xử lý webhook idempotent, manual reconciliation yêu cầu audit log bắt buộc
- BR-1205: Đối soát dựa trên transaction_reference và exact match

---

## Quy tắc xác thực (Validation Rules)

| Trường | Quy tắc |
|--------|---------|
| bank_reference | Bắt buộc, không rỗng (khi xác nhận) |
| rejection_reason | Bắt buộc, không rỗng (khi từ chối) |

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Admin Menu | Menu "Đối soát thanh toán" |
| Admin notification | Cảnh báo hệ thống giao dịch cần đối soát |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | Admin Dashboard / Menu |
| Hoàn tất đối soát | Ở lại screen (danh sách cập nhật) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải danh sách giao dịch |
| Empty State | Không có giao dịch cần đối soát |
| Detail Panel Active | Chọn giao dịch để xem chi tiết |
| Already Processed State | Giao dịch đã được xử lý tự động trong lúc admin mở trang (UC-55 EF-55.1) |
| Confirm Payment Dialog | Xác nhận thanh toán + nhập bank reference |
| Reject Reconciliation Dialog | Từ chối + nhập lý do |
| Duplicate Confirm Dialog | Xác nhận giao dịch trùng lặp |
| Action Success Toast | Đối soát thành công |

## UI Components liên quan

- **Data table** — danh sách giao dịch cần đối soát với filter/sort
- **Detail panel** — chi tiết giao dịch (inline hoặc slide-over)
- **Issue type badge** — MISMATCH / UNMATCHED / NO_WEBHOOK / DUPLICATE
- **Confirm dialog** — xác nhận thanh toán với input bank_reference
- **Reject dialog** — từ chối đối soát với input lý do
- **Audit log display** — lịch sử xử lý (nếu có)
- **Toast notifications** — thành công/lỗi

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-55 | Main-1 | Truy cập trang đối soát | Screen entry | Page load |
| UC-55 | Main-2 | Hiển thị danh sách cần đối soát | Read-only data | Data table |
| UC-55 | Main-3 | Chọn giao dịch để đối soát | Action | Row click → detail panel |
| UC-55 | Main-4 | Hiển thị chi tiết: reference, amount, webhook | Read-only data | Detail panel |
| UC-55 | Main-5 | Admin xác nhận tiền đã vào TK | Action (external) | — |
| UC-55 | Main-6 | Nhập bank_reference + xác nhận | Input + Action | Confirm dialog |
| UC-55 | Main-7~10 | Cập nhật PAID + audit + kích hoạt | Action | System processing → toast |
| UC-55 | AF-55.a | Xác nhận giao dịch trùng lặp | Action | Duplicate confirm |
| UC-55 | AF-55.b | Từ chối đối soát | Action + Input | Reject dialog |
| UC-55 | EF-55.1 | Giao dịch đã xử lý tự động | UI State | Already Processed |
