# SCR-013: User_ContractSign_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-013 |
| **Screen Name** | User_ContractSign_Screen |
| **Mục đích** | Authenticated User ký chữ ký điện tử lên hợp đồng đã được xác nhận nội dung bởi cả hai bên |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 4 — Soạn thảo và ký kết hợp đồng tài trợ |
| **Use case liên quan** | UC-22, UC-49 (breach trigger) |

> **[UPDATED — BP03]** Bổ sung đếm ngược 72 giờ ký kết (hard-lock timer). Nếu hết hạn mà chưa đủ 2 chữ ký → UC-49 xử lý vi phạm.

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Ký kết — mục tiêu hoàn toàn khác với soạn thảo |
| Data scope riêng | ✅ Hợp đồng read-only + signature pad |
| Action set riêng | ✅ Vẽ/gõ chữ ký, xác nhận ký |
| Navigation boundary | ✅ Context switch rõ ràng: edit → sign |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Toàn bộ nội dung hợp đồng (read-only preview)
- Mã hợp đồng (contract_number)
- Trạng thái (status): CONFIRMED
- Trạng thái ký: organizer_signed, sponsor_signed
- Ngày xác nhận nội dung
- Đếm ngược 72 giờ ký kết (signing_deadline_at — hard_locked_at + 72h) `[ADDED — BP03]`
- Thời gian còn lại để ký (countdown timer) `[ADDED — BP03]`

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Hình thức ký | Radio | Bắt buộc | DRAW (vẽ tay) / TYPE (gõ tên) |
| Chữ ký vẽ tay | Signature pad (canvas) | Bắt buộc nếu DRAW | signature_data |
| Tên đầy đủ (gõ) | Text input | Bắt buộc nếu TYPE | typed_name |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Xác nhận ký** | Lưu chữ ký, kiểm tra 2 bên → SIGNED (UC-22 Main) | CONFIRMED + chưa ký |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Xóa chữ ký / Vẽ lại | Reset signature pad |
| Quay lại | SCR-012 (Contract Edit) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0505: Chữ ký chỉ khi CONFIRMED. Mỗi bên ký MỘT LẦN, không rút lại
- BR-0506: Contract → SIGNED khi cả hai đã ký
- BR-0510: Hợp đồng PHẢI hoàn tất 2 chữ ký trong 72 giờ kể từ hard-lock `[ADDED — BP03]`

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-012 (Contract Edit) | CTA "Ký hợp đồng" khi CONFIRMED |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại | SCR-012 |
| Cả hai ký → SIGNED | SCR-014 (Contract View) |
| Chỉ 1 bên ký | Ở lại SCR-013 (chờ đối tác) |
| Quá hạn 72h chưa đủ 2 chữ ký | Hiển thị nút "Tố cáo đối tác" → SCR-027 (Paywall — breach report) `[ADDED — BP03]` |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải hợp đồng |
| Not CONFIRMED Error | HĐ chưa xác nhận (UC-22 EF-22.1) |
| Already Signed State | Actor đã ký, chờ đối tác (UC-22 EF-22.2) |
| One Party Signed | 1 bên ký → thông báo đối tác (UC-22 AF-22.a) |
| Both Signed Success | Cả hai ký → thông báo + tạo nghĩa vụ tự động + đếm ngược dừng |
| Sign Success Toast | Ký thành công |
| Countdown 72h Active | Đếm ngược 72 giờ đang chạy `[ADDED — BP03]` |
| Countdown 72h Expired | Quá hạn 72h → hiển thị cảnh báo + nút tố cáo `[ADDED — BP03]` |
| Grace Period Active | 24h ân hạn sau tố cáo (UC-49) `[ADDED — BP03]` |

## UI Components liên quan

- Contract preview — nội dung hợp đồng read-only
- Signature method selector — radio: vẽ tay / gõ tên
- Signature pad — canvas vẽ chữ ký
- Text input — gõ tên đầy đủ
- Signature preview — hiển thị chữ ký trước khi xác nhận
- CTA button — "Xác nhận ký"
- Signing status — trạng thái ký của 2 bên
- **Countdown timer** — đếm ngược 72h hard-lock `[ADDED — BP03]`
- **Deadline warning banner** — cảnh báo khi gần hết hạn / đã hết hạn `[ADDED — BP03]`
- **Breach report CTA** — nút "Tố cáo đối tác" (conditional, khi quá 72h) `[ADDED — BP03]`

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-22 | Main-1 | Nhấn "Ký hợp đồng" | Screen entry | Page load |
| UC-22 | Main-2 | Hiển thị giao diện ký + countdown 72h | Screen content | Signature UI + timer |
| UC-22 | Main-3 | Vẽ/gõ chữ ký | Input | Signature pad / Text input |
| UC-22 | Main-4 | Nhấn "Xác nhận ký" | Action | CTA button |
| UC-22 | Main-5~6 | Lưu + kiểm tra 2 bên | Action | System processing |
| UC-22 | Main-7~9 | Cả hai ký → SIGNED + tạo obligations + dừng countdown | Action + State | Success + redirect |
| UC-22 | AF-22.a | Chỉ 1 bên ký + countdown tiếp tục | UI State | Waiting for partner + timer |
| UC-22 | EF-22.1 | Chưa CONFIRMED | UI State | Error block |
| UC-22 | EF-22.2 | Đã ký trước đó | UI State | "Đang chờ đối tác ký" |
| UC-49 | trigger | Quá hạn 72h | UI State | Countdown Expired → nút tố cáo `[ADDED — BP03]` |
