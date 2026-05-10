# Use-Case Specification: UC-02 — Chỉnh sửa nội dung hồ sơ tài trợ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người soạn thảo nội dung hồ sơ |
| Secondary | System | Xác thực dữ liệu, lưu trữ, xử lý hình ảnh |

---

### 1. Brief Description

> Organizer nhập và chỉnh sửa các thông tin nội dung của hồ sơ tài trợ sự kiện bao gồm: thông tin cơ bản sự kiện, hình ảnh nhận diện, thông tin chi tiết, và hình thức tài trợ được chấp nhận. Mỗi nội dung được xác thực trước khi lưu.

---

### 2. Flow of Events

**Trigger**
> Organizer mở trang chỉnh sửa hồ sơ tài trợ và chọn mục nội dung muốn cập nhật.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
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

#### 2.2 Alternate Flows

##### AF-02.a: Organizer tải lên hình ảnh nhận diện sự kiện
>
> *Triggered at Step 2 of the Basic Flow when organizer chọn mục "Hình ảnh nhận diện".*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | Organizer | Chọn mục "Hình ảnh nhận diện" và tải lên file hình ảnh (banner hoặc thumbnail) |
| 2b | System | Xác thực định dạng file (chỉ chấp nhận JPEG, PNG, WebP) và kích thước (≤ 5MB) |
| 2c | System | Lưu trữ ảnh gốc và tạo phiên bản thumbnail tự động |
| 2d | System | Trả về image_url và thumbnail_url, hiển thị preview. Tiếp tục tại Step 3 |

##### AF-02.b: Organizer chỉ cập nhật một phần nội dung
>
> *Triggered at Step 3 of the Basic Flow when organizer không nhập tất cả trong một phiên.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Organizer | Chỉ nhập hoặc chỉnh sửa một số mục (không bắt buộc nhập tất cả trong một phiên) |
| 3b | System | Lưu các mục đã nhập, giữ nguyên các mục chưa thay đổi |
| 3c | System | Ghi nhận updated_at. Use case kết thúc |

#### 2.3 Exception Flows

##### EF-02.1: Thời gian tổ chức sự kiện nằm trong quá khứ
>
> *Triggered at Step 4 of the Basic Flow when event_date nằm trong quá khứ.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện event_date_start hoặc event_date_end nằm trong quá khứ |
| 4b | System | Hiển thị thông báo lỗi "Thời gian tổ chức phải trong tương lai" |
| 4c | Organizer | Chỉnh sửa lại thời gian tổ chức và thử lưu lại |

##### EF-02.2: Giá trị số không hợp lệ
>
> *Triggered at Step 6 of the Basic Flow when giá trị số ≤ 0 hoặc không phải số.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện quy mô hoặc ngân sách không phải số dương (≤ 0 hoặc không phải số) |
| 6b | System | Hiển thị thông báo lỗi "Quy mô/Ngân sách phải là số dương" |
| 6c | Organizer | Chỉnh sửa lại giá trị và thử lưu lại |

##### EF-02.3: File hình ảnh không hợp lệ
>
> *Triggered at Step 2b in AF-02.a when file không đúng định dạng hoặc vượt quá giới hạn.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2b-a | System | Phát hiện file không đúng định dạng (ví dụ: PDF, BMP) hoặc vượt quá 5MB |
| 2b-b | System | Hiển thị thông báo lỗi "Chỉ chấp nhận JPEG, PNG, WebP" hoặc "File vượt quá giới hạn 5MB" |
| 2b-c | Organizer | Chọn file khác và thử tải lên lại |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-02-01 | Cập nhật toàn bộ nội dung | Organizer nhập đầy đủ thông tin cơ bản, chi tiết, và hình thức tài trợ trong cùng phiên |
| SC-02-02 | Cập nhật từng phần | Organizer chỉ cập nhật một số mục nội dung; các mục còn lại giữ nguyên (AF-02.b) |
| SC-02-03 | Tải lên hình ảnh nhận diện | Organizer bổ sung hình ảnh banner/thumbnail hợp lệ cho hồ sơ (AF-02.a) |
| SC-02-04 | Dữ liệu không hợp lệ | Organizer nhập thời gian sự kiện trong quá khứ hoặc giá trị số không hợp lệ; nội dung không được lưu (EF-02.1, EF-02.2) |

---

### 5. Preconditions

#### 5.1 Organizer đã xác thực

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`

#### 5.2 Hồ sơ tài trợ tồn tại

- Hồ sơ tài trợ đã tồn tại và đang ở trạng thái DRAFT hoặc PUBLISHED

#### 5.3 Quyền sở hữu

- Organizer là tài khoản đại diện duy nhất của tổ chức BTC sở hữu hồ sơ

---

### 6. Postconditions

#### 6.1 Success

- Hồ sơ tài trợ đã được cập nhật với nội dung mới
- Thời gian cập nhật (updated_at) được ghi nhận
- Nếu hồ sơ đang PUBLISHED, thông tin mới được phản ánh trên trang tìm kiếm

#### 6.2 Failure

- Nội dung hồ sơ không thay đổi
- Organizer được thông báo lỗi xác thực cụ thể để sửa chữa

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0103: Các trường số (quy mô, ngân sách) PHẢI là số dương. Thời gian tổ chức PHẢI nằm trong tương lai
- BR-0104: File hình ảnh PHẢI có định dạng JPEG, PNG, hoặc WebP. Kích thước tối đa 5MB
- BR-0105: Hồ sơ PHẢI có ít nhất một hình thức tài trợ trước khi phát hành (kiểm tra tại UC-04)

---

### 10. Additional Information

**Assumptions:**

- Organizer không bắt buộc phải nhập tất cả nội dung trong một phiên làm việc — có thể lưu dần
- Hình ảnh nhận diện là tùy chọn (SHOULD, không phải MUST)

**Related Use Cases:**

- UC-01: Tạo hồ sơ tài trợ sự kiện (prerequisite — hồ sơ phải tồn tại)
- UC-03: Quản lý gói tài trợ (sequential — bổ sung gói tài trợ)
- UC-04: Phát hành hồ sơ tài trợ (sequential — sau khi nội dung đầy đủ)
