# SCR-010: User_DealList_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-010 |
| **Screen Name** | User_DealList_Screen |
| **Mục đích** | Authenticated User xem danh sách tất cả thương vụ (deals) mà mình tham gia, với trạng thái và thông tin tóm tắt |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 3 — Thương thảo hợp đồng tài trợ |
| **Use case liên quan** | UC-14 (entry point) |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Quản lý danh sách deals — overview/tracking |
| Data scope riêng | ✅ Tất cả deals của user với metadata tóm tắt |
| Action set riêng | ✅ Lọc, chọn deal để điều hướng |
| Navigation boundary | ✅ Entry from dashboard, navigate to SCR-011 |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

Mỗi deal hiển thị:

- Tên đối tác (partner_name)
- Tên sự kiện (event_name)
- Trạng thái deal (status): IN_PROGRESS / AGREED / TERMINATED
- Trạng thái hợp đồng (contract_status): DRAFTING / CONFIRMED / SIGNED — nếu AGREED
- Ngày tạo (created_at)
- Tin nhắn chưa đọc (unread_count)
- Cuộc họp sắp tới (next_meeting) — nếu có

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Bộ lọc trạng thái | Select | ALL / IN_PROGRESS / AGREED / TERMINATED |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Nhấn vào deal** | Chuyển đến SCR-011 (Deal Negotiation) |

---

## Quy tắc nghiệp vụ (Business Rules)

Không có business rule riêng cho screen này.

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Dashboard / Menu chính | Menu "Thương vụ" |
| SCR-009 (sau khi Accept invitation) | Redirect sau tạo deal |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn vào deal | SCR-011 (Deal Negotiation) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải danh sách |
| Empty State | Chưa có deal nào |

## UI Components liên quan

- Card list / Data table — danh sách deals
- Status badge — IN_PROGRESS / AGREED / TERMINATED
- Unread count badge — số tin nhắn chưa đọc
- Filter bar
- Pagination

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-14 | — | Entry point: danh sách deals → chọn deal | Screen entry | Deal list |
| UC-12 | Main-6 | Deal IN_PROGRESS mới tạo xuất hiện | Read-only data | New deal in list |
