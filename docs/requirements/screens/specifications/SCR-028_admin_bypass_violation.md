# SCR-028: Admin_BypassViolationList_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-028 |
| **Screen Name** | Admin_BypassViolationList_Screen |
| **Mục đích** | Admin xem xét và xử lý các vi phạm lách bộ lọc Data Masking: xem chi tiết tin nhắn vi phạm, xác nhận vi phạm, đánh dấu false positive (gỡ FLAGGED), hoặc leo thang khóa tài khoản vĩnh viễn |
| **Actor chính** | Admin |
| **Quy trình nghiệp vụ** | BP-03 / Quản trị Data Masking (SF-13) |
| **Use case liên quan** | UC-53 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Quản trị vi phạm masking — goal admin riêng biệt |
| Data scope riêng | ✅ BypassViolationLog, tin nhắn gốc, lịch sử vi phạm user |
| Action set riêng | ✅ Xác nhận / False Positive / Leo thang |
| Navigation boundary | ✅ Admin panel → violation management |
| Independently testable | ✅ |

**Lý do gộp List + Detail (master-detail):** UC-53 có luồng đơn giản: danh sách → chọn → xem chi tiết → quyết định. Data scope tương đối nhỏ. Gộp list + detail vào 1 screen dạng master-detail là phù hợp, tương tự pattern SCR-025 (Admin Verification Detail).

---

## Dữ liệu hiển thị (Read-only Data)

### Panel: Danh sách vi phạm (List)

- Danh sách BypassViolationLog chưa review (sắp xếp thời gian mới nhất)
- Mỗi item: user vi phạm, deal context, phương pháp phát hiện, thời gian, trạng thái (chờ review / đã xử lý)

### Panel: Chi tiết vi phạm (Detail — khi chọn 1 item)

- Nội dung tin nhắn gốc (full text)
- Phương pháp phát hiện: REGEX / DICTIONARY / HEURISTIC
- Deal context: deal ID, đối tác, sự kiện
- Lịch sử vi phạm của user (tổng vi phạm, chi tiết từng lần)
- Số lần vi phạm hiện tại (violation_count)
- Hành động tự động đã áp dụng (cảnh báo / khóa tạm)
- Admin đã xử lý (nếu đã review): admin name, thời gian, quyết định

---

## Dữ liệu nhập (Input Fields)

### Leo thang — Khóa tài khoản (UC-53 AF-53.b)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Lý do khóa | Textarea | Bắt buộc | escalation_reason |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Xác nhận vi phạm** | admin_decision = CONFIRMED_VIOLATION (UC-53 Main-5~8) | Chưa review |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Đánh dấu False Positive | Gỡ FLAGGED, giảm violation_count, gỡ khóa tạm (UC-53 AF-53.a) | Chưa review |
| Leo thang — Khóa vĩnh viễn | Confirm dialog → khóa tài khoản vĩnh viễn (UC-53 AF-53.b) | Chưa review |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-1303: Zero Tolerance — vi phạm 1-2: cảnh báo, 3: khóa tạm, 3+: admin review → có thể khóa vĩnh viễn
- BR-1303: Thuật toán ưu tiên false positive hơn false negative

---

## Quy tắc xác thực (Validation Rules)

| Trường | Quy tắc |
|--------|---------|
| escalation_reason | Bắt buộc, không rỗng (khi leo thang) |

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Admin Menu | Menu "Quản lý vi phạm Data Masking" |
| Admin notification | Thông báo vi phạm mới |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | Admin Dashboard / Menu |
| Hoàn tất xử lý vi phạm | Ở lại screen (danh sách cập nhật) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải danh sách vi phạm |
| Empty State | Không có vi phạm chờ review |
| Detail Panel Active | Chọn một vi phạm → hiển thị chi tiết |
| Already Reviewed State | Vi phạm đã xử lý bởi admin khác (UC-53 EF-53.1) — read-only |
| Escalation Confirm Dialog | Xác nhận khóa tài khoản vĩnh viễn (UC-53 AF-53.b) |
| Action Success Toast | Quyết định đã lưu thành công |
| False Positive Success | Tin nhắn được gỡ FLAGGED, hiển thị lại |

## UI Components liên quan

- **Data table** — danh sách vi phạm với filter/sort
- **Detail panel** — chi tiết vi phạm (inline hoặc slide-over)
- **Decision button group** — Xác nhận vi phạm / False Positive / Leo thang
- **Confirm dialog** — xác nhận khóa vĩnh viễn (destructive action)
- **Status badge** — chờ review / đã xử lý / escalated
- **User violation history** — timeline vi phạm của user
- **Message content viewer** — hiển thị tin nhắn gốc + highlight phần vi phạm
- **Toast notifications** — thành công/lỗi

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-53 | Main-1 | Truy cập trang quản lý vi phạm | Screen entry | Page load |
| UC-53 | Main-2 | Hiển thị danh sách chưa review | Read-only data | Data table |
| UC-53 | Main-3 | Chọn vi phạm để xem chi tiết | Action | Row click → detail panel |
| UC-53 | Main-4 | Hiển thị chi tiết: tin nhắn, method, history | Read-only data | Detail panel |
| UC-53 | Main-5~8 | Xác nhận vi phạm | Action | Decision button → toast |
| UC-53 | AF-53.a-5a~5e | False Positive | Action | FP button → gỡ FLAGGED |
| UC-53 | AF-53.b-5a~5f | Leo thang khóa vĩnh viễn | Action + Dialog | Escalate → confirm → lock |
| UC-53 | EF-53.1 | Vi phạm đã review trước đó | UI State | Already Reviewed (read-only) |
