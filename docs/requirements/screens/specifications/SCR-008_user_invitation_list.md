# SCR-008: User_InvitationList_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-008 |
| **Screen Name** | User_InvitationList_Screen |
| **Mục đích** | Authenticated User xem và quản lý danh sách tất cả lời mời tài trợ đã gửi và đã nhận, lọc theo trạng thái và chiều lời mời |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 2.3 — Gửi lời mời tài trợ |
| **Use case liên quan** | UC-13 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Theo dõi lời mời — mục tiêu quản lý/tracking |
| Data scope riêng | ✅ Tất cả invitations sent/received với filters |
| Action set riêng | ✅ Lọc, xem chi tiết, điều hướng đến respond |
| Navigation boundary | ✅ Entry point từ dashboard |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

Mỗi lời mời hiển thị:

- Tên đối tác (partner_name)
- Tên sự kiện (event_name)
- Trạng thái (status): PENDING / ACCEPTED / DECLINED / EXPIRED
- Chiều lời mời: ĐÃ GỬI / ĐÃ NHẬN
- Ngày gửi (sent_at)
- Ngày phản hồi (responded_at) — nếu có
- Thống kê phân trang

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Bộ lọc trạng thái | Select | ALL / PENDING / ACCEPTED / DECLINED / EXPIRED |
| Bộ lọc chiều | Select | ALL / SENT / RECEIVED |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Nhấn vào lời mời** | Chuyển đến SCR-009 (Invitation Detail) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Áp dụng bộ lọc | Cập nhật danh sách |
| Chuyển trang | Tải trang tiếp |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0306: Lời mời PENDING tự động hết hạn sau 14 ngày

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Dashboard / Menu chính | Menu "Lời mời tài trợ" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn vào lời mời | SCR-009 (Invitation Detail) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải danh sách |
| Empty State | Chưa có lời mời (UC-13 EF-13.1) |

## UI Components liên quan

- Data table / Card list — danh sách lời mời
- Filter bar — trạng thái + chiều
- Status badge — PENDING / ACCEPTED / DECLINED / EXPIRED
- Direction badge — ĐÃ GỬI / ĐÃ NHẬN
- Pagination

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-13 | Main-1 | Truy cập trang lời mời | Screen entry | Page load |
| UC-13 | Main-2 | Hiển thị danh sách + bộ lọc | Screen content | Table + Filters |
| UC-13 | Main-3 | Áp dụng bộ lọc | Input + Action | Filter selects |
| UC-13 | Main-4 | Truy xuất + phân trang | Action | System processing |
| UC-13 | Main-5 | Mỗi lời mời hiển thị thông tin | Read-only data | Row/card content |
| UC-13 | AF-13.a Step 5a | Nhấn vào lời mời | Navigation | → SCR-009 |
| UC-13 | EF-13.1 | Không có lời mời | UI State | Empty state |
