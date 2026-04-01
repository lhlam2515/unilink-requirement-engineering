# UC-08: Xem chi tiết hồ sơ tài trợ sự kiện

**Brief Description**
> Sponsor xem toàn bộ thông tin chi tiết của một hồ sơ tài trợ sự kiện đã phát hành, bao gồm thông tin sự kiện, gói tài trợ, quyền lợi nhà tài trợ, và hình ảnh nhận diện.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Sponsor (Doanh nghiệp) | Người xem hồ sơ |
| Secondary | System | Truy xuất và hiển thị dữ liệu hồ sơ |

---

**Preconditions**

- Sponsor đã đăng nhập vào hệ thống với vai trò `sponsor`
- Hồ sơ tài trợ đang ở trạng thái PUBLISHED

---

**Trigger**
> Sponsor nhấn vào một sự kiện trong danh sách kết quả tìm kiếm (UC-06) hoặc truy cập bằng đường dẫn trực tiếp.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Sponsor | Chọn một sự kiện từ danh sách kết quả tìm kiếm |
| 2 | System | Truy xuất hồ sơ tài trợ và kiểm tra trạng thái PUBLISHED |
| 3 | System | Hiển thị thông tin cơ bản: tên sự kiện, loại hình, thời gian, địa điểm |
| 4 | System | Hiển thị thông tin chi tiết: quy mô, ngân sách, đối tượng khán giả, nội dung chương trình |
| 5 | System | Hiển thị hình ảnh nhận diện sự kiện (banner, thumbnail) |
| 6 | System | Hiển thị hình thức tài trợ được chấp nhận |
| 7 | System | Hiển thị danh sách gói tài trợ với quyền lợi tương ứng |
| 8 | System | Use case kết thúc thành công — sponsor thấy đầy đủ thông tin hồ sơ |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-08.1: Hồ sơ không ở trạng thái PUBLISHED (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ ở trạng thái DRAFT hoặc không tồn tại |
| 2b | System | Hiển thị thông báo "Hồ sơ không tồn tại hoặc chưa được phát hành" (HTTP 404) |
| 2c | Sponsor | Quay lại trang tìm kiếm |

---

**Postconditions**

*Success:*
- Sponsor xem được toàn bộ thông tin chi tiết hồ sơ tài trợ

*Failure:*
- Sponsor không thể xem hồ sơ và được thông báo lý do

---

**Business Rules**

- BR-0204: Actor chỉ có thể xem chi tiết hồ sơ đã phát hành. Truy cập hồ sơ DRAFT bằng đường dẫn trực tiếp bị từ chối

---

**Notes / Assumptions**

- Sponsor có thể bookmark hồ sơ từ trang chi tiết (UC-10)
- Sponsor có thể gửi lời mời tài trợ từ trang chi tiết (UC-11)
- Liên kết: UC-06, UC-10, UC-11
