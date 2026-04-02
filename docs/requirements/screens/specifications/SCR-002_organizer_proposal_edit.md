# SCR-002: Organizer_ProposalEdit_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-002 |
| **Screen Name** | Organizer_ProposalEdit_Screen |
| **Mục đích** | Organizer soạn thảo toàn bộ nội dung hồ sơ tài trợ sự kiện — bao gồm thông tin cơ bản, hình ảnh, thông tin chi tiết, hình thức tài trợ, quản lý gói tài trợ & quyền lợi, và thực hiện phát hành/hủy phát hành |
| **Actor chính** | Organizer (BTC) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 1 — Tạo hồ sơ tài trợ sự kiện |
| **Use case liên quan** | UC-02, UC-03, UC-04, UC-05 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Soạn thảo/chỉnh sửa hồ sơ — mục tiêu edit mode |
| Data scope riêng | ✅ Toàn bộ fields của 1 proposal cụ thể |
| Action set riêng | ✅ Nhập liệu, upload ảnh, CRUD gói tài trợ, publish/unpublish |
| Navigation boundary | ✅ Context switch từ SCR-001 (list → detail/edit) |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Mã hồ sơ (proposal_id)
- Trạng thái hiện tại (status): `DRAFT` / `PUBLISHED`
- Ngày tạo (created_at)
- Ngày cập nhật gần nhất (updated_at)
- Ngày phát hành (published_at) — nếu PUBLISHED
- Tổ chức BTC sở hữu (organization_name)

---

## Dữ liệu nhập (Input Fields)

### Tab/Section 1: Thông tin cơ bản (UC-02)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Tên chương trình | Text input | Bắt buộc, không rỗng | event_name |
| Loại hình sự kiện | Select | Bắt buộc | Cuộc thi, Workshop, Đêm nhạc,... |
| Thời gian bắt đầu | Datetime picker | Bắt buộc, phải > hiện tại | event_date_start |
| Thời gian kết thúc | Datetime picker | Bắt buộc, phải > bắt đầu | event_date_end |
| Địa điểm tổ chức | Text input | Bắt buộc | event_location |

### Tab/Section 2: Hình ảnh nhận diện (UC-02 AF-02.a)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Banner sự kiện | File upload | JPEG/PNG/WebP, ≤ 5MB | Tùy chọn |
| Thumbnail | File upload | JPEG/PNG/WebP, ≤ 5MB | Auto-generate từ banner |

### Tab/Section 3: Thông tin chi tiết (UC-02)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Quy mô dự kiến | Number input | Bắt buộc, số dương | expected_scale |
| Ngân sách dự kiến | Number input | Bắt buộc, số dương (VND) | expected_budget |
| Đối tượng khán giả | Text/Tags input | Bắt buộc | target_audience |
| Nội dung chương trình | Rich text / Textarea | Tùy chọn | event_description |

### Tab/Section 4: Hình thức tài trợ (UC-02)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Hình thức tài trợ | Radio/Select | Bắt buộc khi phát hành | CASH / IN_KIND / COMBINED |

### Tab/Section 5: Gói tài trợ & Quyền lợi (UC-03)

| Trường gói | Loại | Validation | Ghi chú |
|------------|------|------------|---------|
| Tên gói | Text input | Bắt buộc, duy nhất trong hồ sơ | package_name |
| Cấp độ | Select | Bắt buộc | Title/Co-Sponsor/Associate/Technical/Partner |
| Mô tả gói | Textarea | Tùy chọn | package_description |
| Giá trị tối thiểu | Number input | Bắt buộc, > 0 | min_value (VND) |
| Số slot khả dụng | Number input | Bắt buộc, ≥ 1 | available_slots |

| Trường quyền lợi | Loại | Validation | Ghi chú |
|-------------------|------|------------|---------|
| Nhóm quyền lợi | Select | Bắt buộc | BRANDING / STAGE / DIGITAL / ENGAGEMENT |
| Tiêu đề | Text input | Bắt buộc | benefit_title |
| Mô tả cam kết | Textarea | Bắt buộc | commitment_description |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Lưu thay đổi** | System lưu nội dung, cập nhật updated_at (UC-02 Main-9~12) |
| **Phát hành hồ sơ** | System validate đầy đủ → chuyển DRAFT → PUBLISHED (UC-04 Main Flow) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Hủy phát hành | Confirm dialog → PUBLISHED → DRAFT (UC-05 Main Flow) |
| Thêm gói tài trợ | Mở form inline/section để thêm gói mới (UC-03 Main Flow) |
| Chỉnh sửa gói tài trợ | Hiển thị form chỉnh sửa gói hiện có (UC-03 AF-03.a) |
| Xóa gói tài trợ | Confirm dialog → xóa gói và quyền lợi (UC-03 AF-03.b) |
| Thêm quyền lợi | Mở form thêm quyền lợi cho gói (UC-03 Main-6~9) |
| Chỉnh sửa/Xóa quyền lợi | Inline edit/confirm delete (UC-03 AF-03.c) |
| Upload hình ảnh | File picker + preview (UC-02 AF-02.a) |
| Quay lại danh sách | Chuyển đến SCR-001 |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0103: Quy mô, ngân sách phải là số dương. Thời gian tổ chức phải trong tương lai
- BR-0104: File hình ảnh: JPEG/PNG/WebP, tối đa 5MB
- BR-0105: Hồ sơ phải có ít nhất 1 hình thức tài trợ trước khi phát hành
- BR-0106: Mỗi gói phải có tên duy nhất, giá trị > 0, slot ≥ 1. Hồ sơ phải có ≥ 1 gói trước phát hành
- BR-0107: Mỗi quyền lợi thuộc nhóm BRANDING/STAGE/DIGITAL/ENGAGEMENT, phải có mô tả cam kết
- BR-0108: Phát hành yêu cầu đầy đủ: tên, loại hình, thời gian, địa điểm, quy mô, ngân sách, đối tượng, hình thức, gói tài trợ có quyền lợi
- BR-0109: Không thể hủy phát hành nếu đã có Deal liên kết
- BR-0110: Khi hủy phát hành → thông báo doanh nghiệp đã bookmark + bên gửi lời mời PENDING

---

## Quy tắc xác thực (Validation Rules)

| Trường | Quy tắc |
|--------|---------|
| event_name | Không rỗng |
| event_date_start | Phải > ngày hiện tại |
| event_date_end | Phải > event_date_start |
| expected_scale | Số dương (> 0) |
| expected_budget | Số dương (> 0) |
| Image files | JPEG/PNG/WebP, ≤ 5MB |
| package_name | Duy nhất trong hồ sơ |
| min_value | > 0 |
| available_slots | ≥ 1 |
| Phát hành | Tất cả trường bắt buộc đã nhập + ≥ 1 gói + mỗi gói ≥ 1 quyền lợi |

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-001 (Proposal List) | Nhấn "Tạo hồ sơ mới" hoặc nhấn vào hồ sơ hiện có |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn "Quay lại" / Breadcrumb | SCR-001 (Proposal List) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải nội dung hồ sơ |
| Save Success Toast | Lưu thành công (UC-02 Main-10) |
| Publish Success Toast | Phát hành thành công (UC-04 Main-7) |
| Unpublish Confirm Dialog | Organizer nhấn "Hủy phát hành" (UC-05 Main-4) |
| Unpublish Warning Dialog | Hồ sơ có lời mời PENDING (UC-05 AF-05.b Step 3b) |
| Validation Error Inline | Trường bắt buộc thiếu (UC-02 EF-02.1, EF-02.2) |
| Publish Validation Error | Hồ sơ thiếu trường khi phát hành (UC-04 EF-04.1, EF-04.2, EF-04.3) |
| Image Upload Error | File không hợp lệ (UC-02 EF-02.3) |
| Delete Package Confirm | Xác nhận xóa gói tài trợ (UC-03 AF-03.b) |
| System Error | Lỗi hệ thống (UC-01 EF-01.1) |
| Cannot Unpublish Error | Hồ sơ có Deal liên kết (UC-05 EF-05.1) |

## UI Components liên quan

- Tab/Section navigation — chuyển giữa các mục nội dung
- Rich text editor — nội dung chương trình
- File upload with preview — hình ảnh nhận diện
- Inline form — thêm/sửa gói tài trợ
- Nested list — danh sách quyền lợi trong mỗi gói
- Status badge — DRAFT / PUBLISHED
- Confirm dialog — hủy phát hành, xóa gói
- Toast notification — lưu/phát hành thành công
- Validation error messages — inline errors

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-02 | Main-1 | Mở trang chỉnh sửa hồ sơ | Screen entry | Page load |
| UC-02 | Main-2 | Hiển thị form chỉnh sửa | Screen content | Tab/section layout |
| UC-02 | Main-3~4 | Nhập thông tin cơ bản + validate | Input + Validation | Section "Thông tin cơ bản" |
| UC-02 | Main-5~6 | Nhập thông tin chi tiết + validate | Input + Validation | Section "Thông tin chi tiết" |
| UC-02 | Main-7~8 | Chọn hình thức tài trợ | Input | Section "Hình thức tài trợ" |
| UC-02 | Main-9~12 | Lưu nội dung | Action + State | CTA "Lưu thay đổi" + Success toast |
| UC-02 | AF-02.a | Upload hình ảnh nhận diện | Component | File upload with preview |
| UC-02 | AF-02.b | Cập nhật một phần nội dung | Action | Partial save |
| UC-02 | EF-02.1 | Thời gian trong quá khứ | UI State | Inline validation error |
| UC-02 | EF-02.2 | Giá trị số không hợp lệ | UI State | Inline validation error |
| UC-02 | EF-02.3 | File không hợp lệ | UI State | Upload error message |
| UC-03 | Main-1~5 | Tạo gói tài trợ mới | Component | Inline form trong Section "Gói tài trợ" |
| UC-03 | Main-6~10 | Thêm quyền lợi cho gói | Component | Nested form |
| UC-03 | AF-03.a | Chỉnh sửa gói hiện có | Component | Edit form |
| UC-03 | AF-03.b | Xóa gói tài trợ | UI State | Confirm dialog |
| UC-03 | AF-03.c | Sửa/xóa quyền lợi | Component | Inline edit/delete |
| UC-03 | EF-03.1 | Tên gói trùng lặp | UI State | Inline validation error |
| UC-03 | EF-03.2 | Giá trị/slot không hợp lệ | UI State | Inline validation error |
| UC-04 | Main-1 | Nhấn "Phát hành hồ sơ" | Action | CTA button |
| UC-04 | Main-2~6 | Validate + chuyển trạng thái | Action | System processing |
| UC-04 | Main-7 | Thông báo thành công | UI State | Success toast |
| UC-04 | EF-04.1 | Thiếu trường bắt buộc | UI State | Validation error list |
| UC-04 | EF-04.2 | Gói chưa có quyền lợi | UI State | Warning |
| UC-04 | EF-04.3 | Chưa có gói tài trợ | UI State | Validation error |
| UC-05 | Main-1~5 | Nhấn "Hủy phát hành" + xác nhận | Action + State | CTA + Confirm dialog |
| UC-05 | Main-6~10 | Chuyển về DRAFT | Action | System processing |
| UC-05 | AF-05.a | Hủy thao tác | UI State | Dialog dismissed |
| UC-05 | AF-05.b | Có lời mời PENDING | UI State | Warning dialog |
| UC-05 | EF-05.1 | Có Deal liên kết | UI State | Error blocking |
