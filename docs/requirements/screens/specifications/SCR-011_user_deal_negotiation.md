# SCR-011: User_DealNegotiation_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-011 |
| **Screen Name** | User_DealNegotiation_Screen |
| **Mục đích** | Authenticated User thực hiện toàn bộ hoạt động thương thảo trong một deal: trao đổi tin nhắn (với Data Masking), đặt lịch họp, phản hồi lịch họp, ghi nhận kết quả cuộc họp, tạo thỏa thuận nháp, xem trước phí dịch vụ, xác nhận đồng thuận ký kết, và hủy bỏ thương thảo |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 3 — Thương thảo hợp đồng tài trợ |
| **Use case liên quan** | UC-14, UC-15, UC-16, UC-17, UC-18, UC-19, UC-51, UC-56 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Thương thảo deal — hub cho tất cả hoạt động negotiation |
| Data scope riêng | ✅ Toàn bộ deal context: messages, meetings, agreement status, draft agreement |
| Action set riêng | ✅ Chat, schedule, confirm, terminate, draft agreement, fee preview — phong phú |
| Navigation boundary | ✅ Context switch từ deal list |
| Independently testable | ✅ |

**Lý do gộp 8 UC vào 1 screen:** Tất cả hoạt động thuộc cùng deal context, cùng data scope (deal + 2 parties), và được thiết kế dạng hub với các panel/tab. Việc tách từng UC thành screen riêng sẽ gây over-fragmentation vì không có navigation boundary thực sự giữa chat, meeting management, draft agreement, và agreement trong cùng một phiên thương thảo. UC-51 (fee preview) là modal read-only, UC-56 (draft agreement) là component cùng context.

---

## Dữ liệu hiển thị (Read-only Data)

### Header / Deal Overview

- Tên đối tác (partner_name)
- Tên sự kiện (event_name)
- Trạng thái deal (status): IN_PROGRESS / AWAITING_PAYMENT / AGREED
- Trạng thái đồng thuận: Bên đã xác nhận (organizer_confirmed, sponsor_confirmed)
- Trạng thái thỏa thuận nháp: PENDING_CONFIRMATION / CONFIRMED / REJECTED `[ADDED — BP03]`

### Panel: Tin nhắn (UC-14)

- Lịch sử tin nhắn theo thứ tự thời gian
- Mỗi tin nhắn: nội dung, thời gian gửi (sent_at), người gửi, trạng thái (SENT/DELIVERED/READ)
- **[UPDATED — BP03]** Nếu `deal.contact_unlocked = false`: nội dung hiển thị đã masking (SĐT, email, URL MXH bị che `***`)
- **[ADDED — BP03]** Tin nhắn bị FLAGGED (bypass detected): hiển thị cảnh báo cho người gửi, không hiển thị cho đối tác
- File đính kèm: tên file, dung lượng, nút tải về
- Trạng thái đã đọc/chưa đọc

### Panel: Cuộc họp (UC-15, UC-16, UC-17)

- Danh sách cuộc họp với trạng thái: PROPOSED / CONFIRMED / DECLINED / RESCHEDULED
- Mỗi cuộc họp: ngày giờ, thời lượng, chủ đề, ghi chú, hình thức (ONLINE/IN_PERSON), link/địa điểm
- Ghi chú kết quả cuộc họp (notebook) — nếu CONFIRMED đã diễn ra

### Panel: Thỏa thuận nháp (UC-56) `[ADDED — BP03]`

- Thỏa thuận nháp hiện tại (nếu có): hình thức tài trợ, giá trị, mô tả hiện vật, điều khoản chính
- Trạng thái: PENDING_CONFIRMATION / CONFIRMED / REJECTED
- Người tạo, ngày tạo
- Ghi chú từ chối (nếu REJECTED)
- Người xác nhận, ngày xác nhận (nếu CONFIRMED)

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

### Form Thỏa thuận nháp (UC-56) `[ADDED — BP03]`

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Hình thức tài trợ | Select | Bắt buộc | CASH / IN_KIND / COMBINED |
| Giá trị tiền mặt | Number input | Bắt buộc nếu CASH/COMBINED, > 0 | sponsorship_value |
| Mô tả hiện vật | Textarea | Bắt buộc nếu IN_KIND/COMBINED | in_kind_description |
| Điều khoản chính | Textarea | Bắt buộc | key_terms |
| Ghi chú từ chối (đối tác) | Textarea | Bắt buộc khi từ chối | rejection_note |

### Xem trước phí dịch vụ — Modal (UC-51) `[ADDED — BP03]`

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Hình thức tài trợ | Select | Bắt buộc | CASH / IN_KIND / COMBINED |
| Giá trị tiền mặt (ước tính) | Number input | Bắt buộc nếu CASH/COMBINED | estimated_value |

### Miễn trừ trách nhiệm hiện vật (UC-18 AF-18.c) `[ADDED — BP03]`

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Miễn trừ hiện vật | Checkbox | Bắt buộc nếu IN_KIND/COMBINED | in_kind_disclaimer |

### Form Hủy thương thảo (UC-19)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Lý do hủy | Textarea | Bắt buộc, ≥ 10 ký tự | termination_reason |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Gửi tin nhắn** | Tin nhắn gửi real-time, áp dụng Data Masking + anti-bypass (UC-14 Main) | Deal IN_PROGRESS / AWAITING_PAYMENT / AGREED |
| **Xác nhận đồng thuận** | Kiểm tra DraftAgreement → miễn trừ → ghi nhận → tính phí → AWAITING_PAYMENT (UC-18 Main) → redirect SCR-027 | Deal IN_PROGRESS + DraftAgreement CONFIRMED |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Đặt lịch họp | Tạo meeting PROPOSED (UC-15) | Deal IN_PROGRESS |
| Chấp nhận đề xuất họp | Meeting → CONFIRMED (UC-16 Main) | Meeting PROPOSED + bên nhận |
| Từ chối đề xuất họp | Meeting → DECLINED (UC-16 AF-16.a) | Meeting PROPOSED + bên nhận |
| Đề xuất lại thời gian | Meeting gốc → RESCHEDULED, meeting mới PROPOSED (UC-16 AF-16.b) | Meeting PROPOSED + bên nhận |
| Ghi nhận kết quả họp | Lưu notebook (UC-17 Main) | Meeting CONFIRMED đã diễn ra |
| Tạo thỏa thuận nháp | Tạo DraftAgreement PENDING_CONFIRMATION (UC-56 Main) `[ADDED — BP03]` | Deal IN_PROGRESS + không có DA pending/confirmed |
| Xác nhận thỏa thuận nháp | DA → CONFIRMED (UC-56 Main-8~12) `[ADDED — BP03]` | DA PENDING + đối tác |
| Từ chối thỏa thuận nháp | DA → REJECTED (UC-56 AF-56.a) `[ADDED — BP03]` | DA PENDING + đối tác |
| Xem trước phí dịch vụ | Mở modal ước tính phí (UC-51 Main) `[ADDED — BP03]` | Deal IN_PROGRESS |
| Rút lại xác nhận | Reset confirmed flag (UC-18 AF-18.b) | Đã xác nhận + đối tác chưa xác nhận |
| Hủy bỏ thương thảo | Confirm → TERMINATED (UC-19 Main) | Deal IN_PROGRESS |
| Đính kèm file | Upload file (UC-14 AF-14.a) | — |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0401: Chỉ 2 bên trong deal mới gửi tin nhắn
- BR-0402: Tin nhắn chỉ gửi khi deal IN_PROGRESS / AWAITING_PAYMENT / AGREED `[UPDATED — BP03]`
- BR-0403: File đính kèm: PDF/DOCX/XLSX/JPEG/PNG/WebP, ≤ 10MB/file, ≤ 5 file/tin nhắn
- BR-0404: Lịch họp ≥ 1 giờ từ hiện tại. Nhắc nhở 30 phút trước giờ CONFIRMED
- BR-0405: Deal → AWAITING_PAYMENT khi CẢ HAI xác nhận. Rút lại được trước khi đối tác xác nhận `[UPDATED — BP03]`
- BR-0406: Lý do hủy bắt buộc ≥ 10 ký tự. Deal chỉ hủy khi IN_PROGRESS, AWAITING_PAYMENT/AGREED không hủy được `[UPDATED — BP03]`
- BR-0407: Thỏa thuận nháp BẮT BUỘC trước khi đồng thuận. Mỗi deal 1 CONFIRMED tại 1 thời điểm `[ADDED — BP03]`
- BR-1201: Phí dịch vụ tính tự động từ sponsorship_type + sponsorship_value `[ADDED — BP03]`
- BR-1301: Data Masking áp dụng cho SĐT, email, URL MXH, số TK khi contact_unlocked = false `[ADDED — BP03]`
- BR-1303: Zero Tolerance — vi phạm bypass: 1-2 cảnh báo, 3 khóa tạm tính năng nhắn tin `[ADDED — BP03]`
- BR-1304: Miễn trừ trách nhiệm hiện vật BẮT BUỘC cho deal có IN_KIND/COMBINED `[ADDED — BP03]`

---

## Quy tắc xác thực (Validation Rules)

| Trường | Quy tắc |
|--------|---------| 
| message_content | Không rỗng |
| attachments | ≤ 5 files, ≤ 10MB/file, format hợp lệ |
| meeting_datetime | ≥ 1 giờ từ hiện tại |
| meeting_subject | Bắt buộc |
| termination_reason | ≥ 10 ký tự |
| sponsorship_value | > 0, bắt buộc nếu CASH/COMBINED `[ADDED — BP03]` |
| in_kind_description | Bắt buộc nếu IN_KIND/COMBINED `[ADDED — BP03]` |
| key_terms | Bắt buộc `[ADDED — BP03]` |
| rejection_note | Bắt buộc khi từ chối thỏa thuận `[ADDED — BP03]` |
| in_kind_disclaimer | Bắt buộc checkbox khi IN_KIND/COMBINED `[ADDED — BP03]` |

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-010 (Deal List) | Nhấn vào deal IN_PROGRESS / AGREED |
| SCR-009 (sau Accept invitation) | Auto-redirect sau tạo deal |
| In-app notification | Thông báo tin nhắn mới / meeting |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | SCR-010 (Deal List) |
| Deal AWAITING_PAYMENT (2/2 đồng thuận) | SCR-027 (Service Fee Paywall) — redirect `[UPDATED — BP03]` |
| Deal AGREED + hợp đồng sẵn sàng | SCR-012 (Contract Edit) — link "Soạn thảo hợp đồng" |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải deal data |
| Message Sending State | Đang gửi tin nhắn |
| Message Delivered/Read | Trạng thái delivery (UC-14 Main-9) |
| Masked Message Display | `contact_unlocked = false` — tin nhắn hiển thị nội dung đã masking `[ADDED — BP03]` |
| Unmasked Message Display | `contact_unlocked = true` — nội dung gốc hiển thị đầy đủ `[ADDED — BP03]` |
| Bypass Warning Toast | Phát hiện hành vi lách bộ lọc — tin nhắn bị chặn (UC-14 EF-14.4) `[ADDED — BP03]` |
| Flagged Message State | Tin nhắn bị FLAGGED — hiển thị cho người gửi, không hiển thị cho đối tác `[ADDED — BP03]` |
| Chat Locked State | Vi phạm ≥ 3 → tạm khóa tính năng nhắn tin trong deal (UC-14 EF-14.4) `[ADDED — BP03]` |
| Meeting Proposed Toast | Đề xuất họp thành công |
| Meeting Response Toast | Phản hồi lịch họp thành công |
| Draft Agreement Form | Form tạo thỏa thuận nháp (UC-56 Main-2~3) `[ADDED — BP03]` |
| Draft Agreement Pending | Thỏa thuận đang chờ đối tác xác nhận (UC-56 Main-5~6) `[ADDED — BP03]` |
| Draft Agreement Confirmed | Thỏa thuận đã xác nhận → mở khóa CTA đồng thuận (UC-56 Main-10) `[ADDED — BP03]` |
| Draft Agreement Rejected | Thỏa thuận bị từ chối + ghi chú lý do → có thể tạo lại (UC-56 AF-56.a) `[ADDED — BP03]` |
| Draft Agreement Already Exists Error | Đã có DA PENDING_CONFIRMATION (UC-56 EF-56.1) `[ADDED — BP03]` |
| Fee Preview Modal | Modal ước tính phí dịch vụ (UC-51 Main) `[ADDED — BP03]` |
| Fee Preview In-kind | Phí cố định cho hiện vật hoàn toàn (UC-51 AF-51.a) `[ADDED — BP03]` |
| In-kind Disclaimer Checkbox | Checkbox miễn trừ trách nhiệm hiện vật (UC-18 AF-18.c) `[ADDED — BP03]` |
| In-kind Disclaimer Required Error | Chưa tick miễn trừ (UC-18 EF-18.2) `[ADDED — BP03]` |
| No Draft Agreement Error | Chưa có DraftAgreement CONFIRMED khi xác nhận đồng thuận (UC-18 EF-18.1) `[ADDED — BP03]` |
| Agreement Status Banner | Hiển thị ai đã xác nhận (0/2, 1/2, 2/2) |
| Agreement to Paywall Redirect | Cả hai xác nhận → chuyển sang AWAITING_PAYMENT → redirect SCR-027 `[UPDATED — BP03]` |
| Terminate Confirm Dialog | Xác nhận hủy (UC-19 Main-5) |
| Deal Terminated State | Deal TERMINATED — read-only, hiển thị lý do |
| Deal AWAITING_PAYMENT Lock | Deal AWAITING_PAYMENT — không thể hủy (BR-0406) `[UPDATED — BP03]` |
| File Upload Error | File không hợp lệ (UC-14 EF-14.2, EF-14.3) |
| Meeting Time Error | Thời gian < 1 giờ (UC-15 EF-15.1) |

## UI Components liên quan

- **Chat panel** — tin nhắn real-time với message bubbles
- **Masked message indicator** — badge "Thông tin bị che giấu" trên tin nhắn `[ADDED — BP03]`
- **File attachment** — upload + preview + download
- **Meeting cards** — danh sách cuộc họp với hành động
- **Meeting form** — inline form hoặc modal đặt/phản hồi lịch
- **Notebook editor** — ghi nhận kết quả cuộc họp
- **Draft agreement form** — form tạo thỏa thuận nháp (sponsorship_type, value, key_terms) `[ADDED — BP03]`
- **Draft agreement card** — hiển thị thỏa thuận + hành động xác nhận/từ chối `[ADDED — BP03]`
- **Fee preview modal** — modal ước tính phí dịch vụ `[ADDED — BP03]`
- **In-kind disclaimer checkbox** — checkbox miễn trừ trách nhiệm hiện vật `[ADDED — BP03]`
- **Agreement progress** — hiển thị trạng thái xác nhận 2 bên
- **CTA buttons** — xác nhận đồng thuận, tạo thỏa thuận nháp, hủy thương thảo
- **Confirm dialog** — hủy thương thảo
- **Status badges** — meeting status, message delivery, draft agreement status
- **Toast notifications** — thành công/lỗi/cảnh báo bypass

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-14 | Main-1 | Mở trang thương thảo | Screen entry | Page load |
| UC-14 | Main-2 | Hiển thị lịch sử tin nhắn (masking nếu chưa mở khóa) | Read-only data | Chat panel |
| UC-14 | Main-3~4 | Nhập + gửi tin nhắn | Input + Action | Message input + Send button |
| UC-14 | Main-5 | Lưu nội dung gốc, áp dụng masking khi hiển thị | Action | System processing |
| UC-14 | Main-6 | Anti-bypass detection song song | Action | System processing |
| UC-14 | Main-7~9 | Hiển thị tin nhắn (đã masking nếu cần) + delivery status | Action + State | Message bubbles |
| UC-14 | Main-10 | Use case kết thúc | Action | — |
| UC-14 | AF-14.a | Đính kèm file | Component | File upload |
| UC-14 | EF-14.1 | Deal TERMINATED | UI State | Deal terminated state |
| UC-14 | EF-14.2~3 | File/số lượng không hợp lệ | UI State | Error message |
| UC-14 | EF-14.4 | Bypass detected → chặn tin nhắn | UI State | Bypass warning + flagged |
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
| UC-56 | Main-1 | Nhấn "Tạo thỏa thuận nháp" | Action | CTA button |
| UC-56 | Main-2~3 | Điền form thỏa thuận nháp | Input | Draft agreement form |
| UC-56 | Main-4~6 | Validate + tạo PENDING + thông báo | Action | System processing |
| UC-56 | Main-7~8 | Đối tác xem + xác nhận | Action | Confirm button on DA card |
| UC-56 | Main-9~12 | DA → CONFIRMED, mở khóa CTA đồng thuận | Action + State | DA confirmed state |
| UC-56 | AF-56.a | Đối tác từ chối + ghi chú | Action | Reject button + notes |
| UC-56 | AF-56.b | Tài trợ COMBINED | Input | Both value + description fields |
| UC-56 | EF-56.1 | Đã có DA pending | UI State | Error toast |
| UC-56 | EF-56.2 | Thiếu giá trị CASH | UI State | Validation error |
| UC-56 | EF-56.3 | Thiếu mô tả IN_KIND | UI State | Validation error |
| UC-51 | Main-1 | Nhấn "Xem trước phí dịch vụ" | Action | CTA → modal |
| UC-51 | Main-2~3 | Nhập hình thức + giá trị | Input | Modal form |
| UC-51 | Main-4~6 | Tính + hiển thị ước tính | Read-only data | Modal result |
| UC-51 | AF-51.a | IN_KIND toàn bộ → phí cố định | Read-only data | Modal fixed fee |
| UC-51 | EF-51.1 | Thiếu giá trị CASH/COMBINED | UI State | Validation error |
| UC-18 | Main-1 | Nhấn "Xác nhận đồng thuận" | Action | CTA button |
| UC-18 | Main-2 | Kiểm tra DraftAgreement CONFIRMED | Action | System gate check |
| UC-18 | Main-3 | Kiểm tra IN_KIND → miễn trừ | Action + Component | Disclaimer checkbox |
| UC-18 | Main-4~5 | Ghi nhận xác nhận + thông báo | Action | System processing |
| UC-18 | Main-6 | Kiểm tra 2/2 xác nhận | Action | System check |
| UC-18 | Main-7~10 | Tính phí + tạo Paywall + AWAITING_PAYMENT | Action + State | Redirect to SCR-027 |
| UC-18 | AF-18.a | Chỉ 1 bên xác nhận | UI State | Partial agreement (1/2) |
| UC-18 | AF-18.b | Rút lại xác nhận | Action | "Rút lại" button |
| UC-18 | AF-18.c | Miễn trừ hiện vật | Component | Disclaimer checkbox |
| UC-18 | EF-18.1 | Chưa có DA CONFIRMED | UI State | Error: cần tạo thỏa thuận |
| UC-18 | EF-18.2 | Chưa tick miễn trừ | UI State | Error: cần tick checkbox |
| UC-19 | Main-1~3 | Nhấn "Hủy" + nhập lý do | Action + Input | CTA + form |
| UC-19 | Main-4~6 | Validate + xác nhận | Action + State | Confirm dialog |
| UC-19 | Main-7~9 | TERMINATED + thông báo | Action + State | Deal terminated state |
| UC-19 | AF-19.a | Hủy thao tác | UI State | Dialog dismissed |
| UC-19 | EF-19.1 | Deal AWAITING_PAYMENT/AGREED | UI State | Error: không thể hủy |
| UC-19 | EF-19.2 | Lý do < 10 ký tự | UI State | Validation error |
