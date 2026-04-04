# UC-46: Xem hồ sơ tổ chức công khai

**Brief Description**
> Guest hoặc Authenticated User xem hồ sơ tổ chức công khai để nhận diện tổ chức, kiểm tra trạng thái xác thực, xem tóm tắt uy tín, và truy cập lịch sử công khai phù hợp theo vai trò của tổ chức.

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Guest hoặc Authenticated User | Người truy cập public profile |
| Secondary | System | Truy xuất và hiển thị dữ liệu public |

**Preconditions**

- Hồ sơ tổ chức tồn tại trên hệ thống
- Tổ chức có verification_status = VERIFIED (BR-1101)

**Trigger**
> Người dùng mở đường dẫn public profile của tổ chức hoặc nhấn vào liên kết công khai tới hồ sơ tổ chức.

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Guest / AU | Mở trang hồ sơ tổ chức công khai |
| 2 | System | Truy xuất hồ sơ public và kiểm tra tổ chức có verification_status = VERIFIED |
| 3 | System | Hiển thị tên tổ chức, logo, vai trò (Organizer hoặc Sponsor), trạng thái xác thực, khu vực hoạt động, và mô tả ngắn |
| 4 | System | Hiển thị tóm tắt uy tín: điểm uy tín trung bình, điểm chất lượng trung bình, tổng số đánh giá (read-only, không có link đến SCR-018) |
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
| 4b | System | Hiển thị "Chưa có đánh giá" trong khu vực tóm tắt uy tín |

**Exception Flows**

> EF-46.1: Hồ sơ không tồn tại hoặc tổ chức chưa VERIFIED (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện organization_id không hợp lệ hoặc tổ chức chưa có verification_status = VERIFIED |
| 2b | System | Hiển thị thông báo không khả dụng hoặc trang 404 |
| 2c | Guest / AU | Quay lại trang trước |

**Postconditions**

*Success:*

- Người dùng xem được hồ sơ tổ chức công khai, tóm tắt uy tín, và có thể đi tới lịch sử công khai

*Failure:*

- Người dùng không thể xem hồ sơ và được thông báo lý do

**Business Rules**

- BR-1101: Chỉ tổ chức VERIFIED mới được hiển thị hồ sơ công khai
- BR-1102: Chỉ dữ liệu công khai mới được phép hiển thị
- BR-1104: Mỗi tổ chức chỉ có một vai trò duy nhất — chỉ hiển thị loại lịch sử tương ứng
- BR-1105: Tóm tắt uy tín là read-only, không có điều hướng sang SCR-018 cho Guest

**Notes / Assumptions**

- UC-47 và UC-48 là các use case mở rộng (`<<extend>>`) từ UC-46
- Guest chỉ xem tóm tắt uy tín tại đây; Authenticated User có thể truy cập UC-30 (chi tiết uy tín, SF-07) từ các luồng khác trong hệ thống, không phải từ UC-46
- Mỗi tổ chức chỉ có một vai trò (BR-1104), do đó chỉ có một loại lịch sử được hiển thị
- Liên kết: UC-47 (`<<extend>>`), UC-48 (`<<extend>>`)
