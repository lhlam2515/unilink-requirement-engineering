# UC-15: Đặt lịch họp thương thảo

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) đề xuất lịch họp/meeting với đối tác trong phạm vi thương vụ, bao gồm ngày giờ, thời lượng, chủ đề, và hình thức họp (online/trực tiếp). Hệ thống chỉ ghi nhận lịch hẹn và gửi nhắc nhở; hệ thống không tổ chức hoặc host cuộc gọi.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người đề xuất lịch họp |
| Secondary | System | Xác thực, gửi thông báo, nhắc nhở |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái IN_PROGRESS
- Actor là một trong hai bên liên quan trong deal

---

**Trigger**
> Actor nhấn "Đặt lịch họp" trong trang thương thảo của deal.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Nhấn "Đặt lịch họp" trong trang thương thảo |
| 2 | System | Hiển thị form đặt lịch: ngày giờ, thời lượng, chủ đề, ghi chú, hình thức họp (ONLINE/IN_PERSON), link/địa điểm |
| 3 | Authenticated User | Nhập ngày giờ họp, thời lượng dự kiến (phút), và chủ đề (bắt buộc) |
| 4 | Authenticated User | Chọn hình thức họp và nhập link/địa điểm (nếu có) |
| 5 | Authenticated User | Nhấn "Gửi đề xuất" |
| 6 | System | Xác thực: ngày giờ phải trong tương lai (≥ 1 giờ từ hiện tại) |
| 7 | System | Tạo meeting với trạng thái PROPOSED, ghi nhận proposed_at |
| 8 | System | Gửi thông báo in-app và email cho đối tác về đề xuất lịch họp |
| 9 | System | Use case kết thúc thành công — đề xuất đã gửi, chờ đối tác phản hồi |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-15.1: Ngày giờ không hợp lệ (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện ngày giờ họp trong quá khứ hoặc dưới 1 giờ từ hiện tại |
| 6b | System | Từ chối với thông báo "Thời gian họp phải ít nhất 1 giờ kể từ hiện tại" |
| 6c | Authenticated User | Chọn thời gian hợp lệ và thử gửi lại |

---

**Postconditions**

*Success:*
- Meeting được tạo với trạng thái PROPOSED
- Đối tác được thông báo và có thể phản hồi (UC-16)
- Khi meeting CONFIRMED, hệ thống gửi nhắc nhở 30 phút trước giờ họp

*Failure:*
- Meeting không được tạo
- Actor được thông báo lỗi xác thực

---

**Business Rules**

- BR-0404: Lịch họp chỉ đề xuất với ngày giờ trong tương lai (≥ 1 giờ). Hệ thống gửi nhắc nhở 30 phút trước giờ họp CONFIRMED

---

**Notes / Assumptions**

- Hệ thống chỉ đóng vai trò ghi nhận lịch hẹn và gửi nhắc nhở; không cung cấp giao diện video call hay tổ chức cuộc họp trực tuyến
- Hệ thống không tích hợp lịch bên ngoài (Google Calendar, v.v.) ở phiên bản đầu
- Đối tác phản hồi đề xuất qua UC-16
- Kết quả họp được ghi nhận qua UC-17
- Liên kết: UC-14, UC-16, UC-17
