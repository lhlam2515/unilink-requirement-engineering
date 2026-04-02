# UC-11: Gửi lời mời tài trợ

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) gửi lời mời tài trợ cho đối tác dựa trên một hồ sơ tài trợ sự kiện cụ thể. Lời mời bao gồm tin nhắn giới thiệu và hệ thống tự động gửi thông báo cho bên nhận qua in-app và email.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người gửi lời mời |
| Secondary | System | Xác thực, tạo lời mời, gửi thông báo tự động |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hồ sơ tài trợ liên quan đang ở trạng thái PUBLISHED
- Chưa tồn tại lời mời PENDING giữa hai bên cho cùng hồ sơ

---

**Trigger**
> Actor nhấn "Gửi lời mời tài trợ" từ trang chi tiết hồ sơ đối tác hoặc hồ sơ sự kiện.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Nhấn "Gửi lời mời tài trợ" trên trang chi tiết hồ sơ |
| 2 | System | Hiển thị form gửi lời mời: tin nhắn giới thiệu (bắt buộc), gói tài trợ ưu tiên (tùy chọn) |
| 3 | Authenticated User | Nhập tin nhắn giới thiệu (tối thiểu 20 ký tự) |
| 4 | Authenticated User | Chọn gói tài trợ ưu tiên (nếu muốn) |
| 5 | Authenticated User | Nhấn "Gửi" |
| 6 | System | Xác thực: hồ sơ tài trợ đang PUBLISHED, chưa có lời mời PENDING cho cặp này, tin nhắn ≥ 20 ký tự |
| 7 | System | Tạo lời mời với trạng thái PENDING, ghi nhận sent_at |
| 8 | System | Gửi thông báo in-app cho bên nhận (trong vòng 30 giây) |
| 9 | System | Gửi email thông báo cho bên nhận (trong vòng 5 phút) — bao gồm tên bên gửi, tên sự kiện, tin nhắn giới thiệu, liên kết đến chi tiết lời mời |
| 10 | System | Hiển thị xác nhận "Lời mời tài trợ đã được gửi thành công" |
| 11 | System | Use case kết thúc thành công |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-11.1: Đã tồn tại lời mời PENDING cho cặp này (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện đã tồn tại lời mời PENDING giữa hai bên cho cùng hồ sơ |
| 6b | System | Từ chối với thông báo "Đã tồn tại lời mời đang chờ xử lý cho đối tác này" |
| 6c | Authenticated User | Có thể xem lời mời hiện có hoặc chờ bên nhận phản hồi |

> EF-11.2: Tin nhắn giới thiệu quá ngắn (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện tin nhắn giới thiệu dưới 20 ký tự |
| 6b | System | Từ chối với thông báo "Tin nhắn giới thiệu phải có ít nhất 20 ký tự" |
| 6c | Authenticated User | Nhập tin nhắn dài hơn và thử gửi lại |

> EF-11.3: Hồ sơ tài trợ không còn PUBLISHED (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện hồ sơ tài trợ đã bị hủy phát hành (DRAFT) |
| 6b | System | Từ chối với thông báo "Hồ sơ tài trợ không còn khả dụng" |

---

**Postconditions**

*Success:*
- Lời mời tài trợ được tạo với trạng thái PENDING
- Bên nhận được thông báo qua in-app (≤ 30 giây) và email (≤ 5 phút)
- Lời mời xuất hiện trong danh sách lời mời đã gửi của actor

*Failure:*
- Lời mời không được tạo
- Actor được thông báo lỗi cụ thể

---

**Business Rules**

- BR-0301: Lời mời chỉ gửi đến hồ sơ tài trợ ở trạng thái PUBLISHED
- BR-0302: Mỗi cặp (proposal_id + target_business_id) chỉ có MỘT lời mời PENDING tại một thời điểm
- BR-0303: Tin nhắn giới thiệu BẮT BUỘC, tối thiểu 20 ký tự
- BR-0304: Thông báo gửi qua in-app (≤ 30 giây) và email (≤ 5 phút)

---

**Notes / Assumptions**

- Cả Organizer và Sponsor đều có thể gửi lời mời (hai chiều)
- Lời mời PENDING sẽ tự động hết hạn sau 30 ngày (1 tháng) kể từ ngày gửi (xử lý tự động bởi System, xem BR-0306)
- Liên kết: UC-08, UC-09, UC-12, UC-13
