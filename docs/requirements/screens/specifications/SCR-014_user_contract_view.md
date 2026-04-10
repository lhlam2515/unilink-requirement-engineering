# SCR-014: User_ContractView_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-014 |
| **Screen Name** | User_ContractView_Screen |
| **Mục đích** | Authenticated User xem hợp đồng đã ký dạng read-only và xuất PDF |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 4 — Soạn thảo và ký kết hợp đồng tài trợ |
| **Use case liên quan** | UC-23 |

> **[UPDATED — BP03]** UC-24 (Yêu cầu hóa đơn VAT cho giá trị tài trợ) đã bị **LOẠI BỎ**. Nền tảng chỉ xuất hóa đơn VAT cho Phí quản lý chiến dịch (SF-12, UC-52 — tự động). Modal VAT và tất cả trường nhập liệu VAT đã bị xóa khỏi screen này.

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Xem hợp đồng đã ký — read-only context, khác edit |
| Data scope riêng | ✅ HĐ SIGNED + chữ ký |
| Action set riêng | ✅ Xuất PDF |
| Navigation boundary | ✅ Post-signature context |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Toàn bộ nội dung hợp đồng (read-only)
- Chữ ký điện tử của cả hai bên
- Mã hợp đồng (contract_number)
- Ngày ký (signing_date)
- Trạng thái: SIGNED

---

## Dữ liệu nhập (Input Fields)

Không có dữ liệu nhập. Screen hoàn toàn read-only.

> **[REMOVED — BP03]** Modal yêu cầu hóa đơn VAT (UC-24) và tất cả input fields liên quan đã bị loại bỏ.

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Xuất PDF** | Tải file PDF hợp đồng (UC-23 Main) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Xem nghĩa vụ | Chuyển đến SCR-015 (Obligation Dashboard) | — |
| Đánh giá đối tác | Chuyển đến SCR-019 (Partner Review) | HĐ kết thúc |

> **[REMOVED — BP03]** Hành động "Yêu cầu hóa đơn VAT" và "Xem/Tải lại hóa đơn VAT PDF" đã bị loại bỏ.

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0507: Chỉ HĐ SIGNED mới xuất PDF. Watermark "BẢN GỐC ĐIỆN TỬ"

> **[REMOVED — BP03]** BR-0508 (Hóa đơn VAT cho HĐ SIGNED + CASH) đã bị loại bỏ.

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-013 (sau cả hai ký) | Auto-redirect khi SIGNED |
| SCR-010 (Deal List) | Nhấn deal có HĐ SIGNED |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn "Xem nghĩa vụ" | SCR-015 (Obligation Dashboard) |
| Nhấn "Đánh giá đối tác" | SCR-019 (Partner Review) |
| Quay lại | SCR-010 (Deal List) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải HĐ |
| PDF Generating | Đang tạo PDF (UC-23 Main-2~4) |
| PDF Error | Lỗi tạo PDF (UC-23 EF-23.2) |
| Not Signed Error | HĐ chưa ký (UC-23 EF-23.1) |

> **[REMOVED — BP03]** Các states liên quan đến VAT (VAT Modal, VAT Success Toast, VAT Not Applicable, VAT Invalid Tax Code, VAT Already Issued) đã bị loại bỏ.

## UI Components liên quan

- Contract preview — nội dung HĐ read-only
- Signature display — hiển thị chữ ký 2 bên
- Download button — "Xuất PDF"
- Navigation links — nghĩa vụ, đánh giá

> **[REMOVED — BP03]** Các components liên quan đến VAT (CTA "Yêu cầu hóa đơn VAT", Modal dialog form VAT, Invoice summary card) đã bị loại bỏ.

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-23 | Main-1 | Nhấn "Xuất PDF" | Action | Download button |
| UC-23 | Main-2~4 | Tạo PDF + watermark | Action | System processing |
| UC-23 | Main-5 | Tải file | Action | Browser download |
| UC-23 | EF-23.1 | Chưa ký | UI State | Error |
| UC-23 | EF-23.2 | Lỗi tạo PDF | UI State | Error toast |

> **[REMOVED — BP03]** Toàn bộ UC-24 mapping (Main-1~10, EF-24.1~3) đã bị loại bỏ.
