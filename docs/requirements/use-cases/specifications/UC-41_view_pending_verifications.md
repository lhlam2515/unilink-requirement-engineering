# UC-41: Xem danh sách hồ sơ chờ kiểm duyệt

**Brief Description**
> Admin xem danh sách tất cả hồ sơ xác thực tổ chức đang chờ kiểm duyệt. Danh sách hiển thị thông tin tóm tắt, sắp xếp theo thứ tự ưu tiên (hồ sơ gửi trước xử lý trước), và hỗ trợ lọc theo vai trò cùng tìm kiếm theo tên tổ chức.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người kiểm duyệt hồ sơ tổ chức |
| Secondary | System | Truy xuất và hiển thị danh sách hồ sơ |

---

**Preconditions**

- Admin đã đăng nhập vào hệ thống với vai trò `admin`
- Có ít nhất một hồ sơ xác thực với status = PENDING trong hệ thống

---

**Trigger**
> Admin truy cập trang "Kiểm duyệt hồ sơ" từ admin panel.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Admin | Truy cập trang "Kiểm duyệt hồ sơ" |
| 2 | System | Truy xuất danh sách hồ sơ xác thực có status = PENDING |
| 3 | System | Sắp xếp danh sách theo submitted_at ASC (hồ sơ gửi trước hiển thị trước) |
| 4 | System | Hiển thị danh sách với thông tin tóm tắt: tên tổ chức, vai trò (Organizer/Sponsor), ngày gửi, thời gian chờ xử lý |
| 5 | System | Hiển thị tổng số hồ sơ pending và thời gian chờ trung bình |
| 6 | System | Use case kết thúc thành công — danh sách hồ sơ đã được hiển thị |

---

**Alternate Flows**

> AF-41.a: Lọc theo vai trò (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Admin | Chọn bộ lọc vai trò: Organizer, Sponsor, hoặc Tất cả |
| 4b | System | Áp dụng bộ lọc và hiển thị chỉ hồ sơ thuộc vai trò được chọn |

> AF-41.b: Tìm kiếm theo tên tổ chức (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4c | Admin | Nhập từ khóa tìm kiếm (tên tổ chức) |
| 4d | System | Hiển thị hồ sơ có tên tổ chức chứa từ khóa |

> AF-41.c: Phân trang khi danh sách dài (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4e | System | Hiển thị tối đa 20 hồ sơ mỗi trang (BR-1001) |
| 4f | Admin | Chuyển trang để xem thêm hồ sơ |

> AF-41.d: Không có hồ sơ pending (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Không tìm thấy hồ sơ nào với status = PENDING |
| 2b | System | Hiển thị thông báo "Không có hồ sơ nào đang chờ kiểm duyệt" |

---

**Exception Flows**

> EF-41.1: Lỗi truy xuất dữ liệu (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2c | System | Phát hiện lỗi khi truy xuất danh sách (database, service không khả dụng) |
| 2d | System | Hiển thị thông báo lỗi "Không thể tải danh sách. Vui lòng thử lại sau." |

---

**Postconditions**

*Success:*

- Admin đã xem danh sách hồ sơ chờ kiểm duyệt
- Admin có thể chọn hồ sơ để xem chi tiết (UC-42)

*Failure:*

- Danh sách không hiển thị
- Admin được thông báo về lỗi

---

**Business Rules**

- BR-1001: Danh sách sắp xếp theo submitted_at ASC (FIFO). Phân trang 20 hồ sơ/trang

---

**Notes / Assumptions**

- Danh sách chỉ hiển thị hồ sơ PENDING — hồ sơ đã xử lý (APPROVED, REJECTED, INFO_REQUIRED) có thể xem qua bộ lọc riêng hoặc lịch sử
- Admin nhấn vào một hồ sơ để xem chi tiết (UC-42) — đây là include relationship
- Liên kết: UC-42 (<<include>> xem chi tiết), UC-40 (hồ sơ đã gửi)
