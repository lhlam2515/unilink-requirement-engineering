# SCR-009: User_InvitationDetail_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-009 |
| **Screen Name** | User_InvitationDetail_Screen |
| **Mục đích** | Authenticated User xem chi tiết lời mời tài trợ, và nếu là bên nhận với lời mời PENDING — thực hiện chấp nhận hoặc từ chối |
| **Actor chính** | Authenticated User (Bên nhận — Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 2.3 — Gửi lời mời tài trợ |
| **Use case liên quan** | UC-12, UC-13 (AF-13.a) |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Xem chi tiết + quyết định chấp nhận/từ chối — detail context |
| Data scope riêng | ✅ Toàn bộ thông tin 1 lời mời cụ thể |
| Action set riêng | ✅ Accept / Decline — khác hoàn toàn với list view |
| Navigation boundary | ✅ Context switch từ SCR-008 |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Trạng thái lời mời (status): PENDING / ACCEPTED / DECLINED / EXPIRED
- **Thông tin bên gửi**: Tên (partner_name), vai trò
- **Thông tin sự kiện**: Tên, loại hình, thời gian, địa điểm (summary)
- Tin nhắn giới thiệu (invitation_message)
- Gói tài trợ ưu tiên (preferred_package) — nếu có
- Ngày gửi (sent_at)
- Ngày phản hồi (responded_at) — nếu đã phản hồi
- Tin nhắn phản hồi (response_message) — nếu có
- Lý do từ chối (decline_reason) — nếu DECLINED

---

## Dữ liệu nhập (Input Fields)

Chỉ hiển thị khi lời mời PENDING và actor là bên nhận:

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Tin nhắn phản hồi (khi Accept) | Textarea | Tùy chọn | response_message |
| Lý do từ chối (khi Decline) | Textarea | Tùy chọn | decline_reason |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Chấp nhận lời mời** | Invitation → ACCEPTED, tạo Deal IN_PROGRESS (UC-12 Main) | PENDING + bên nhận |
| **Từ chối lời mời** | Invitation → DECLINED (UC-12 AF-12.a) | PENDING + bên nhận |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Quay lại | SCR-008 (Invitation List) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0305: Chỉ lời mời PENDING mới accept/decline. ACCEPTED/DECLINED/EXPIRED không thể thay đổi
- BR-0306: Lời mời PENDING tự động hết hạn sau 14 ngày

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-008 (Invitation List) | Nhấn vào lời mời trong danh sách |
| In-app notification | Nhấn vào thông báo mời mới |
| Email link | Link "Xem chi tiết lời mời" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | SCR-008 (Invitation List) |
| Chấp nhận thành công | SCR-011 (Deal Negotiation) hoặc ở lại SCR-009 |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải chi tiết |
| Pending — Recipient View | PENDING + actor là bên nhận: hiển thị Accept/Decline buttons |
| Pending — Sender View | PENDING + actor là bên gửi: chỉ hiển thị "Đang chờ phản hồi" |
| Accepted State | ACCEPTED: hiển thị thông tin deal link |
| Declined State | DECLINED: hiển thị lý do (nếu có) |
| Expired State | EXPIRED: hiển thị "Lời mời đã hết hạn" (UC-12 EF-12.1) |
| Already Processed Error | Lời mời đã xử lý (UC-12 EF-12.2) |
| Accept Success Toast | Chấp nhận thành công |
| Decline Confirm | Xác nhận từ chối + form lý do |

## UI Components liên quan

- Status badge — PENDING / ACCEPTED / DECLINED / EXPIRED
- Info card — thông tin bên gửi + sự kiện
- Message display — tin nhắn giới thiệu
- Action buttons — Accept / Decline (conditional)
- Textarea — lý do từ chối / tin nhắn phản hồi
- Confirm dialog — xác nhận từ chối
- Toast notification

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-13 | AF-13.a Step 5a~5b | Nhấn vào lời mời → xem chi tiết | Screen entry | Page load |
| UC-13 | AF-13.a Step 5c | Hiển thị Accept/Decline nếu PENDING + bên nhận | Component | Action buttons |
| UC-12 | Main-1 | Mở chi tiết lời mời PENDING | Screen entry | Page load |
| UC-12 | Main-2 | Hiển thị thông tin lời mời | Read-only data | Info sections |
| UC-12 | Main-3 | Nhấn "Chấp nhận" | Action | Accept button |
| UC-12 | Main-4 | Nhập tin nhắn phản hồi | Input | Response textarea |
| UC-12 | Main-5~7 | Chuyển ACCEPTED → tạo Deal | Action | System processing |
| UC-12 | AF-12.a Step 3a~3g | Từ chối: form lý do + xác nhận | Action + Input | Decline flow |
| UC-12 | EF-12.1 | Lời mời hết hạn | UI State | Expired state |
| UC-12 | EF-12.2 | Lời mời đã xử lý | UI State | Error message |
