# UC-02: Chỉnh sửa nội dung hồ sơ tài trợ

**Brief Description**
> Organizer nhập và chỉnh sửa các thông tin nội dung của hồ sơ tài trợ sự kiện bao gồm: thông tin cơ bản sự kiện, hình ảnh nhận diện, thông tin chi tiết, và hình thức tài trợ được chấp nhận. Mỗi nội dung được xác thực trước khi lưu.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người soạn thảo nội dung hồ sơ |
| Secondary | System | Xác thực dữ liệu, lưu trữ, xử lý hình ảnh |

---

**Preconditions**

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`
- Hồ sơ tài trợ đã tồn tại và đang ở trạng thái DRAFT hoặc PUBLISHED
- Organizer là tài khoản đại diện duy nhất của tổ chức BTC sở hữu hồ sơ

---

**Trigger**
> Organizer mở trang chỉnh sửa hồ sơ tài trợ và chọn mục nội dung muốn cập nhật.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Organizer | Mở trang chỉnh sửa hồ sơ tài trợ |
| 2 | System | Hiển thị form chỉnh sửa với các mục: Thông tin cơ bản, Hình ảnh nhận diện, Thông tin chi tiết, Hình thức tài trợ |
| 3 | Organizer | Nhập thông tin cơ bản: tên chương trình, loại hình sự kiện, thời gian bắt đầu/kết thúc, địa điểm tổ chức |
| 4 | System | Xác thực thông tin cơ bản: tên không rỗng, thời gian phải trong tương lai, các trường bắt buộc đầy đủ |
| 5 | Organizer | Nhập thông tin chi tiết: quy mô dự kiến, ngân sách dự kiến, đối tượng khán giả, nội dung chương trình |
| 6 | System | Xác thực thông tin chi tiết: quy mô và ngân sách phải là số dương |
| 7 | Organizer | Chọn hình thức tài trợ được chấp nhận: tiền mặt (CASH), hiện vật (IN_KIND), hoặc kết hợp (COMBINED) |
| 8 | System | Ghi nhận hình thức tài trợ đã chọn |
| 9 | Organizer | Lưu toàn bộ nội dung đã nhập |
| 10 | System | Lưu hồ sơ với thông tin đã cập nhật, ghi nhận updated_at |
| 11 | System | Nếu hồ sơ đang PUBLISHED, cập nhật thông tin trên trang tìm kiếm |
| 12 | System | Use case kết thúc thành công — hồ sơ đã được cập nhật nội dung |

---

**Alternate Flows**

> AF-02.a: Organizer tải lên hình ảnh nhận diện sự kiện (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | Organizer | Chọn mục "Hình ảnh nhận diện" và tải lên file hình ảnh (banner hoặc thumbnail) |
| 2b | System | Xác thực định dạng file (chỉ chấp nhận JPEG, PNG, WebP) và kích thước (≤ 5MB) |
| 2c | System | Lưu trữ ảnh gốc và tạo phiên bản thumbnail tự động |
| 2d | System | Trả về image_url và thumbnail_url, hiển thị preview. Tiếp tục tại Step 3 |

> AF-02.b: Organizer chỉ cập nhật một phần nội dung (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Organizer | Chỉ nhập hoặc chỉnh sửa một số mục (không bắt buộc nhập tất cả trong một phiên) |
| 3b | System | Lưu các mục đã nhập, giữ nguyên các mục chưa thay đổi |
| 3c | System | Ghi nhận updated_at. Use case kết thúc |

---

**Exception Flows**

> EF-02.1: Thời gian tổ chức sự kiện nằm trong quá khứ (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện event_date_start hoặc event_date_end nằm trong quá khứ |
| 4b | System | Hiển thị thông báo lỗi "Thời gian tổ chức phải trong tương lai" |
| 4c | Organizer | Chỉnh sửa lại thời gian tổ chức và thử lưu lại |

> EF-02.2: Giá trị số không hợp lệ (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện quy mô hoặc ngân sách không phải số dương (≤ 0 hoặc không phải số) |
| 6b | System | Hiển thị thông báo lỗi "Quy mô/Ngân sách phải là số dương" |
| 6c | Organizer | Chỉnh sửa lại giá trị và thử lưu lại |

> EF-02.3: File hình ảnh không hợp lệ (triggered at Step 2b trong AF-02.a)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2b-a | System | Phát hiện file không đúng định dạng (ví dụ: PDF, BMP) hoặc vượt quá 5MB |
| 2b-b | System | Hiển thị thông báo lỗi "Chỉ chấp nhận JPEG, PNG, WebP" hoặc "File vượt quá giới hạn 5MB" |
| 2b-c | Organizer | Chọn file khác và thử tải lên lại |

---

**Postconditions**

*Success:*
- Hồ sơ tài trợ đã được cập nhật với nội dung mới
- Thời gian cập nhật (updated_at) được ghi nhận
- Nếu hồ sơ đang PUBLISHED, thông tin mới được phản ánh trên trang tìm kiếm

*Failure:*
- Nội dung hồ sơ không thay đổi
- Organizer được thông báo lỗi xác thực cụ thể để sửa chữa

---

**Business Rules**

- BR-0103: Các trường số (quy mô, ngân sách) PHẢI là số dương. Thời gian tổ chức PHẢI nằm trong tương lai
- BR-0104: File hình ảnh PHẢI có định dạng JPEG, PNG, hoặc WebP. Kích thước tối đa 5MB
- BR-0105: Hồ sơ PHẢI có ít nhất một hình thức tài trợ trước khi phát hành (kiểm tra tại UC-04)

---

**Notes / Assumptions**

- Organizer không bắt buộc phải nhập tất cả nội dung trong một phiên làm việc — có thể lưu dần
- Hình ảnh nhận diện là tùy chọn (SHOULD, không phải MUST)
- Liên kết: UC-01 (tạo hồ sơ trước), UC-03 (quản lý gói tài trợ), UC-04 (phát hành)
