# Use-Case Specification: UC-12 — Phản hồi lời mời tài trợ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Bên nhận — Organizer hoặc Sponsor) | Người phản hồi lời mời |
| Secondary | System | Chuyển trạng thái, tạo deal, gửi thông báo |

---

### 1. Brief Description

> Authenticated User (bên nhận lời mời) chấp nhận hoặc từ chối lời mời tài trợ đang ở trạng thái PENDING. Khi chấp nhận, hệ thống tự động tạo deal/negotiation context để tiến vào giai đoạn thương thảo. Khi từ chối, bên nhận có thể kèm lý do.

---

### 2. Flow of Events

**Trigger**
> Bên nhận nhấn "Chấp nhận" hoặc "Từ chối" trên trang chi tiết lời mời tài trợ.

#### 2.1 Basic Flow — Chấp nhận lời mời

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Mở trang chi tiết lời mời tài trợ đang PENDING |
| 2 | System | Hiển thị thông tin lời mời: bên gửi, sự kiện, tin nhắn giới thiệu, gói tài trợ ưu tiên (nếu có) |
| 3 | Authenticated User | Nhấn "Chấp nhận lời mời" |
| 4 | Authenticated User | Nhập tin nhắn phản hồi (tùy chọn) |
| 5 | System | Chuyển trạng thái lời mời sang ACCEPTED, ghi nhận accepted_at |
| 6 | System | Tự động tạo deal (negotiation context) mới với trạng thái IN_PROGRESS |
| 7 | System | Gửi thông báo in-app và email cho bên gửi: "Lời mời tài trợ đã được chấp nhận" |
| 8 | System | Use case kết thúc thành công — deal sẵn sàng cho giai đoạn thương thảo (SF-04) |

#### 2.2 Alternate Flows

##### AF-12.a: Từ chối lời mời tài trợ
>
> *Triggered at Step 3 of the Basic Flow when bên nhận chọn từ chối.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Nhấn "Từ chối lời mời" |
| 3b | System | Hiển thị form nhập lý do từ chối (tùy chọn) |
| 3c | Authenticated User | Nhập lý do từ chối (hoặc bỏ trống) |
| 3d | Authenticated User | Nhấn xác nhận từ chối |
| 3e | System | Chuyển trạng thái lời mời sang DECLINED, ghi nhận declined_at |
| 3f | System | Gửi thông báo cho bên gửi bao gồm lý do từ chối (nếu có) |
| 3g | System | Use case kết thúc — lời mời bị từ chối |

#### 2.3 Exception Flows

##### EF-12.1: Lời mời đã hết hạn
>
> *Triggered at Step 3 of the Basic Flow when lời mời đã EXPIRED (quá 30 ngày theo BR-0306).*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện lời mời đã ở trạng thái EXPIRED (quá 30 ngày) |
| 3b | System | Từ chối với thông báo "Lời mời đã hết hạn" |
| 3c | Authenticated User | Có thể liên hệ bên gửi để yêu cầu lời mời mới |

##### EF-12.2: Lời mời không còn PENDING
>
> *Triggered at Step 3 of the Basic Flow when lời mời đã được xử lý.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện lời mời đã ở trạng thái ACCEPTED, DECLINED, hoặc EXPIRED |
| 3b | System | Từ chối với thông báo "Lời mời đã được xử lý trước đó" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-12-01 | Chấp nhận lời mời | Bên nhận chấp nhận lời mời PENDING; deal mới được tạo cho giai đoạn thương thảo |
| SC-12-02 | Từ chối lời mời | Bên nhận từ chối lời mời PENDING, có thể kèm lý do; bên gửi được thông báo (AF-12.a) |
| SC-12-03 | Lời mời đã hết hạn | Bên nhận phản hồi lời mời đã EXPIRED; phản hồi không thành công (EF-12.1) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Lời mời đang PENDING

- Lời mời tài trợ đang ở trạng thái PENDING

#### 5.3 Actor là bên nhận

- Actor là bên nhận (recipient) của lời mời

---

### 6. Postconditions

#### 6.1 Success (Chấp nhận)

- Lời mời chuyển sang trạng thái ACCEPTED
- Deal mới được tạo với trạng thái IN_PROGRESS, liên kết đến lời mời
- Bên gửi được thông báo
- Hai bên sẵn sàng thương thảo (UC-14 đến UC-19)

#### 6.2 Success (Từ chối)

- Lời mời chuyển sang trạng thái DECLINED
- Bên gửi được thông báo kèm lý do (nếu có)
- Bên gửi có thể gửi lời mời mới sau này (khi lời mời cũ không còn PENDING)

#### 6.3 Failure

- Lời mời không thay đổi trạng thái
- Actor được thông báo lỗi cụ thể

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0305: Chỉ lời mời ở trạng thái PENDING mới có thể chấp nhận hoặc từ chối. Lời mời đã ACCEPTED, DECLINED, EXPIRED không thể thay đổi
- BR-0306: Lời mời PENDING tự động hết hạn sau 30 ngày (xử lý bởi System scheduled job)

---

### 10. Additional Information

**Assumptions:**

- Lý do từ chối là tùy chọn nhưng được khuyến khích
- Khi chấp nhận, deal_id mới được tạo tự động — liên kết đến SF-04
- Lời mời hết hạn được xử lý tự động bởi scheduled job (chạy mỗi giờ), thông báo cho cả hai bên

**Related Use Cases:**

- UC-11: Gửi lời mời tài trợ (prerequisite — lời mời phải tồn tại)
- UC-13: Theo dõi danh sách lời mời tài trợ (`<<extend>>` base — UC-12 mở rộng UC-13)
- UC-14: Trao đổi tin nhắn trong deal (sequential — sau khi chấp nhận)
