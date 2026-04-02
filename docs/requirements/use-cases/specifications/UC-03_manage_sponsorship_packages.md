# UC-03: Quản lý gói tài trợ

**Brief Description**
> Organizer tạo, chỉnh sửa và quản lý các gói tài trợ (sponsorship packages) trong hồ sơ tài trợ sự kiện. Mỗi gói bao gồm cấp độ tài trợ, giá trị tối thiểu, số lượng slot, và danh sách quyền lợi nhà tài trợ tương ứng.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người định nghĩa gói tài trợ và quyền lợi |
| Secondary | System | Xác thực dữ liệu, gán mã định danh gói |

---

**Preconditions**

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`
- Hồ sơ tài trợ đã tồn tại và đang ở trạng thái DRAFT hoặc PUBLISHED
- Organizer là tài khoản đại diện duy nhất của tổ chức BTC sở hữu hồ sơ

---

**Trigger**
> Organizer chọn "Thêm gói tài trợ" hoặc chọn chỉnh sửa gói tài trợ hiện có trên trang quản lý hồ sơ tài trợ.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Organizer | Chọn "Thêm gói tài trợ" trên trang chỉnh sửa hồ sơ |
| 2 | System | Hiển thị form tạo gói tài trợ |
| 3 | Organizer | Nhập thông tin gói: tên gói, cấp độ (Title/Co-Sponsor/Associate/Technical/Partner), mô tả, giá trị tài trợ tối thiểu, số lượng slot khả dụng |
| 4 | System | Xác thực: tên gói duy nhất trong hồ sơ, giá trị tối thiểu > 0, số slot ≥ 1 |
| 5 | System | Tạo gói tài trợ với package_id duy nhất |
| 6 | Organizer | Chọn "Thêm quyền lợi" cho gói vừa tạo |
| 7 | System | Hiển thị form thêm quyền lợi nhà tài trợ |
| 8 | Organizer | Nhập quyền lợi: nhóm quyền lợi (BRANDING/STAGE/DIGITAL/ENGAGEMENT), tiêu đề, mô tả cam kết thực hiện |
| 9 | System | Lưu quyền lợi vào gói tài trợ |
| 10 | Organizer | Lặp lại Steps 6-9 để thêm các quyền lợi khác (nếu cần) |
| 11 | System | Use case kết thúc thành công — gói tài trợ và quyền lợi đã được tạo |

---

**Alternate Flows**

> AF-03.a: Organizer chỉnh sửa gói tài trợ hiện có (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Organizer | Chọn gói tài trợ hiện có từ danh sách để chỉnh sửa |
| 1b | System | Hiển thị form chỉnh sửa với thông tin hiện tại của gói |
| 1c | Organizer | Cập nhật thông tin gói (tên, cấp độ, giá trị, slot) |
| 1d | System | Xác thực và lưu thay đổi. Tiếp tục tại Step 6 nếu muốn chỉnh sửa quyền lợi |

> AF-03.b: Organizer xóa gói tài trợ (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Organizer | Chọn "Xóa" trên gói tài trợ hiện có |
| 1b | System | Hiển thị xác nhận "Bạn có chắc chắn muốn xóa gói [tên gói]? Tất cả quyền lợi liên quan sẽ bị xóa." |
| 1c | Organizer | Xác nhận xóa |
| 1d | System | Xóa gói tài trợ và toàn bộ quyền lợi liên quan. Use case kết thúc |

> AF-03.c: Organizer chỉnh sửa hoặc xóa quyền lợi (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | Organizer | Chọn chỉnh sửa hoặc xóa một quyền lợi trong gói |
| 6b | System | Nếu chỉnh sửa: hiển thị form với thông tin hiện tại, cho phép cập nhật |
| 6c | System | Nếu xóa: yêu cầu xác nhận và xóa quyền lợi khỏi gói |
| 6d | System | Lưu thay đổi. Use case kết thúc hoặc tiếp tục chỉnh sửa |

---

**Exception Flows**

> EF-03.1: Tên gói tài trợ trùng lặp (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện tên gói đã tồn tại trong hồ sơ này |
| 4b | System | Hiển thị thông báo lỗi "Tên gói tài trợ đã tồn tại. Vui lòng chọn tên khác." |
| 4c | Organizer | Nhập tên gói khác và thử lưu lại |

> EF-03.2: Giá trị tối thiểu hoặc slot không hợp lệ (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện giá trị tối thiểu ≤ 0 hoặc số slot < 1 |
| 4b | System | Hiển thị thông báo lỗi "Giá trị tối thiểu phải lớn hơn 0" hoặc "Số lượng slot phải ít nhất là 1" |
| 4c | Organizer | Chỉnh sửa giá trị và thử lưu lại |

---

**Postconditions**

*Success:*
- Gói tài trợ với package_id duy nhất đã được tạo/cập nhật trong hồ sơ
- Quyền lợi nhà tài trợ đã được gắn vào gói
- Hồ sơ đã cập nhật danh sách gói tài trợ

*Failure:*
- Gói tài trợ không được tạo hoặc cập nhật
- Organizer được thông báo lỗi cụ thể

---

**Business Rules**

- BR-0106: Mỗi gói tài trợ PHẢI có tên duy nhất trong phạm vi hồ sơ, giá trị tối thiểu > 0, và số slot ≥ 1. Hồ sơ PHẢI có ít nhất một gói trước khi phát hành
- BR-0107: Mỗi quyền lợi PHẢI thuộc một trong các nhóm: BRANDING, STAGE, DIGITAL, ENGAGEMENT. Mỗi quyền lợi PHẢI có mô tả cam kết thực hiện cụ thể

---

**Notes / Assumptions**

- Organizer có thể tạo nhiều gói tài trợ cho một hồ sơ
- Gói tài trợ chưa có quyền lợi sẽ được cảnh báo khi phát hành (UC-04)
- Cấp độ gói: Title Sponsor, Co-Sponsor, Associate Sponsor, Technical Sponsor, Sponsorship Partner
- Liên kết: UC-01, UC-02, UC-04
