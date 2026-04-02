# SCR-007: User_BookmarkList_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-007 |
| **Screen Name** | User_BookmarkList_Screen |
| **Mục đích** | Authenticated User xem và quản lý danh sách hồ sơ đã bookmark (hồ sơ tài trợ sự kiện hoặc hồ sơ doanh nghiệp) |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 2 — Tìm kiếm và tiếp cận đối tác phù hợp |
| **Use case liên quan** | UC-10 (AF-10.a — xem danh sách, AF-10.b — bỏ bookmark) |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Quản lý bookmark — mục tiêu tham khảo/review |
| Data scope riêng | ✅ Tất cả bookmark của user, khác với search results |
| Action set riêng | ✅ Bỏ bookmark, điều hướng đến chi tiết |
| Navigation boundary | ✅ Entry point từ dashboard |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

Mỗi bookmark hiển thị:

- Loại (target_type): Hồ sơ tài trợ (PROPOSAL) hoặc Doanh nghiệp (BUSINESS)
- Tên (tên sự kiện hoặc tên doanh nghiệp)
- Thumbnail / Logo
- Thông tin tóm tắt (loại hình, địa điểm, lĩnh vực)
- Ngày bookmark (bookmarked_at)
- Trạng thái khả dụng: "Đang hoạt động" hoặc "Hồ sơ không còn khả dụng"

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Bộ lọc loại | Select (ALL / PROPOSAL / BUSINESS) | Lọc theo loại bookmark |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Nhấn vào bookmark** | Chuyển đến SCR-005 (nếu PROPOSAL) hoặc SCR-006 (nếu BUSINESS) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Bỏ bookmark | Xóa khỏi danh sách (UC-10 AF-10.b) |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0205: Bookmark hồ sơ bị hủy phát hành được giữ lại nhưng đánh dấu "không khả dụng"

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Dashboard / Menu chính | Menu "Danh sách quan tâm" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn bookmark PROPOSAL | SCR-005 (Proposal Detail) |
| Nhấn bookmark BUSINESS | SCR-006 (Business Detail) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải danh sách |
| Empty State | Chưa có bookmark, hiển thị CTA "Khám phá sự kiện/doanh nghiệp" |
| Unavailable Badge | Hồ sơ đã bị hủy phát hành |

## UI Components liên quan

- Card list — danh sách bookmark
- Filter tabs — ALL / PROPOSAL / BUSINESS
- Unavailable badge — nhãn "không còn khả dụng"
- Remove button — bỏ bookmark
- Empty state illustration

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-10 | AF-10.a Step 1a | Truy cập danh sách bookmark | Screen entry | Page load |
| UC-10 | AF-10.a Step 1b | Hiển thị danh sách + tóm tắt | Read-only data | Card list |
| UC-10 | AF-10.a Step 1c | Đánh dấu hồ sơ hủy phát hành | Read-only data | Unavailable badge |
| UC-10 | AF-10.b | Bỏ bookmark | Action | Remove button |
