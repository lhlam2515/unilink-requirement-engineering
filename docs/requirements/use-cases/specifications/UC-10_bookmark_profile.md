# UC-10: Lưu hồ sơ quan tâm

**Brief Description**
> Authenticated User (Sponsor hoặc Organizer) lưu bookmark hồ sơ tài trợ sự kiện hoặc hồ sơ doanh nghiệp vào danh sách quan tâm để tham khảo sau. Có thể xem danh sách đã bookmark và xóa bookmark khi không còn quan tâm.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Sponsor hoặc Organizer) | Người lưu/bỏ lưu bookmark |
| Secondary | System | Quản lý bookmark, kiểm tra trùng lặp |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hồ sơ mục tiêu đang ở trạng thái PUBLISHED (sự kiện) hoặc ACTIVE (doanh nghiệp)

---

**Trigger**
> Actor nhấn nút "Bookmark" / "Lưu" trên trang chi tiết hoặc kết quả tìm kiếm.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Nhấn nút "Bookmark" trên hồ sơ quan tâm |
| 2 | System | Kiểm tra actor chưa bookmark hồ sơ này trước đó |
| 3 | System | Tạo bookmark mới với target_type (PROPOSAL hoặc BUSINESS) và target_id |
| 4 | System | Ghi nhận thời gian bookmark (bookmarked_at) |
| 5 | System | Cập nhật giao diện: hiển thị trạng thái "Đã lưu" |
| 6 | System | Use case kết thúc thành công — hồ sơ đã được lưu vào danh sách quan tâm |

---

**Alternate Flows**

> AF-10.a: Actor xem danh sách bookmark (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Truy cập trang "Danh sách quan tâm" trong dashboard |
| 1b | System | Hiển thị danh sách hồ sơ đã bookmark với thông tin tóm tắt |
| 1c | System | Đánh dấu các hồ sơ đã bị hủy phát hành với trạng thái "Hồ sơ không còn khả dụng" |
| 1d | System | Use case kết thúc |

> AF-10.b: Actor bỏ bookmark (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Nhấn "Bỏ lưu" trên hồ sơ đã bookmark |
| 1b | System | Xóa bookmark khỏi danh sách |
| 1c | System | Cập nhật giao diện: hiển thị nút "Bookmark" thay vì "Đã lưu" |
| 1d | System | Use case kết thúc |

---

**Exception Flows**

> EF-10.1: Hồ sơ đã được bookmark trước đó (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện actor đã bookmark hồ sơ này |
| 2b | System | Không tạo bookmark trùng lặp, giữ nguyên trạng thái "Đã lưu" |

---

**Postconditions**

*Success:*
- Bookmark được tạo/xóa thành công
- Danh sách quan tâm được cập nhật

*Failure:*
- Không có thay đổi nào đối với bookmark

---

**Business Rules**

- BR-0205: Mỗi actor chỉ có thể bookmark một hồ sơ MỘT LẦN (không trùng lặp). Bookmark hồ sơ bị hủy phát hành được giữ lại nhưng đánh dấu "không khả dụng"

---

**Notes / Assumptions**

- Sponsor bookmark hồ sơ tài trợ sự kiện, Organizer bookmark hồ sơ doanh nghiệp
- Bookmark hồ sơ bị hủy phát hành không bị xóa tự động — chỉ đánh dấu trạng thái
- Liên kết: UC-06, UC-07, UC-08, UC-09
