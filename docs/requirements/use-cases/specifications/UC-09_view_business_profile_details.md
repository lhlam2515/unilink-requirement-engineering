# UC-09: Xem chi tiết hồ sơ doanh nghiệp

**Brief Description**
> Organizer xem toàn bộ thông tin chi tiết của một doanh nghiệp, bao gồm tên, logo, lĩnh vực hoạt động, khu vực, đối tượng khách hàng, mục tiêu tài trợ, và ngân sách tài trợ dự kiến.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người xem hồ sơ doanh nghiệp |
| Secondary | System | Truy xuất và hiển thị dữ liệu hồ sơ |

---

**Preconditions**

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`
- Doanh nghiệp có hồ sơ đang ở trạng thái ACTIVE

---

**Trigger**
> Organizer nhấn vào một doanh nghiệp trong danh sách kết quả tìm kiếm (UC-07) hoặc truy cập bằng đường dẫn trực tiếp.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Organizer | Chọn một doanh nghiệp từ danh sách kết quả tìm kiếm |
| 2 | System | Truy xuất hồ sơ doanh nghiệp và kiểm tra trạng thái ACTIVE |
| 3 | System | Hiển thị thông tin doanh nghiệp: tên, logo, lĩnh vực hoạt động, khu vực |
| 4 | System | Hiển thị thông tin tài trợ: đối tượng khách hàng, mục tiêu tài trợ (Marketing/CSR), ngân sách tài trợ dự kiến |
| 5 | System | Use case kết thúc thành công — organizer thấy đầy đủ thông tin doanh nghiệp |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-09.1: Hồ sơ doanh nghiệp không ACTIVE (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ doanh nghiệp ở trạng thái INACTIVE hoặc không tồn tại |
| 2b | System | Hiển thị thông báo "Hồ sơ doanh nghiệp không tồn tại hoặc đã ngừng hoạt động" |
| 2c | Organizer | Quay lại trang tìm kiếm |

---

**Postconditions**

*Success:*
- Organizer xem được toàn bộ thông tin chi tiết doanh nghiệp

*Failure:*
- Organizer không thể xem hồ sơ và được thông báo lý do

---

**Business Rules**

- BR-0204: Actor chỉ có thể xem chi tiết hồ sơ ACTIVE. Truy cập hồ sơ INACTIVE bị từ chối

---

**Notes / Assumptions**

- Hồ sơ doanh nghiệp (BusinessProfile) được quản lý bởi feature ngoài phạm vi quy trình này
- Organizer có thể bookmark doanh nghiệp từ trang chi tiết (UC-10)
- Organizer có thể gửi lời mời tài trợ từ trang chi tiết (UC-11)
- Liên kết: UC-07, UC-10, UC-11
