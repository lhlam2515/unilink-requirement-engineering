# SCR-024: Admin_VerificationList_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-024 |
| **Screen Name** | Admin_VerificationList_Screen |
| **Mục đích** | Admin xem danh sách hồ sơ xác thực tổ chức đang chờ kiểm duyệt |
| **Actor chính** | Admin |
| **Quy trình nghiệp vụ** | BP-02 / Bước 5 — Kiểm duyệt hồ sơ |
| **Use case liên quan** | UC-41 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Quản lý hàng đợi kiểm duyệt |
| Data scope riêng | ✅ Danh sách hồ sơ pending với metadata tóm tắt |
| Action set riêng | ✅ Lọc, tìm kiếm, phân trang, điều hướng chi tiết |
| Navigation boundary | ✅ Entry point từ admin panel → SCR-025 |
| Independently testable | ✅ Có thể viết acceptance criteria riêng |

---

## Dữ liệu hiển thị (Read-only Data)

- Tổng số hồ sơ đang chờ (pending count)
- Thời gian chờ trung bình
- Mỗi hồ sơ: tên tổ chức, vai trò (badge), ngày gửi, thời gian chờ, đánh dấu "Đã cập nhật" (nếu có), số lần gửi

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Bộ lọc vai trò | Select (Tất cả / Organizer / Sponsor) | Mặc định: Tất cả |
| Tìm kiếm tên tổ chức | Text input | Tìm kiếm theo tên |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Nhấn vào hồ sơ** | Chuyển đến SCR-025 (Admin_VerificationDetail_Screen) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Áp dụng bộ lọc | Cập nhật danh sách (AF-41.a) |
| Tìm kiếm | Cập nhật danh sách (AF-41.b) |
| Chuyển trang | Trang tiếp theo (AF-41.c) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-1001: Sắp xếp theo submitted_at ASC (FIFO). Phân trang 20 hồ sơ/trang

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Admin Panel | Menu "Kiểm duyệt hồ sơ" |
| SCR-025 | Nút "Quay lại danh sách" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn vào hồ sơ | SCR-025 |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải danh sách |
| Empty State | Không có hồ sơ pending (AF-41.d) |
| Error State | Lỗi hệ thống (EF-41.1) |

## UI Components liên quan

- Statistics summary, Data table/Card list, Filter bar, Search input, Pagination, Role badge, Wait time indicator

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-41 | Main-1 | Admin truy cập trang kiểm duyệt | Navigation In | Screen entry |
| UC-41 | Main-2~3 | System truy xuất và sắp xếp | Action | System xử lý |
| UC-41 | Main-4 | Hiển thị danh sách | Read-only | Data table |
| UC-41 | Main-5 | Hiển thị thống kê | Read-only | Statistics summary |
| UC-41 | AF-41.a | Lọc theo vai trò | Action | Filter dropdown |
| UC-41 | AF-41.b | Tìm kiếm theo tên | Action | Search input |
| UC-41 | AF-41.c | Phân trang | Action | Pagination |
| UC-41 | AF-41.d | Không có hồ sơ pending | UI State | Empty state |
| UC-41 | EF-41.1 | Lỗi truy xuất | UI State | Error state |
