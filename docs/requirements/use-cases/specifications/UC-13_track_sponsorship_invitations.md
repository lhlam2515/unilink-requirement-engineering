# UC-13: Theo dõi danh sách lời mời tài trợ

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) xem và quản lý danh sách tất cả lời mời tài trợ đã gửi và đã nhận, lọc theo trạng thái và chiều lời mời.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người theo dõi lời mời |
| Secondary | System | Truy xuất và hiển thị danh sách lời mời |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống

---

**Trigger**
> Actor truy cập trang "Lời mời tài trợ" trong dashboard cá nhân.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Truy cập trang "Lời mời tài trợ" trong dashboard |
| 2 | System | Hiển thị danh sách lời mời với bộ lọc: trạng thái (ALL/PENDING/ACCEPTED/DECLINED/EXPIRED) và chiều (SENT/RECEIVED/ALL) |
| 3 | Authenticated User | Áp dụng bộ lọc mong muốn |
| 4 | System | Truy xuất và hiển thị danh sách lời mời phân trang |
| 5 | System | Mỗi lời mời hiển thị: tên đối tác, tên sự kiện, trạng thái, ngày gửi, ngày phản hồi (nếu có) |
| 6 | System | Use case kết thúc thành công — actor thấy danh sách lời mời |

---

**Alternate Flows**

> AF-13.a: Actor xem chi tiết lời mời (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Authenticated User | Nhấn vào một lời mời cụ thể trong danh sách |
| 5b | System | Hiển thị trang chi tiết lời mời với đầy đủ thông tin |
| 5c | System | Nếu lời mời PENDING và actor là bên nhận: hiển thị nút "Chấp nhận" / "Từ chối" (chuyển sang UC-12) |

---

**Exception Flows**

> EF-13.1: Không có lời mời nào (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Không tìm thấy lời mời nào khớp bộ lọc |
| 4b | System | Hiển thị thông báo "Chưa có lời mời tài trợ nào" |

---

**Postconditions**

*Success:*
- Actor xem được danh sách lời mời đã gửi/nhận với trạng thái hiện tại

*Failure:*
- Danh sách trống — actor được thông báo

---

**Business Rules**

- Không có business rule riêng cho use case này

---

**Notes / Assumptions**

- Danh sách lời mời hiển thị cả lời mời đã gửi và đã nhận
- Actor có thể chuyển sang UC-12 để phản hồi lời mời PENDING
- Liên kết: UC-11, UC-12
