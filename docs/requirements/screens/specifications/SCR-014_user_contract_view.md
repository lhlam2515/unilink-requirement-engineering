# SCR-014: User_ContractView_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-014 |
| **Screen Name** | User_ContractView_Screen |
| **Mục đích** | Authenticated User xem hợp đồng đã ký dạng read-only, xuất PDF, và yêu cầu hóa đơn VAT (dành cho Sponsor) |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 4 — Soạn thảo và ký kết hợp đồng tài trợ |
| **Use case liên quan** | UC-23, UC-24 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Xem hợp đồng đã ký — read-only context, khác edit |
| Data scope riêng | ✅ HĐ SIGNED + chữ ký + VAT invoice |
| Action set riêng | ✅ Xuất PDF, yêu cầu VAT |
| Navigation boundary | ✅ Post-signature context |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Toàn bộ nội dung hợp đồng (read-only)
- Chữ ký điện tử của cả hai bên
- Mã hợp đồng (contract_number)
- Ngày ký (signing_date)
- Trạng thái: SIGNED
- Thông tin hóa đơn VAT (nếu đã phát hành): số hóa đơn, giá trị, thuế

---

## Dữ liệu nhập (Input Fields)

### Modal: Yêu cầu hóa đơn VAT (UC-24 — chỉ Sponsor)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Tên doanh nghiệp | Text input | Bắt buộc | business_name |
| Mã số thuế | Text input | Bắt buộc, 10 hoặc 13 chữ số | tax_code |
| Địa chỉ doanh nghiệp | Text input | Bắt buộc | business_address |
| Mô tả dịch vụ | Textarea | Bắt buộc | service_description |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Xuất PDF** | Tải file PDF hợp đồng (UC-23 Main) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Yêu cầu hóa đơn VAT | Mở modal VAT form (UC-24) | Sponsor + CASH + chưa có hóa đơn |
| Xem/Tải lại hóa đơn VAT PDF | Tải PDF hóa đơn | Đã có hóa đơn |
| Xem nghĩa vụ | Chuyển đến SCR-015 (Obligation Dashboard) | — |
| Đánh giá đối tác | Chuyển đến SCR-019 (Partner Review) | HĐ kết thúc |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0507: Chỉ HĐ SIGNED mới xuất PDF. Watermark "BẢN GỐC ĐIỆN TỬ"
- BR-0508: Hóa đơn VAT chỉ cho HĐ SIGNED + CASH. MST 10/13 chữ số. Mỗi HĐ tối đa 1 hóa đơn

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
| VAT Modal | Form yêu cầu hóa đơn |
| VAT Success Toast | Hóa đơn phát hành thành công |
| VAT Not Applicable | Hình thức không có CASH (UC-24 EF-24.1) |
| VAT Invalid Tax Code | MST sai định dạng (UC-24 EF-24.2) |
| VAT Already Issued | Đã có hóa đơn (UC-24 EF-24.3) |

## UI Components liên quan

- Contract preview — nội dung HĐ read-only
- Signature display — hiển thị chữ ký 2 bên
- Download button — "Xuất PDF"
- CTA — "Yêu cầu hóa đơn VAT" (conditional)
- Modal dialog — form VAT
- Invoice summary card — thông tin hóa đơn đã phát hành
- Navigation links — nghĩa vụ, đánh giá

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-23 | Main-1 | Nhấn "Xuất PDF" | Action | Download button |
| UC-23 | Main-2~4 | Tạo PDF + watermark | Action | System processing |
| UC-23 | Main-5 | Tải file | Action | Browser download |
| UC-23 | EF-23.1 | Chưa ký | UI State | Error |
| UC-23 | EF-23.2 | Lỗi tạo PDF | UI State | Error toast |
| UC-24 | Main-1 | Nhấn "Yêu cầu VAT" | Action | CTA button |
| UC-24 | Main-2~4 | Nhập thông tin | Input | Modal form |
| UC-24 | Main-5~10 | Validate + tính thuế + tạo | Action | System processing |
| UC-24 | EF-24.1 | Không có CASH | UI State | CTA hidden/disabled |
| UC-24 | EF-24.2 | MST sai | UI State | Validation error |
| UC-24 | EF-24.3 | Đã có hóa đơn | UI State | "Tải lại hóa đơn" |
