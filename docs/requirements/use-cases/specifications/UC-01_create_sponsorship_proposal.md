# UC-01: Tạo hồ sơ tài trợ sự kiện

**Brief Description**
> Organizer khởi tạo một hồ sơ tài trợ sự kiện mới trên hệ thống. Hệ thống tạo hồ sơ với trạng thái ban đầu DRAFT, gán mã định danh duy nhất, và chuyển organizer đến trang chỉnh sửa để tiếp tục soạn thảo nội dung.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người khởi tạo hồ sơ tài trợ |
| Secondary | System | Gán mã định danh, ghi nhận thời gian tạo |

---

**Preconditions**

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`
- Organizer là tài khoản đại diện duy nhất của một tổ chức BTC hợp lệ trên hệ thống

---

**Trigger**
> Organizer chọn chức năng "Tạo hồ sơ tài trợ mới" từ dashboard hoặc menu quản lý hồ sơ.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Organizer | Chọn chức năng "Tạo hồ sơ tài trợ mới" |
| 2 | System | Tạo hồ sơ mới với trạng thái DRAFT |
| 3 | System | Gán proposal_id (UUID) duy nhất cho hồ sơ |
| 4 | System | Gán hồ sơ cho tổ chức BTC mà tài khoản organizer đại diện |
| 5 | System | Ghi nhận thời gian tạo (created_at) và người tạo (created_by) |
| 6 | System | Chuyển organizer đến trang chỉnh sửa hồ sơ tài trợ |
| 7 | System | Use case kết thúc thành công — hồ sơ đã được khởi tạo ở trạng thái DRAFT |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-01.1: Không thể tạo hồ sơ do lỗi hệ thống (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện lỗi khi khởi tạo hồ sơ (database, service không khả dụng) |
| 2b | System | Hiển thị thông báo lỗi "Không thể tạo hồ sơ tài trợ. Vui lòng thử lại sau." |
| 2c | Organizer | Có thể thử tạo lại hoặc quay về dashboard |

---

**Postconditions**

*Success:*

- Hồ sơ tài trợ mới được tạo với trạng thái DRAFT
- Hồ sơ có proposal_id duy nhất và được gắn với tổ chức BTC
- Thời gian tạo và danh tính người tạo được ghi nhận
- Organizer đang ở trang chỉnh sửa hồ sơ, sẵn sàng soạn thảo nội dung

*Failure:*

- Không có hồ sơ nào được tạo
- Organizer được thông báo về lỗi và ở lại trang trước đó

---

**Business Rules**

- BR-0101: Mỗi hồ sơ tài trợ khi khởi tạo PHẢI có trạng thái ban đầu là DRAFT
- BR-0102: Mỗi hồ sơ tài trợ PHẢI được gán cho đúng một tổ chức BTC. Một BTC có thể có nhiều hồ sơ tài trợ đồng thời

---

**Notes / Assumptions**

- Hồ sơ được tạo ở trạng thái DRAFT — chưa có nội dung chi tiết tại bước này (cho phép tạo bản nháp trống)
- Organizer tiếp tục soạn thảo nội dung thông qua UC-02 (Chỉnh sửa nội dung hồ sơ tài trợ) và UC-03 (Quản lý gói tài trợ)
- Liên kết: UC-02, UC-03, UC-04
