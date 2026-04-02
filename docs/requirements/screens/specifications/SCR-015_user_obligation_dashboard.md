# SCR-015: User_ObligationDashboard_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-015 |
| **Screen Name** | User_ObligationDashboard_Screen |
| **Mục đích** | Authenticated User xem tổng quan tiến trình nghĩa vụ tài trợ và danh sách nghĩa vụ của cả hai bên cho một hợp đồng đã ký |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 5 — Thực hiện nghĩa vụ tài trợ |
| **Use case liên quan** | UC-25 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Theo dõi nghĩa vụ — overview/tracking context |
| Data scope riêng | ✅ Danh sách nghĩa vụ + tiến trình tổng thể |
| Action set riêng | ✅ Lọc, xem chi tiết, điều hướng đến submit/confirm |
| Navigation boundary | ✅ Từ contract view sang obligation tracking |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

### Dashboard Overview

- Tiến trình tổng thể: tổng nghĩa vụ / hoàn thành / đang chờ / quá hạn
- Progress bar hoặc chart

### Danh sách nghĩa vụ đối tác

Mỗi nghĩa vụ hiển thị:

- Tên nghĩa vụ (obligation_name)
- Mô tả (description)
- Trạng thái: PENDING / IN_PROGRESS / SUBMITTED / CONFIRMED / DISPUTED / OVERDUE
- Deadline
- Nhãn "QUÁ HẠN" (màu đỏ) nếu overdue

### Danh sách nghĩa vụ bản thân

- Tương tự cấu trúc trên

---

## Dữ liệu nhập (Input Fields)

Không có input fields trên screen này (chỉ hiển thị).

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Nhấn vào nghĩa vụ** | Chuyển đến SCR-016 (Obligation Detail) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Nộp báo cáo kết quả | Chuyển đến SCR-017 (Event Report) — chỉ Organizer |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0601: Nghĩa vụ tạo tự động khi SIGNED. DN: chuyển khoản/bàn giao. BTC: quảng bá, truyền thông
- BR-0605: Quá deadline → OVERDUE tự động. Nhắc nhở T-3, T-0, T+1

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-014 (Contract View) | Link "Xem nghĩa vụ" |
| SCR-010 (Deal List) | Nhấn deal có HĐ SIGNED |
| Dashboard / Menu chính | Menu "Nghĩa vụ" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn vào nghĩa vụ | SCR-016 (Obligation Detail) |
| Nhấn "Nộp báo cáo" | SCR-017 (Event Report) |
| Quay lại | SCR-014 (Contract View) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải |
| Empty State | Chưa có nghĩa vụ — lỗi tạo tự động (UC-25 EF-25.1) |
| Overdue Highlight | Nghĩa vụ quá hạn — nhãn đỏ |

## UI Components liên quan

- Progress overview — tổng/hoàn thành/chờ/quá hạn (chart/progress bar)
- Obligation list — danh sách nghĩa vụ theo bên
- Status badges — PENDING / SUBMITTED / CONFIRMED / DISPUTED / OVERDUE
- Deadline display — ngày hạn
- Overdue badge — nhãn "QUÁ HẠN" (đỏ)
- Section divider — nghĩa vụ đối tác vs. nghĩa vụ bản thân

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-25 | Main-1 | Truy cập trang nghĩa vụ | Screen entry | Page load |
| UC-25 | Main-2 | Hiển thị dashboard tiến trình | Read-only data | Progress overview |
| UC-25 | Main-3 | Danh sách nghĩa vụ đối tác | Read-only data | Partner obligations list |
| UC-25 | Main-4 | Danh sách nghĩa vụ bản thân | Read-only data | My obligations list |
| UC-25 | Main-5 | Đánh dấu quá hạn | Read-only data | Overdue badge |
| UC-25 | Main-6 | Hiển thị deadline | Read-only data | Deadline display |
| UC-25 | AF-25.a | Nhấn vào nghĩa vụ | Navigation | → SCR-016 |
| UC-25 | EF-25.1 | Chưa có nghĩa vụ | UI State | Empty state |
