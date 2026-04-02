# SCR-017: Organizer_EventReport_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-017 |
| **Screen Name** | Organizer_EventReport_Screen |
| **Mục đích** | Organizer nộp báo cáo kết quả sự kiện cho sponsor sau khi sự kiện kết thúc, bao gồm số liệu thực tế, media, và đánh giá mức độ hoàn thành quyền lợi |
| **Actor chính** | Organizer (BTC) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 5 — Thực hiện nghĩa vụ tài trợ |
| **Use case liên quan** | UC-28 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Nộp báo cáo sự kiện — mục tiêu riêng, form phức tạp |
| Data scope riêng | ✅ Số liệu thực tế, media, đánh giá quyền lợi — khác obligations |
| Action set riêng | ✅ Nhập báo cáo, upload media, submit |
| Navigation boundary | ✅ Tách khỏi obligation detail do data scope khác |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Thông tin hợp đồng liên quan (tóm tắt)
- Thông tin sự kiện (tên, thời gian, địa điểm)
- Danh sách quyền lợi nhà tài trợ theo hợp đồng (để đánh giá mức hoàn thành)

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Tóm tắt kết quả sự kiện | Textarea / Rich text | Bắt buộc | event_summary |
| Lượng khán giả thực tế | Number input | Bắt buộc | actual_audience |
| Reach truyền thông | Number input | Tùy chọn | media_reach |
| Đánh giá hoàn thành quyền lợi | Textarea (per benefit) | Tùy chọn | benefit_fulfillment_notes |
| Hình ảnh/Video sự kiện | File upload (multiple) | Tùy chọn, JPEG/PNG/WebP/MP4 | event_media |
| File báo cáo chi tiết | File upload | Tùy chọn, PDF/DOCX | report_attachment |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Nộp báo cáo** | Lưu + gửi thông báo cho sponsor (UC-28 Main) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Quay lại | SCR-015 (Obligation Dashboard) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0604: Báo cáo chỉ cho HĐ SIGNED. Mỗi HĐ chỉ 1 báo cáo

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-015 (Obligation Dashboard) | Link "Nộp báo cáo kết quả" |
| SCR-014 (Contract View) | Link "Nộp báo cáo" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nộp thành công | SCR-015 (Obligation Dashboard) |
| Quay lại | SCR-015 |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải |
| Not Signed Error | HĐ chưa ký (UC-28 EF-28.1) |
| Already Submitted Error | Đã có báo cáo (UC-28 EF-28.2) — hiển thị link chỉnh sửa |
| Submit Success Toast | Nộp thành công |
| Upload Progress | Đang upload media |

## UI Components liên quan

- Contract summary card — thông tin hợp đồng/sự kiện
- Form sections — tóm tắt, số liệu, media, quyền lợi, attachment
- Number inputs — khán giả, reach
- Media upload gallery — ảnh/video
- File upload — báo cáo chi tiết
- Benefits checklist — danh sách quyền lợi + đánh giá
- Submit button

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-28 | Main-1 | Nhấn "Nộp báo cáo" | Screen entry | Page load |
| UC-28 | Main-2 | Hiển thị form báo cáo | Screen content | Form sections |
| UC-28 | Main-3 | Nhập tóm tắt | Input | Textarea |
| UC-28 | Main-4 | Nhập số liệu | Input | Number inputs |
| UC-28 | Main-5 | Upload media | Input | Media upload |
| UC-28 | Main-6 | Đánh giá quyền lợi | Input | Benefits notes |
| UC-28 | Main-7 | Đính kèm báo cáo | Input | File upload |
| UC-28 | Main-8~11 | Nộp + lưu + thông báo | Action | Submit button |
| UC-28 | EF-28.1 | HĐ chưa ký | UI State | Error |
| UC-28 | EF-28.2 | Đã có báo cáo | UI State | Edit/view existing |
