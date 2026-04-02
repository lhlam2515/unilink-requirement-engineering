# SCR-001: Organizer_ProposalList_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-001 |
| **Screen Name** | Organizer_ProposalList_Screen |
| **Mục đích** | Organizer xem, quản lý danh sách tất cả hồ sơ tài trợ sự kiện của tổ chức mình — bao gồm tạo mới, xem trạng thái, và điều hướng đến chi tiết/chỉnh sửa |
| **Actor chính** | Organizer (BTC) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 1 — Tạo hồ sơ tài trợ sự kiện |
| **Use case liên quan** | UC-01, UC-04, UC-05 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Quản lý danh sách hồ sơ — mục tiêu overview/browsing |
| Data scope riêng | ✅ Danh sách tất cả proposals với metadata tóm tắt |
| Action set riêng | ✅ Tạo mới, lọc, sắp xếp, điều hướng |
| Navigation boundary | ✅ Entry point từ dashboard/menu, điều hướng sang SCR-002 |
| Independently testable | ✅ Có thể viết acceptance criteria riêng |

---

## Dữ liệu hiển thị (Read-only Data)

Mỗi hồ sơ trong danh sách hiển thị:

- Tên chương trình (event_name)
- Thumbnail ảnh nhận diện (thumbnail_url)
- Trạng thái (status): `DRAFT` / `PUBLISHED`
- Loại hình sự kiện (event_type)
- Thời gian tổ chức (event_date_start ~ event_date_end)
- Số gói tài trợ (package_count)
- Ngày tạo (created_at)
- Ngày phát hành (published_at) — nếu PUBLISHED

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Bộ lọc trạng thái | Select (ALL / DRAFT / PUBLISHED) | Mặc định: ALL |
| Bộ lọc từ khóa | Text input | Tìm theo tên sự kiện |
| Tiêu chí sắp xếp | Select (Ngày tạo / Ngày phát hành / Tên) | Mặc định: Ngày tạo giảm dần |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Tạo hồ sơ tài trợ mới** | System tạo hồ sơ DRAFT → Chuyển đến SCR-002 (UC-01 Main Flow) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Nhấn vào một hồ sơ | Chuyển đến SCR-002 (chỉnh sửa/xem chi tiết hồ sơ) |
| Áp dụng bộ lọc | Cập nhật danh sách hiển thị |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0101: Mỗi hồ sơ khi khởi tạo PHẢI có trạng thái DRAFT
- BR-0102: Mỗi hồ sơ PHẢI gắn với đúng 1 tổ chức BTC; 1 BTC có thể có nhiều hồ sơ

---

## Quy tắc xác thực (Validation Rules)

Không có validation đặc biệt trên screen này (chỉ hiển thị dữ liệu).

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Dashboard / Menu chính | Menu "Quản lý hồ sơ tài trợ" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn "Tạo hồ sơ tài trợ mới" | SCR-002 (hồ sơ mới DRAFT) |
| Nhấn vào hồ sơ trong danh sách | SCR-002 (chỉnh sửa hồ sơ hiện có) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải danh sách hồ sơ |
| Empty State | Organizer chưa có hồ sơ nào, hiển thị CTA "Tạo hồ sơ đầu tiên" |
| Error State | Lỗi hệ thống khi tải danh sách (EF-01.1) |

## UI Components liên quan

- Data table / Card grid — hiển thị danh sách hồ sơ
- Filter bar — bộ lọc trạng thái + từ khóa
- Sort dropdown — tiêu chí sắp xếp
- Pagination — phân trang
- Status badge — nhãn trạng thái DRAFT / PUBLISHED

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-01 | Main-1 | Organizer chọn "Tạo hồ sơ tài trợ mới" | Action | CTA button "Tạo hồ sơ tài trợ mới" |
| UC-01 | Main-2~5 | System tạo hồ sơ DRAFT | Action | System xử lý → redirect sang SCR-002 |
| UC-01 | EF-01.1 | Lỗi hệ thống khi tạo | UI State | Error toast notification |
| UC-04 | — | Trạng thái PUBLISHED hiển thị trên danh sách | Read-only data | Status badge "PUBLISHED" |
| UC-05 | — | Trạng thái DRAFT sau hủy phát hành | Read-only data | Status badge "DRAFT" |
