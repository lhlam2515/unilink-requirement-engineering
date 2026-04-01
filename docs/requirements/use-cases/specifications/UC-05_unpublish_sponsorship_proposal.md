# UC-05: Hủy phát hành hồ sơ tài trợ

**Brief Description**
> Organizer hủy phát hành một hồ sơ tài trợ đang ở trạng thái PUBLISHED, chuyển về trạng thái DRAFT. Hệ thống xóa hồ sơ khỏi chỉ mục tìm kiếm và thông báo cho các doanh nghiệp đã bookmark hồ sơ.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người hủy phát hành |
| Secondary | System | Kiểm tra điều kiện, xóa chỉ mục, gửi thông báo |

---

**Preconditions**

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`
- Hồ sơ tài trợ đang ở trạng thái PUBLISHED
- Organizer có quyền quản lý hồ sơ này

---

**Trigger**
> Organizer nhấn "Hủy phát hành" trên trang quản lý hồ sơ tài trợ.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Organizer | Nhấn "Hủy phát hành" trên hồ sơ đang PUBLISHED |
| 2 | System | Kiểm tra hồ sơ không có lời mời tài trợ đang ở trạng thái PENDING |
| 3 | System | Hiển thị xác nhận "Bạn có chắc muốn hủy phát hành? Hồ sơ sẽ không còn hiển thị cho doanh nghiệp." |
| 4 | Organizer | Xác nhận hủy phát hành |
| 5 | System | Chuyển trạng thái hồ sơ từ PUBLISHED về DRAFT |
| 6 | System | Ghi nhận thời gian hủy phát hành (unpublished_at) |
| 7 | System | Xóa hồ sơ khỏi chỉ mục tìm kiếm |
| 8 | System | Thông báo cho các doanh nghiệp đã bookmark hồ sơ rằng hồ sơ không còn khả dụng |
| 9 | System | Use case kết thúc thành công — hồ sơ đã ẩn khỏi trang tìm kiếm |

---

**Alternate Flows**

> AF-05.a: Organizer hủy thao tác (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Organizer | Nhấn "Hủy" trong dialog xác nhận |
| 3b | System | Giữ nguyên trạng thái PUBLISHED, không thay đổi gì. Use case kết thúc |

---

**Exception Flows**

> EF-05.1: Hồ sơ có lời mời tài trợ đang PENDING (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ có lời mời tài trợ đang ở trạng thái PENDING |
| 2b | System | Từ chối hủy phát hành với thông báo "Không thể hủy phát hành khi còn [N] lời mời đang chờ xử lý" |
| 2c | Organizer | Cần chờ hoặc xử lý các lời mời PENDING trước khi hủy phát hành |

---

**Postconditions**

*Success:*
- Hồ sơ tài trợ chuyển về trạng thái DRAFT
- Hồ sơ không còn xuất hiện trong kết quả tìm kiếm
- Các bookmark liên quan được đánh dấu "không khả dụng"
- Doanh nghiệp đã bookmark được thông báo

*Failure:*
- Hồ sơ vẫn ở trạng thái PUBLISHED
- Organizer được thông báo lý do không thể hủy phát hành

---

**Business Rules**

- BR-0109: Hồ sơ tài trợ KHÔNG THỂ hủy phát hành nếu có lời mời tài trợ đang ở trạng thái PENDING liên kết với hồ sơ

---

**Notes / Assumptions**

- Hồ sơ quay về DRAFT có thể được chỉnh sửa (UC-02, UC-03) và phát hành lại (UC-04)
- Bookmark của doanh nghiệp được giữ lại nhưng đánh dấu "không khả dụng" (BR-0205)
- Liên kết: UC-04, UC-10, UC-11
