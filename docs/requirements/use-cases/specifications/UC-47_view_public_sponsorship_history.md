# UC-47: Xem lịch sử hồ sơ tài trợ công khai

**Brief Description**
> Authenticated User xem danh sách lịch sử công khai các hồ sơ tài trợ đã từng thực hiện của một tổ chức theo vai trò Organizer, để đánh giá năng lực tổ chức và mức độ hoạt động thực tế.

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User | Người xem lịch sử public |
| Secondary | System | Truy xuất và hiển thị lịch sử |

**Preconditions**

- Tổ chức có verification_status = VERIFIED (BR-1101)
- Tổ chức có vai trò Organizer (BR-1104)
- UC-46 đã hiển thị thành công (public profile đang mở)

**Trigger**
> Người dùng chọn tab "Lịch sử tài trợ" trên public profile của tổ chức.

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Chọn tab lịch sử tài trợ |
| 2 | System | Truy xuất danh sách lịch sử tài trợ công khai của tổ chức |
| 3 | System | Áp dụng quy tắc lọc public: chỉ hiển thị các mục đủ điều kiện công khai (BR-1102, BR-1106) |
| 4 | System | Hiển thị tối đa 5 mục theo thứ tự mới nhất trước (BR-1103) |
| 5 | System | Hiển thị các trường tóm tắt: tên sự kiện, năm, trạng thái công khai, nhãn tóm tắt, và thời điểm phát hành/lưu trữ |
| 6 | System | Mỗi mục có thể hiển thị liên kết sang view chi tiết sự kiện tương ứng (nếu có và đủ điều kiện public) |
| 7 | System | Use case kết thúc thành công |

**Alternate Flows**

> AF-47.a: Không có lịch sử công khai (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện không có mục lịch sử hợp lệ |
| 2b | System | Hiển thị empty state "Chưa có lịch sử tài trợ công khai" |

> AF-47.b: Có nhiều hơn 5 mục — phân trang (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Authenticated User | Nhấn "Xem thêm" (load more) |
| 4b | System | Tải và hiển thị 5 mục tiếp theo |

> AF-47.c: Áp dụng bộ lọc (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Chọn bộ lọc theo năm hoặc trạng thái |
| 1b | System | Áp dụng bộ lọc và hiển thị kết quả phù hợp |
| 1c | System | Nếu không có kết quả, hiển thị empty state phù hợp |

**Exception Flows**

> EF-47.1: Tổ chức không phải Organizer hoặc chưa VERIFIED (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ không thuộc vai trò Organizer hoặc chưa VERIFIED |
| 2b | System | Hiển thị lỗi không khả dụng hoặc 404 |

> **Ghi chú**: Exception này chỉ xảy ra khi truy cập trực tiếp URL tab, vì UC-46 đã validate tổ chức.

**Postconditions**

*Success:*

- Người dùng xem được lịch sử tài trợ công khai của tổ chức

*Failure:*

- Người dùng không xem được lịch sử và nhận thông báo phù hợp

**Business Rules**

- BR-1101: Chỉ tổ chức VERIFIED mới hiển thị
- BR-1102: Không hiển thị draft, thương thảo nội bộ, hoặc dữ liệu nhạy cảm
- BR-1103: Hiển thị tối đa 5 mục gần nhất, page size cố định, hỗ trợ load more
- BR-1104: Chỉ hiển thị lịch sử public phù hợp với vai trò Organizer
- BR-1106: Không hiển thị giá trị tài trợ, điều khoản hợp đồng, số lượng deal

**Notes / Assumptions**

- Đây là lịch sử public, không phải danh sách đầy đủ của hệ thống nội bộ
- Mỗi mục có thể liên kết sang view chi tiết sự kiện (nếu có và đủ điều kiện public)
- Liên kết: UC-46 (base UC — `<<extend>>`)
