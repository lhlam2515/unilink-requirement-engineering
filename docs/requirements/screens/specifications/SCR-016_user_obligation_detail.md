# SCR-016: User_ObligationDetail_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-016 |
| **Screen Name** | User_ObligationDetail_Screen |
| **Mục đích** | Authenticated User xem chi tiết nghĩa vụ cụ thể, nộp bằng chứng hoàn thành (bên thực hiện), hoặc xác nhận/từ chối hoàn thành (bên đối tác) |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor — tùy vai trò) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 5 — Thực hiện nghĩa vụ tài trợ |
| **Use case liên quan** | UC-25 (AF-25.a), UC-26, UC-27 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Chi tiết nghĩa vụ — context switch từ dashboard |
| Data scope riêng | ✅ Toàn bộ data của 1 obligation + bằng chứng + lịch sử |
| Action set riêng | ✅ Submit evidence / Confirm/Dispute — role-based |
| Navigation boundary | ✅ Detail view từ list |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Tên nghĩa vụ (obligation_name)
- Mô tả chi tiết (description)
- Trạng thái: PENDING / IN_PROGRESS / SUBMITTED / CONFIRMED / DISPUTED / OVERDUE
- Deadline
- Bên chịu trách nhiệm (responsible_party)
- Bằng chứng đã nộp (nếu có): mô tả, file đính kèm, thời gian nộp
- Lý do từ chối (nếu DISPUTED)
- Lịch sử thay đổi trạng thái

---

## Dữ liệu nhập (Input Fields)

### Form Nộp bằng chứng (UC-26 — bên thực hiện)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Mô tả bằng chứng | Textarea | Bắt buộc, ≥ 20 ký tự | evidence_description |
| File đính kèm | File upload (multiple) | Tùy chọn | evidence_files |

### Form Từ chối (UC-27 AF-27.a — bên đối tác)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Lý do từ chối | Textarea | Bắt buộc | dispute_reason |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Báo cáo hoàn thành** | Nghĩa vụ → SUBMITTED (UC-26 Main) | Bên thực hiện + PENDING/IN_PROGRESS/DISPUTED |
| **Xác nhận hoàn thành** | Nghĩa vụ → CONFIRMED (UC-27 Main) | Bên đối tác + SUBMITTED |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Từ chối | Nghĩa vụ → DISPUTED (UC-27 AF-27.a) | Bên đối tác + SUBMITTED |
| Quay lại | SCR-015 (Obligation Dashboard) | — |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0602: Bằng chứng phải có mô tả ≥ 20 ký tự. File tùy chọn. CONFIRMED không nộp lại
- BR-0603: Chỉ bên ĐỐI TÁC xác nhận/từ chối. Lý do từ chối bắt buộc

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-015 (Obligation Dashboard) | Nhấn vào nghĩa vụ |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại | SCR-015 |
| Sau submit/confirm | Ở lại SCR-016 (cập nhật trạng thái) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải |
| Responsible View | Actor là bên thực hiện — hiển thị form nộp bằng chứng |
| Partner View | Actor là bên đối tác — hiển thị Confirm/Dispute |
| DISPUTED — Resubmit | Nghĩa vụ DISPUTED: hiển thị lý do + cho phép nộp lại (UC-26 AF-26.a) |
| CONFIRMED State | Nghĩa vụ đã xác nhận — read-only |
| Already CONFIRMED Error | Cố nộp lại khi CONFIRMED (UC-26 EF-26.1) |
| Self-Confirm Error | Bên thực hiện tự xác nhận (UC-27 EF-27.1) |
| Short Description Error | Mô tả < 20 ký tự (UC-26 EF-26.2) |
| Submit Success Toast | Nộp bằng chứng thành công |
| Confirm Success Toast | Xác nhận hoàn thành thành công |
| Dispute Success Toast | Từ chối thành công |

## UI Components liên quan

- Obligation info card — thông tin chi tiết nghĩa vụ
- Status badge — trạng thái hiện tại
- Evidence display — bằng chứng đã nộp (text + files)
- Evidence form — nộp bằng chứng mới
- File upload
- Dispute reason display — lý do từ chối (DISPUTED)
- Action buttons — Submit / Confirm / Dispute (conditional)
- Dispute form — nhập lý do từ chối
- Status history timeline — lịch sử thay đổi

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-25 | AF-25.a Step 3a | Nhấn vào nghĩa vụ | Screen entry | Page load |
| UC-25 | AF-25.a Step 3b | Hiển thị chi tiết | Read-only data | Obligation card |
| UC-26 | Main-1 | Nhấn "Báo cáo hoàn thành" | Action | CTA button |
| UC-26 | Main-2 | Hiển thị form nộp | Component | Evidence form |
| UC-26 | Main-3~5 | Nhập mô tả + file + nộp | Input + Action | Form fields |
| UC-26 | Main-6~9 | Validate + lưu + → SUBMITTED | Action | System processing |
| UC-26 | AF-26.a | Nộp lại sau DISPUTED | Component | Resubmit with dispute context |
| UC-26 | EF-26.1 | Đã CONFIRMED | UI State | Error |
| UC-26 | EF-26.2 | Mô tả < 20 | UI State | Validation error |
| UC-27 | Main-1~2 | Mở chi tiết SUBMITTED | Screen entry | Page load |
| UC-27 | Main-3~4 | Kiểm tra + nhấn "Xác nhận" | Action | Confirm button |
| UC-27 | Main-5~6 | → CONFIRMED + thông báo | Action | System processing |
| UC-27 | AF-27.a Step 4a~4f | Từ chối + lý do | Action + Input | Dispute flow |
| UC-27 | EF-27.1 | Bên thực hiện tự xác nhận | UI State | Error |
