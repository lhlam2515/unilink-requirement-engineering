# SCR-005: Sponsor_ProposalDetail_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-005 |
| **Screen Name** | Sponsor_ProposalDetail_Screen |
| **Mục đích** | Sponsor xem toàn bộ thông tin chi tiết của hồ sơ tài trợ sự kiện, thực hiện bookmark, và gửi lời mời tài trợ cho BTC |
| **Actor chính** | Sponsor (Doanh nghiệp) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 2 — Tìm kiếm và tiếp cận đối tác phù hợp |
| **Use case liên quan** | UC-08, UC-10 (bookmark action), UC-11 (send invitation modal) |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Xem chi tiết sự kiện — context switch từ list |
| Data scope riêng | ✅ Toàn bộ fields của 1 proposal cụ thể (detail view) |
| Action set riêng | ✅ Bookmark, gửi lời mời — khác với list view |
| Navigation boundary | ✅ From SCR-003 (search results) vào detail |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- **Thông tin cơ bản**: Tên sự kiện, loại hình, thời gian tổ chức, địa điểm
- **Hình ảnh nhận diện**: Banner, thumbnail
- **Thông tin chi tiết**: Quy mô dự kiến, ngân sách dự kiến, đối tượng khán giả, nội dung chương trình
- **Hình thức tài trợ**: CASH / IN_KIND / COMBINED
- **Danh sách gói tài trợ** (cho mỗi gói):
  - Tên gói, cấp độ, mô tả, giá trị tối thiểu, số slot khả dụng
  - Danh sách quyền lợi: nhóm, tiêu đề, mô tả cam kết
- **Thông tin BTC**: Tên tổ chức, điểm uy tín (nếu có — link đến UC-30)

---

## Dữ liệu nhập (Input Fields)

Không có input field trực tiếp trên screen chính. Các input nằm trong modal gửi lời mời:

### Modal: Gửi lời mời tài trợ (UC-11)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Tin nhắn giới thiệu | Textarea | Bắt buộc, ≥ 20 ký tự | invitation_message |
| Gói tài trợ ưu tiên | Select | Tùy chọn | preferred_package |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Gửi lời mời tài trợ** | Mở modal gửi lời mời (UC-11) → Tạo invitation PENDING |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Bookmark / Bỏ lưu | Toggle bookmark (UC-10 Main Flow / AF-10.b) |
| Quay lại kết quả tìm kiếm | Chuyển đến SCR-003 |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0204: Chỉ xem chi tiết hồ sơ PUBLISHED. Truy cập DRAFT bị từ chối
- BR-0205: Mỗi actor bookmark 1 hồ sơ MỘT LẦN
- BR-0301: Lời mời chỉ gửi đến hồ sơ PUBLISHED
- BR-0302: Mỗi cặp (proposal + business) chỉ có 1 lời mời PENDING
- BR-0303: Tin nhắn giới thiệu bắt buộc, ≥ 20 ký tự

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-003 (Event Search) | Nhấn vào sự kiện trong kết quả tìm kiếm hoặc tab gợi ý |
| SCR-007 (Bookmark List) | Nhấn vào bookmark sự kiện |
| Đường dẫn trực tiếp (URL) | Deep link |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | SCR-003 (Event Search) |
| Gửi lời mời thành công | Ở lại SCR-005 (success toast) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải chi tiết hồ sơ |
| Not Found State | Hồ sơ DRAFT hoặc không tồn tại (UC-08 EF-08.1) — hiển thị 404 |
| Bookmark Active State | Hồ sơ đã được bookmark — hiển thị "Đã lưu" |
| Invitation Sent Toast | Lời mời gửi thành công (UC-11 Main-10) |
| Invitation Modal | Form gửi lời mời (UC-11 Main-2~5) |
| Duplicate Invitation Error | Đã có lời mời PENDING (UC-11 EF-11.1) |
| Short Message Error | Tin nhắn < 20 ký tự (UC-11 EF-11.2) |
| Proposal Unavailable Error | Hồ sơ bị hủy phát hành (UC-11 EF-11.3) |

## UI Components liên quan

- Banner / Hero image — hình ảnh nhận diện
- Section cards — thông tin cơ bản, chi tiết, hình thức tài trợ
- Accordion / Expandable list — gói tài trợ + quyền lợi
- Bookmark toggle button — lưu/bỏ lưu
- CTA button — "Gửi lời mời tài trợ"
- Modal dialog — form gửi lời mời (UC-11)
- Toast notification — thành công/lỗi

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-08 | Main-1 | Chọn sự kiện từ kết quả | Navigation In | Page load |
| UC-08 | Main-2 | Kiểm tra PUBLISHED | Action | System check → 404 if not |
| UC-08 | Main-3~7 | Hiển thị toàn bộ thông tin | Read-only data | Sections: cơ bản, chi tiết, ảnh, hình thức, gói |
| UC-08 | EF-08.1 | Hồ sơ không PUBLISHED | UI State | 404 Not Found page |
| UC-10 | Main-1~5 | Nhấn Bookmark | Action + State | Bookmark toggle button |
| UC-10 | AF-10.b | Bỏ bookmark | Action + State | Toggle "Bỏ lưu" |
| UC-10 | EF-10.1 | Đã bookmark trước đó | UI State | Nút hiển thị "Đã lưu" |
| UC-11 | Main-1 | Nhấn "Gửi lời mời" | Action | CTA button |
| UC-11 | Main-2 | Hiển thị form modal | Component | Invitation modal |
| UC-11 | Main-3~5 | Nhập message + chọn package + gửi | Input + Action | Modal form fields + Submit |
| UC-11 | Main-6~9 | Validate + tạo + thông báo | Action | System processing |
| UC-11 | Main-10 | Xác nhận thành công | UI State | Success toast |
| UC-11 | EF-11.1 | Đã có PENDING invitation | UI State | Error in modal |
| UC-11 | EF-11.2 | Message < 20 chars | UI State | Validation error |
| UC-11 | EF-11.3 | Proposal no longer PUBLISHED | UI State | Error message |
