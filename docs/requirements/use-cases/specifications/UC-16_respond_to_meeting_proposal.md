# UC-16: Phản hồi đề xuất lịch họp

**Brief Description**
> Authenticated User (bên nhận đề xuất) phản hồi lịch họp đã đề xuất bằng cách chấp nhận, từ chối, hoặc đề xuất thời gian khác. Hệ thống thông báo kết quả cho bên đề xuất.

---

**Actors**
k
| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Bên nhận đề xuất) | Người phản hồi đề xuất lịch họp |
| Secondary | System | Cập nhật trạng thái, gửi thông báo |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Meeting đang ở trạng thái PROPOSED
- Actor là bên nhận đề xuất (không phải bên đề xuất)

---

**Trigger**
> Bên nhận nhấn "Chấp nhận", "Từ chối", hoặc "Đề xuất lại" trên lịch họp đã đề xuất.

---

**Main Flow (Basic Path) — Chấp nhận**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Xem chi tiết đề xuất lịch họp |
| 2 | System | Hiển thị thông tin: ngày giờ, thời lượng, chủ đề, ghi chú, hình thức |
| 3 | Authenticated User | Nhấn "Chấp nhận" |
| 4 | System | Chuyển meeting sang trạng thái CONFIRMED |
| 5 | System | Ghi nhận responded_at |
| 6 | System | Gửi thông báo cho bên đề xuất "Đề xuất lịch họp đã được chấp nhận" |
| 7 | System | Lên lịch nhắc nhở tự động 30 phút trước giờ họp cho cả hai bên |
| 8 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-16.a: Từ chối đề xuất (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Nhấn "Từ chối" |
| 3b | Authenticated User | Nhập ghi chú phản hồi (tùy chọn) |
| 3c | System | Chuyển meeting sang trạng thái DECLINED |
| 3d | System | Gửi thông báo cho bên đề xuất kèm ghi chú. Use case kết thúc |

> AF-16.b: Đề xuất thời gian khác (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Nhấn "Đề xuất lại" |
| 3b | System | Hiển thị form chọn ngày giờ mới |
| 3c | Authenticated User | Chọn ngày giờ mới và nhập ghi chú (tùy chọn) |
| 3d | System | Chuyển meeting gốc sang trạng thái RESCHEDULED |
| 3e | System | Tạo meeting mới với thời gian đề xuất mới, liên kết (rescheduled_from_id) đến meeting gốc |
| 3f | System | Gửi thông báo cho bên đề xuất về đề xuất thời gian mới |
| 3g | System | Use case kết thúc — bên đề xuất nhận đề xuất mới để phản hồi |

---

**Exception Flows**

> EF-16.1: Đề xuất thời gian mới không hợp lệ (triggered at Step 3c trong AF-16.b)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3c-a | System | Phát hiện ngày giờ mới trong quá khứ hoặc dưới 1 giờ từ hiện tại |
| 3c-b | System | Từ chối với thông báo "Thời gian họp phải ít nhất 1 giờ kể từ hiện tại" |
| 3c-c | Authenticated User | Chọn thời gian hợp lệ |

---

**Postconditions**

*Success (Chấp nhận):*
- Meeting chuyển sang CONFIRMED
- Nhắc nhở 30 phút trước giờ họp được lên lịch

*Success (Từ chối):*
- Meeting chuyển sang DECLINED

*Success (Đề xuất lại):*
- Meeting gốc chuyển sang RESCHEDULED
- Meeting mới được tạo với trạng thái PROPOSED

*Failure:*
- Meeting không thay đổi trạng thái

---

**Business Rules**

- BR-0404: Thời gian đề xuất lại phải trong tương lai (≥ 1 giờ). Nhắc nhở 30 phút trước giờ họp CONFIRMED

---

**Notes / Assumptions**

- Quá trình đề xuất lại có thể lặp lại nhiều lần cho đến khi hai bên đồng ý
- Sau khi meeting CONFIRMED, cả hai bên có thể ghi nhận kết quả qua UC-17
- Liên kết: UC-15, UC-17
