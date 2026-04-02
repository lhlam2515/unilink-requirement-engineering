# UC-05: Hủy phát hành hồ sơ tài trợ

**Brief Description**
> Organizer hủy phát hành một hồ sơ tài trợ đang ở trạng thái PUBLISHED. Hệ thống kiểm tra điều kiện cho phép hủy theo BR-0109: nếu chưa có Deal liên kết (chưa bắt đầu thương thảo), hồ sơ sẽ được chuyển về trạng thái DRAFT, xóa khỏi chỉ mục tìm kiếm, và gửi thông báo theo BR-0110.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Tài khoản đại diện tổ chức — người hủy phát hành |
| Secondary | System | Kiểm tra điều kiện, xóa chỉ mục, gửi thông báo theo BR-0110 |

---

**Preconditions**

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`
- Hồ sơ tài trợ đang ở trạng thái PUBLISHED
- Organizer là tài khoản đại diện của tổ chức BTC sở hữu hồ sơ

---

**Trigger**
> Organizer nhấn "Hủy phát hành" trên trang quản lý hồ sơ tài trợ.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Organizer | Nhấn "Hủy phát hành" trên hồ sơ đang PUBLISHED |
| 2 | System | Kiểm tra hồ sơ chưa có Deal nào liên kết từ các lời mời tài trợ đã chấp nhận (BR-0109) |
| 3 | System | Kiểm tra hồ sơ không có lời mời tài trợ đang ở trạng thái SENT/PENDING |
| 4 | System | Hiển thị xác nhận "Bạn có chắc muốn hủy phát hành? Hồ sơ sẽ không còn hiển thị cho doanh nghiệp." |
| 5 | Organizer | Xác nhận hủy phát hành |
| 6 | System | Chuyển trạng thái hồ sơ từ PUBLISHED về DRAFT |
| 7 | System | Ghi nhận thời gian hủy phát hành (unpublished_at) |
| 8 | System | Xóa hồ sơ khỏi chỉ mục tìm kiếm |
| 9 | System | Thông báo cho các doanh nghiệp đã bookmark hồ sơ rằng hồ sơ không còn khả dụng (BR-0110) |
| 10 | System | Use case kết thúc thành công — hồ sơ đã ẩn khỏi trang tìm kiếm |

---

**Alternate Flows**

> AF-05.a: Organizer hủy thao tác (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Organizer | Nhấn "Hủy" trong dialog xác nhận |
| 4b | System | Giữ nguyên trạng thái PUBLISHED, không thay đổi gì. Use case kết thúc |

> AF-05.b: Hồ sơ có lời mời SENT/PENDING nhưng chưa có Deal liên kết (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện hồ sơ có lời mời tài trợ đang ở trạng thái SENT/PENDING nhưng chưa có Deal liên kết |
| 3b | System | Hiển thị cảnh báo "Hồ sơ có [N] lời mời đang chờ xử lý. Hủy phát hành sẽ hủy hiệu lực các lời mời này." |
| 3c | Organizer | Xác nhận tiếp tục hủy phát hành |
| 3d | System | Chuyển trạng thái hồ sơ từ PUBLISHED về DRAFT |
| 3e | System | Ghi nhận thời gian hủy phát hành (unpublished_at) |
| 3f | System | Xóa hồ sơ khỏi chỉ mục tìm kiếm |
| 3g | System | Gửi thông báo phản hồi cho bên gửi các lời mời SENT/PENDING, nêu rõ hồ sơ đã hủy phát hành và lời mời không còn hiệu lực (BR-0110) |
| 3h | System | Thông báo cho các doanh nghiệp đã bookmark hồ sơ (BR-0110) |
| 3i | System | Use case kết thúc thành công |

---

**Exception Flows**

> EF-05.1: Hồ sơ đã có Deal liên kết (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ đã có ít nhất một Deal liên kết (đã bắt đầu thương thảo) |
| 2b | System | Từ chối hủy phát hành với thông báo "Không thể hủy phát hành hồ sơ đã bắt đầu thương thảo" |
| 2c | Organizer | Cần chờ hoàn tất hoặc hủy các Deal liên kết trước khi hủy phát hành |

---

**Postconditions**

*Success:*
- Hồ sơ tài trợ chuyển về trạng thái DRAFT
- Hồ sơ không còn xuất hiện trong kết quả tìm kiếm
- Các bookmark liên quan được đánh dấu "không khả dụng"
- Doanh nghiệp đã bookmark được thông báo (BR-0110)
- Nếu có lời mời SENT/PENDING: bên gửi lời mời được thông báo lời mời không còn hiệu lực (BR-0110)

*Failure:*
- Hồ sơ vẫn ở trạng thái PUBLISHED
- Organizer được thông báo lý do không thể hủy phát hành

---

**Business Rules**

- BR-0109: Hồ sơ tài trợ KHÔNG THỂ hủy phát hành nếu đã có ít nhất một Deal liên kết được tạo từ lời mời tài trợ đã chấp nhận (đã bắt đầu thương thảo)
- BR-0110: Khi hủy phát hành thành công, hệ thống PHẢI gửi thông báo phản hồi: (1) cho các doanh nghiệp đã bookmark hồ sơ, (2) cho bên gửi các lời mời tài trợ SENT/PENDING chưa có Deal. Nội dung PHẢI nêu rõ hồ sơ đã hủy phát hành và lời mời không còn hiệu lực

---

**Notes / Assumptions**

- Hồ sơ quay về DRAFT có thể được chỉnh sửa (UC-02, UC-03) và phát hành lại (UC-04)
- Bookmark của doanh nghiệp được giữ lại nhưng đánh dấu "không khả dụng" (BR-0205)
- Liên kết: UC-04, UC-10, UC-11
