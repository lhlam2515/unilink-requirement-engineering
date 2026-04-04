# UC-46: Xem hồ sơ tổ chức công khai

**Brief Description**
> Authenticated User xem hồ sơ tổ chức công khai để nhận diện tổ chức, kiểm tra trạng thái xác thực, xem tóm tắt uy tín (có thể điều hướng sang chi tiết), và truy cập lịch sử công khai phù hợp theo vai trò duy nhất của tổ chức.

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User | Người xem public profile (đã đăng nhập) |
| Secondary | System | Truy xuất và hiển thị dữ liệu public |

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hồ sơ tổ chức tồn tại trên hệ thống
- Tổ chức có verification_status = VERIFIED (BR-1101)

**Trigger**
> Người dùng mở đường dẫn public profile của tổ chức hoặc nhấn vào liên kết công khai tới hồ sơ tổ chức.

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Mở trang hồ sơ tổ chức công khai |
| 2 | System | Truy xuất hồ sơ public và kiểm tra tổ chức có verification_status = VERIFIED |
| 3 | System | Hiển thị tên tổ chức, logo, vai trò (Organizer hoặc Sponsor), trạng thái xác thực, khu vực hoạt động, và mô tả ngắn |
| 4 | System | Hiển thị tóm tắt uy tín: điểm uy tín trung bình, điểm chất lượng trung bình, tổng số đánh giá, với liên kết "Xem chi tiết uy tín" dẫn sang SCR-018 (UC-30) |
| 5 | System | Hiển thị khu vực lịch sử công khai theo vai trò duy nhất của tổ chức (tab lịch sử hồ sơ tài trợ cho Organizer, hoặc tab lịch sử giao dịch cho Sponsor) |
| 6 | System | Use case kết thúc thành công |

**Alternate Flows**

> AF-46.a: Tổ chức chưa có lịch sử công khai (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện không có lịch sử công khai nào để hiển thị |
| 5b | System | Hiển thị empty state phù hợp |

> AF-46.b: Tổ chức chưa có đánh giá nào (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện tổ chức chưa nhận đánh giá nào |
| 4b | System | Hiển thị "Chưa có đánh giá" trong khu vực tóm tắt uy tín, ẩn liên kết chi tiết |

> AF-46.c: Người dùng chọn xem chi tiết uy tín (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4c | Authenticated User | Nhấn liên kết "Xem chi tiết uy tín" |
| 4d | System | Điều hướng sang SCR-018 — kích hoạt UC-30 (<<extend>>) |

**Exception Flows**

> EF-46.1: Hồ sơ không tồn tại hoặc tổ chức chưa VERIFIED (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện organization_id không hợp lệ hoặc tổ chức chưa có verification_status = VERIFIED |
| 2b | System | Hiển thị thông báo không khả dụng hoặc trang 404 |
| 2c | Authenticated User | Quay lại trang trước |

**Postconditions**

*Success:*
- Người dùng xem được hồ sơ tổ chức công khai, tóm tắt uy tín, và có thể đi tới lịch sử công khai hoặc chi tiết uy tín

*Failure:*
- Người dùng không thể xem hồ sơ và được thông báo lý do

**Business Rules**

- BR-1101: Chỉ tổ chức VERIFIED mới được hiển thị hồ sơ công khai
- BR-1102: Chỉ dữ liệu công khai mới được phép hiển thị
- BR-1104: Mỗi tổ chức chỉ có một vai trò duy nhất — chỉ hiển thị loại lịch sử tương ứng
- BR-1105: Tóm tắt uy tín + liên kết điều hướng sang SCR-018 (UC-30)

**Notes / Assumptions**

- UC-47 và UC-48 là các use case mở rộng (`<<extend>>`) từ UC-46
- UC-30 (Xem điểm uy tín) là use case mở rộng (`<<extend>>`) từ UC-46 — khi người dùng nhấn "Xem chi tiết uy tín"
- Mỗi tổ chức chỉ có một vai trò (BR-1104), do đó chỉ có một loại lịch sử được hiển thị
- Chỉ Authenticated User mới truy cập được public profile
- Liên kết: UC-30 (`<<extend>>`), UC-47 (`<<extend>>`), UC-48 (`<<extend>>`)
