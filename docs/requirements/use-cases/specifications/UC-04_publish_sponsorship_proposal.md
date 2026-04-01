# UC-04: Phát hành hồ sơ tài trợ

**Brief Description**
> Organizer phát hành hồ sơ tài trợ sự kiện đã đầy đủ nội dung. Hệ thống xác thực toàn bộ hồ sơ theo quy tắc nghiệp vụ, chuyển trạng thái từ DRAFT sang PUBLISHED, và lập chỉ mục để hồ sơ hiển thị trên trang tìm kiếm cho doanh nghiệp.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người phát hành hồ sơ |
| Secondary | System | Xác thực tính đầy đủ, chuyển trạng thái, lập chỉ mục tìm kiếm |

---

**Preconditions**

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`
- Hồ sơ tài trợ đang ở trạng thái DRAFT
- Organizer có quyền quản lý hồ sơ này

---

**Trigger**
> Organizer nhấn "Phát hành hồ sơ" trên trang chỉnh sửa hoặc quản lý hồ sơ tài trợ.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Organizer | Nhấn "Phát hành hồ sơ" |
| 2 | System | Xác thực toàn bộ hồ sơ theo BR-0108: kiểm tra tên sự kiện, loại hình, thời gian, địa điểm, quy mô, ngân sách, đối tượng khán giả, hình thức tài trợ, gói tài trợ và quyền lợi |
| 3 | System | Xác thực tất cả gói tài trợ có ít nhất một quyền lợi |
| 4 | System | Chuyển trạng thái hồ sơ từ DRAFT sang PUBLISHED |
| 5 | System | Ghi nhận thời gian phát hành (published_at) |
| 6 | System | Lập chỉ mục hồ sơ cho hệ thống tìm kiếm (SF-02) |
| 7 | System | Hiển thị thông báo xác nhận "Hồ sơ đã được phát hành thành công" |
| 8 | System | Use case kết thúc thành công — hồ sơ xuất hiện trên trang tìm kiếm |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-04.1: Hồ sơ thiếu trường bắt buộc (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ thiếu một hoặc nhiều trường bắt buộc |
| 2b | System | Từ chối phát hành và hiển thị danh sách trường còn thiếu (ví dụ: "Chưa nhập tên sự kiện", "Chưa chọn hình thức tài trợ") |
| 2c | Organizer | Quay lại chỉnh sửa hồ sơ để bổ sung thông tin (UC-02, UC-03) |

> EF-04.2: Gói tài trợ chưa có quyền lợi (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện một hoặc nhiều gói tài trợ chưa có quyền lợi nào |
| 3b | System | Cảnh báo "Gói tài trợ [tên gói] chưa có quyền lợi nào" |
| 3c | Organizer | Quay lại thêm quyền lợi cho gói (UC-03) hoặc xóa gói trống |

> EF-04.3: Chưa có gói tài trợ nào (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ chưa có gói tài trợ nào |
| 2b | System | Từ chối phát hành với thông báo "Cần có ít nhất một gói tài trợ để phát hành" |
| 2c | Organizer | Quay lại tạo gói tài trợ (UC-03) |

---

**Postconditions**

*Success:*
- Hồ sơ tài trợ chuyển sang trạng thái PUBLISHED
- Thời gian phát hành (published_at) được ghi nhận
- Hồ sơ xuất hiện trong kết quả tìm kiếm của doanh nghiệp (SF-02)

*Failure:*
- Hồ sơ vẫn ở trạng thái DRAFT
- Organizer được thông báo chi tiết các trường/điều kiện cần bổ sung

---

**Business Rules**

- BR-0105: Hồ sơ PHẢI có ít nhất một hình thức tài trợ
- BR-0106: Hồ sơ PHẢI có ít nhất một gói tài trợ với giá trị tối thiểu > 0 và slot ≥ 1
- BR-0108: Hồ sơ chỉ được phát hành khi ĐẦY ĐỦ: tên sự kiện, loại hình, thời gian, địa điểm, quy mô, ngân sách, đối tượng khán giả, hình thức tài trợ, gói tài trợ có quyền lợi

---

**Notes / Assumptions**

- Hồ sơ phát hành sẽ hiển thị cho tất cả doanh nghiệp trên trang tìm kiếm
- Organizer vẫn có thể chỉnh sửa hồ sơ sau khi phát hành (UC-02)
- Organizer có thể hủy phát hành thông qua UC-05
- Liên kết: UC-02, UC-03, UC-05, UC-06
