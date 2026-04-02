# SCR-011: User_DealNegotiation_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-011 |
| **Screen Name** | User_DealNegotiation_Screen |
| **Mục đích** | Authenticated User thực hiện toàn bộ hoạt động thương thảo trong một deal: trao đổi tin nhắn, đặt lịch họp, phản hồi lịch họp, ghi nhận kết quả cuộc họp, xác nhận đồng thuận ký kết, và hủy bỏ thương thảo |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 3 — Thương thảo hợp đồng tài trợ |
| **Use case liên quan** | UC-14, UC-15, UC-16, UC-17, UC-18, UC-19 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Thương thảo deal — hub cho tất cả hoạt động negotiation |
| Data scope riêng | ✅ Toàn bộ deal context: messages, meetings, agreement status |
| Action set riêng | ✅ Chat, schedule, confirm, terminate — phong phú |
| Navigation boundary | ✅ Context switch từ deal list |
| Independently testable | ✅ |

**Lý do gộp 6 UC vào 1 screen:** Tất cả hoạt động thuộc cùng deal context, cùng data scope (deal + 2 parties), và được thiết kế dạng hub với các panel/tab. Việc tách từng UC thành screen riêng sẽ gây over-fragmentation vì không có navigation boundary thực sự giữa chat, meeting management, và agreement trong cùng một phiên thương thảo.

---

## Dữ liệu hiển thị (Read-only Data)

### Header / Deal Overview

- Tên đối tác (partner_name)
- Tên sự kiện (event_name)
- Trạng thái deal (status): IN_PROGRESS / AGREED
- Trạng thái đồng thuận: Bên đã xác nhận (organizer_confirmed, sponsor_confirmed)

### Panel: Tin nhắn (UC-14)

- Lịch sử tin nhắn theo thứ tự thời gian
- Mỗi tin nhắn: nội dung, thời gian gửi (sent_at), người gửi, trạng thái (SENT/DELIVERED/READ)
- File đính kèm: tên file, dung lượng, nút tải về
- Trạng thái đã đọc/chưa đọc

### Panel: Cuộc họp (UC-15, UC-16, UC-17)

- Danh sách cuộc họp với trạng thái: PROPOSED / CONFIRMED / DECLINED / RESCHEDULED
- Mỗi cuộc họp: ngày giờ, thời lượng, chủ đề, ghi chú, hình thức (ONLINE/IN_PERSON), link/địa điểm
- Ghi chú kết quả cuộc họp (notebook) — nếu CONFIRMED đã diễn ra

---

## Dữ liệu nhập (Input Fields)

### Panel Tin nhắn (UC-14)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Nội dung tin nhắn | Textarea | Không rỗng | message_content |
| File đính kèm | File upload (multiple) | ≤ 5 files, ≤ 10MB/file, PDF/DOCX/XLSX/JPEG/PNG/WebP | attachments |

### Form Đặt lịch họp (UC-15)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Ngày giờ họp | Datetime picker | Bắt buộc, ≥ 1 giờ từ hiện tại | meeting_datetime |
| Thời lượng (phút) | Number input | Bắt buộc | duration_minutes |
| Chủ đề | Text input | Bắt buộc | meeting_subject |
| Ghi chú | Textarea | Tùy chọn | meeting_notes |
| Hình thức | Radio | Bắt buộc | ONLINE / IN_PERSON |
| Link/Địa điểm | Text input | Tùy chọn | meeting_location |

### Form Phản hồi lịch họp (UC-16)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Ghi chú phản hồi | Textarea | Tùy chọn | response_note |
| Ngày giờ mới (khi đề xuất lại) | Datetime picker | ≥ 1 giờ từ hiện tại | new_datetime |

### Form Ghi nhận kết quả (UC-17)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Tóm tắt nội dung | Textarea | Tùy chọn | meeting_summary |
| Quyết định đã thống nhất | Textarea / List | Tùy chọn | decisions |
| Action items | Textarea / List | Tùy chọn | action_items |

### Form Hủy thương thảo (UC-19)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Lý do hủy | Textarea | Bắt buộc, ≥ 10 ký tự | termination_reason |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Gửi tin nhắn** | Tin nhắn gửi real-time (UC-14 Main) | Deal IN_PROGRESS / AGREED |
| **Xác nhận đồng thuận** | Ghi nhận xác nhận (UC-18 Main) | Deal IN_PROGRESS |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Đặt lịch họp | Tạo meeting PROPOSED (UC-15) | Deal IN_PROGRESS |
| Chấp nhận đề xuất họp | Meeting → CONFIRMED (UC-16 Main) | Meeting PROPOSED + bên nhận |
| Từ chối đề xuất họp | Meeting → DECLINED (UC-16 AF-16.a) | Meeting PROPOSED + bên nhận |
| Đề xuất lại thời gian | Meeting gốc → RESCHEDULED, meeting mới PROPOSED (UC-16 AF-16.b) | Meeting PROPOSED + bên nhận |
| Ghi nhận kết quả họp | Lưu notebook (UC-17 Main) | Meeting CONFIRMED đã diễn ra |
| Rút lại xác nhận | Reset confirmed flag (UC-18 AF-18.b) | Đã xác nhận + đối tác chưa xác nhận |
| Hủy bỏ thương thảo | Confirm → TERMINATED (UC-19 Main) | Deal IN_PROGRESS |
| Đính kèm file | Upload file (UC-14 AF-14.a) | — |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0401: Chỉ 2 bên trong deal mới gửi tin nhắn
- BR-0402: Tin nhắn chỉ gửi khi deal IN_PROGRESS / AGREED
- BR-0403: File đính kèm: PDF/DOCX/XLSX/JPEG/PNG/WebP, ≤ 10MB/file, ≤ 5 file/tin nhắn
- BR-0404: Lịch họp ≥ 1 giờ từ hiện tại. Nhắc nhở 30 phút trước giờ CONFIRMED
- BR-0405: Deal → AGREED khi CẢ HAI xác nhận. Rút lại được trước khi đối tác xác nhận
- BR-0406: Lý do hủy bắt buộc ≥ 10 ký tự. Deal chỉ hủy khi IN_PROGRESS, AGREED không hủy được

---

## Quy tắc xác thực (Validation Rules)

| Trường | Quy tắc |
|--------|---------|
| message_content | Không rỗng |
| attachments | ≤ 5 files, ≤ 10MB/file, format hợp lệ |
| meeting_datetime | ≥ 1 giờ từ hiện tại |
| meeting_subject | Bắt buộc |
| termination_reason | ≥ 10 ký tự |

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-010 (Deal List) | Nhấn vào deal |
| SCR-009 (sau Accept invitation) | Auto-redirect sau tạo deal |
| In-app notification | Thông báo tin nhắn mới / meeting |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | SCR-010 (Deal List) |
| Deal AGREED + cả hai xác nhận | SCR-012 (Contract Edit) — link "Soạn thảo hợp đồng" |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải deal data |
| Message Sending State | Đang gửi tin nhắn |
| Message Delivered/Read | Trạng thái delivery (UC-14 Main-8) |
| Meeting Proposed Toast | Đề xuất họp thành công |
| Meeting Response Toast | Phản hồi lịch họp thành công |
| Agreement Status Banner | Hiển thị ai đã xác nhận (0/2, 1/2, 2/2) |
| Agreement Complete Banner | Cả hai xác nhận → "Sẵn sàng ký kết" + link SCR-012 |
| Terminate Confirm Dialog | Xác nhận hủy (UC-19 Main-5) |
| Deal Terminated State | Deal TERMINATED — read-only, hiển thị lý do |
| Deal AGREED Lock | Deal AGREED — không thể hủy (UC-19 EF-19.1) |
| File Upload Error | File không hợp lệ (UC-14 EF-14.2, EF-14.3) |
| Meeting Time Error | Thời gian < 1 giờ (UC-15 EF-15.1) |

## UI Components liên quan

- **Chat panel** — tin nhắn real-time với message bubbles
- **File attachment** — upload + preview + download
- **Meeting cards** — danh sách cuộc họp với hành động
- **Meeting form** — inline form hoặc modal đặt/phản hồi lịch
- **Notebook editor** — ghi nhận kết quả cuộc họp
- **Agreement progress** — hiển thị trạng thái xác nhận 2 bên
- **CTA buttons** — xác nhận đồng thuận, hủy thương thảo
- **Confirm dialog** — hủy thương thảo
- **Status badges** — meeting status, message delivery
- **Toast notifications** — thành công/lỗi

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-14 | Main-1 | Mở trang thương thảo | Screen entry | Page load |
| UC-14 | Main-2 | Hiển thị lịch sử tin nhắn | Read-only data | Chat panel |
| UC-14 | Main-3~4 | Nhập + gửi tin nhắn | Input + Action | Message input + Send button |
| UC-14 | Main-5~8 | Lưu + delivery + read status | Action + State | Message bubbles |
| UC-14 | AF-14.a | Đính kèm file | Component | File upload |
| UC-14 | EF-14.1 | Deal TERMINATED | UI State | Deal terminated state |
| UC-14 | EF-14.2~3 | File/số lượng không hợp lệ | UI State | Error message |
| UC-15 | Main-1~2 | Đặt lịch họp: mở form | Component | Meeting form |
| UC-15 | Main-3~5 | Nhập thông tin + gửi đề xuất | Input + Action | Form fields + Submit |
| UC-15 | Main-6~8 | Validate + tạo PROPOSED + thông báo | Action | System processing |
| UC-15 | EF-15.1 | Ngày giờ < 1 giờ | UI State | Validation error |
| UC-16 | Main-1~2 | Xem chi tiết đề xuất | Read-only data | Meeting card details |
| UC-16 | Main-3~7 | Chấp nhận → CONFIRMED | Action | Accept button |
| UC-16 | AF-16.a | Từ chối | Action | Decline button + notes |
| UC-16 | AF-16.b | Đề xuất lại thời gian | Action + Input | Reschedule form |
| UC-16 | EF-16.1 | Thời gian mới < 1 giờ | UI State | Validation error |
| UC-17 | Main-1~2 | Mở ghi nhận kết quả | Component | Notebook form on meeting card |
| UC-17 | Main-3~9 | Nhập tóm tắt + quyết định + actions | Input | Notebook fields |
| UC-17 | AF-17.a | Chỉnh sửa ghi chú đã lưu | Component | Edit notebook |
| UC-18 | Main-1 | Nhấn "Xác nhận đồng thuận" | Action | CTA button |
| UC-18 | Main-2~4 | Ghi nhận + thông báo | Action | System processing |
| UC-18 | Main-5~8 | Cả hai xác nhận → AGREED | Action + State | Agreement complete banner |
| UC-18 | AF-18.a | Chỉ 1 bên xác nhận | UI State | Partial agreement (1/2) |
| UC-18 | AF-18.b | Rút lại xác nhận | Action | "Rút lại" button |
| UC-19 | Main-1~3 | Nhấn "Hủy" + nhập lý do | Action + Input | CTA + form |
| UC-19 | Main-4~6 | Validate + xác nhận | Action + State | Confirm dialog |
| UC-19 | Main-7~9 | TERMINATED + thông báo | Action + State | Deal terminated state |
| UC-19 | AF-19.a | Hủy thao tác | UI State | Dialog dismissed |
| UC-19 | EF-19.1 | Deal AGREED | UI State | Error: không thể hủy |
| UC-19 | EF-19.2 | Lý do < 10 ký tự | UI State | Validation error |
