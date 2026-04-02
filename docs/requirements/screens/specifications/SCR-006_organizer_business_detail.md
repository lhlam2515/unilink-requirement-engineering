# SCR-006: Organizer_BusinessDetail_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-006 |
| **Screen Name** | Organizer_BusinessDetail_Screen |
| **Mục đích** | Organizer xem toàn bộ thông tin chi tiết doanh nghiệp, thực hiện bookmark, và gửi lời mời tài trợ |
| **Actor chính** | Organizer (BTC) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 2 — Tìm kiếm và tiếp cận đối tác phù hợp |
| **Use case liên quan** | UC-09, UC-10 (bookmark action), UC-11 (send invitation modal) |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Xem chi tiết doanh nghiệp — context switch từ list |
| Data scope riêng | ✅ Toàn bộ thông tin 1 doanh nghiệp cụ thể |
| Action set riêng | ✅ Bookmark, gửi lời mời — khác với list |
| Navigation boundary | ✅ From SCR-004 vào detail |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Tên doanh nghiệp (business_name)
- Logo (logo_url)
- Lĩnh vực hoạt động (industry)
- Khu vực (region)
- Đối tượng khách hàng (target_customers)
- Mục tiêu tài trợ (sponsorship_goal): MARKETING / CSR
- Ngân sách tài trợ dự kiến (sponsorship_budget)
- Điểm uy tín (nếu có — link đến UC-30)

---

## Dữ liệu nhập (Input Fields)

### Modal: Gửi lời mời tài trợ (UC-11)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Tin nhắn giới thiệu | Textarea | Bắt buộc, ≥ 20 ký tự | invitation_message |
| Gói tài trợ ưu tiên | Select | Tùy chọn — organizer chọn từ gói của hồ sơ mình | preferred_package |
| Hồ sơ tài trợ liên quan | Select | Bắt buộc — chọn hồ sơ PUBLISHED | proposal_id |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Gửi lời mời tài trợ** | Mở modal gửi lời mời (UC-11) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Bookmark / Bỏ lưu | Toggle bookmark (UC-10) |
| Quay lại | SCR-004 (Business Search) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0204: Chỉ xem chi tiết doanh nghiệp ACTIVE
- BR-0205: Mỗi actor bookmark 1 lần
- BR-0301: Lời mời chỉ gửi đến hồ sơ PUBLISHED
- BR-0302: Mỗi cặp chỉ 1 lời mời PENDING
- BR-0303: Tin nhắn ≥ 20 ký tự

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-004 (Business Search) | Nhấn vào doanh nghiệp trong kết quả / tab gợi ý |
| SCR-007 (Bookmark List) | Nhấn vào bookmark doanh nghiệp |
| Đường dẫn trực tiếp | Deep link |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | SCR-004 |
| Gửi lời mời thành công | Ở lại SCR-006 (success toast) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải chi tiết |
| Not Found State | Doanh nghiệp INACTIVE / không tồn tại (UC-09 EF-09.1) |
| Bookmark Active State | Đã bookmark |
| Invitation Modal | Form gửi lời mời |
| Invitation Sent Toast | Thành công |
| Duplicate Invitation Error | Đã có PENDING (UC-11 EF-11.1) |
| Short Message Error | < 20 ký tự (UC-11 EF-11.2) |
| Proposal Unavailable Error | Hồ sơ không PUBLISHED (UC-11 EF-11.3) |

## UI Components liên quan

- Logo display — logo doanh nghiệp
- Info sections — thông tin chi tiết
- Bookmark toggle button
- CTA button — "Gửi lời mời tài trợ"
- Modal dialog — form gửi lời mời
- Toast notification

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-09 | Main-1 | Chọn doanh nghiệp | Navigation In | Page load |
| UC-09 | Main-2 | Kiểm tra ACTIVE | Action | System check |
| UC-09 | Main-3~4 | Hiển thị thông tin DN + tài trợ | Read-only data | Info sections |
| UC-09 | EF-09.1 | DN INACTIVE / không tồn tại | UI State | 404 page |
| UC-10 | Main-1~5 | Bookmark | Action + State | Toggle button |
| UC-10 | AF-10.b | Bỏ bookmark | Action | Toggle |
| UC-11 | Main-1~11 | Gửi lời mời (full flow) | Component | CTA + Modal + Toast |
| UC-11 | EF-11.1~3 | Lỗi gửi lời mời | UI State | Error messages |
