# UC-12: Phản hồi lời mời tài trợ

**Brief Description**
> Authenticated User (bên nhận lời mời) chấp nhận hoặc từ chối lời mời tài trợ đang ở trạng thái PENDING. Khi chấp nhận, hệ thống tự động tạo deal/negotiation context để tiến vào giai đoạn thương thảo. Khi từ chối, bên nhận có thể kèm lý do.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Bên nhận — Organizer hoặc Sponsor) | Người phản hồi lời mời |
| Secondary | System | Chuyển trạng thái, tạo deal, gửi thông báo |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Lời mời tài trợ đang ở trạng thái PENDING
- Actor là bên nhận (recipient) của lời mời

---

**Trigger**
> Bên nhận nhấn "Chấp nhận" hoặc "Từ chối" trên trang chi tiết lời mời tài trợ.

---

**Main Flow (Basic Path) — Chấp nhận lời mời**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Mở trang chi tiết lời mời tài trợ đang PENDING |
| 2 | System | Hiển thị thông tin lời mời: bên gửi, sự kiện, tin nhắn giới thiệu, gói tài trợ ưu tiên (nếu có) |
| 3 | Authenticated User | Nhấn "Chấp nhận lời mời" |
| 4 | Authenticated User | Nhập tin nhắn phản hồi (tùy chọn) |
| 5 | System | Chuyển trạng thái lời mời sang ACCEPTED, ghi nhận accepted_at |
| 6 | System | Tự động tạo deal (negotiation context) mới với trạng thái IN_PROGRESS |
| 7 | System | Gửi thông báo in-app và email cho bên gửi: "Lời mời tài trợ đã được chấp nhận" |
| 8 | System | Use case kết thúc thành công — deal sẵn sàng cho giai đoạn thương thảo (SF-04) |

---

**Alternate Flows**

> AF-12.a: Từ chối lời mời tài trợ (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Nhấn "Từ chối lời mời" |
| 3b | System | Hiển thị form nhập lý do từ chối (tùy chọn) |
| 3c | Authenticated User | Nhập lý do từ chối (hoặc bỏ trống) |
| 3d | Authenticated User | Nhấn xác nhận từ chối |
| 3e | System | Chuyển trạng thái lời mời sang DECLINED, ghi nhận declined_at |
| 3f | System | Gửi thông báo cho bên gửi bao gồm lý do từ chối (nếu có) |
| 3g | System | Use case kết thúc — lời mời bị từ chối |

---

**Exception Flows**

> EF-12.1: Lời mời đã hết hạn (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện lời mời đã ở trạng thái EXPIRED (quá 14 ngày) |
| 3b | System | Từ chối với thông báo "Lời mời đã hết hạn" |
| 3c | Authenticated User | Có thể liên hệ bên gửi để yêu cầu lời mời mới |

> EF-12.2: Lời mời không còn PENDING (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện lời mời đã ở trạng thái ACCEPTED, DECLINED, hoặc EXPIRED |
| 3b | System | Từ chối với thông báo "Lời mời đã được xử lý trước đó" |

---

**Postconditions**

*Success (Chấp nhận):*
- Lời mời chuyển sang trạng thái ACCEPTED
- Deal mới được tạo với trạng thái IN_PROGRESS, liên kết đến lời mời
- Bên gửi được thông báo
- Hai bên sẵn sàng thương thảo (UC-14 đến UC-19)

*Success (Từ chối):*
- Lời mời chuyển sang trạng thái DECLINED
- Bên gửi được thông báo kèm lý do (nếu có)
- Bên gửi có thể gửi lời mời mới sau này (khi lời mời cũ không còn PENDING)

*Failure:*
- Lời mời không thay đổi trạng thái
- Actor được thông báo lỗi cụ thể

---

**Business Rules**

- BR-0305: Chỉ lời mời ở trạng thái PENDING mới có thể chấp nhận hoặc từ chối. Lời mời đã ACCEPTED, DECLINED, EXPIRED không thể thay đổi
- BR-0306: Lời mời PENDING tự động hết hạn sau 14 ngày (xử lý bởi System scheduled job)

---

**Notes / Assumptions**

- Lý do từ chối là tùy chọn nhưng được khuyến khích
- Khi chấp nhận, deal_id mới được tạo tự động — liên kết đến SF-04
- Lời mời hết hạn được xử lý tự động bởi scheduled job (chạy mỗi giờ), thông báo cho cả hai bên
- Liên kết: UC-11, UC-13, UC-14
