# Use-Case Specification: UC-04 — Phát hành hồ sơ tài trợ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người phát hành hồ sơ |
| Secondary | System | Xác thực tính đầy đủ, chuyển trạng thái, lập chỉ mục tìm kiếm |

---

### 1. Brief Description

> Organizer phát hành hồ sơ tài trợ sự kiện đã đầy đủ nội dung. Hệ thống xác thực toàn bộ hồ sơ theo quy tắc nghiệp vụ, chuyển trạng thái từ DRAFT sang PUBLISHED, và lập chỉ mục để hồ sơ hiển thị trên trang tìm kiếm cho doanh nghiệp.

---

### 2. Flow of Events

**Trigger**
> Organizer nhấn "Phát hành hồ sơ" trên trang chỉnh sửa hoặc quản lý hồ sơ tài trợ.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Organizer | Nhấn "Phát hành hồ sơ" |
| 2 | System | Xác thực toàn bộ hồ sơ theo BR-0108: kiểm tra tên sự kiện, loại hình, thời gian, địa điểm, quy mô, ngân sách, đối tượng khán giả, hình thức tài trợ, gói tài trợ và quyền lợi |
| 3 | System | Xác thực tất cả gói tài trợ có ít nhất một quyền lợi |
| 4 | System | Chuyển trạng thái hồ sơ từ DRAFT sang PUBLISHED |
| 5 | System | Ghi nhận thời gian phát hành (published_at) |
| 6 | System | Lập chỉ mục hồ sơ cho hệ thống tìm kiếm (SF-02) |
| 7 | System | Hiển thị thông báo xác nhận "Hồ sơ đã được phát hành thành công" |
| 8 | System | Use case kết thúc thành công — hồ sơ xuất hiện trên trang tìm kiếm |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-04.1: Hồ sơ thiếu trường bắt buộc
>
> *Triggered at Step 2 of the Basic Flow when hồ sơ thiếu thông tin theo BR-0108.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ thiếu một hoặc nhiều trường bắt buộc |
| 2b | System | Từ chối phát hành và hiển thị danh sách trường còn thiếu (ví dụ: "Chưa nhập tên sự kiện", "Chưa chọn hình thức tài trợ") |
| 2c | Organizer | Quay lại chỉnh sửa hồ sơ để bổ sung thông tin (UC-02, UC-03) |

##### EF-04.2: Gói tài trợ chưa có quyền lợi
>
> *Triggered at Step 3 of the Basic Flow when gói tài trợ không có quyền lợi nào.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện một hoặc nhiều gói tài trợ chưa có quyền lợi nào |
| 3b | System | Cảnh báo "Gói tài trợ [tên gói] chưa có quyền lợi nào" |
| 3c | Organizer | Quay lại thêm quyền lợi cho gói (UC-03) hoặc xóa gói trống |

##### EF-04.3: Chưa có gói tài trợ nào
>
> *Triggered at Step 2 of the Basic Flow when hồ sơ chưa có gói tài trợ.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hồ sơ chưa có gói tài trợ nào |
| 2b | System | Từ chối phát hành với thông báo "Cần có ít nhất một gói tài trợ để phát hành" |
| 2c | Organizer | Quay lại tạo gói tài trợ (UC-03) |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-04-01 | Phát hành thành công | Hồ sơ đạt đầy đủ điều kiện BR-0108; chuyển sang PUBLISHED và khả dụng trên trang tìm kiếm |
| SC-04-02 | Hồ sơ chưa đủ điều kiện | Hồ sơ thiếu trường bắt buộc hoặc chưa có gói tài trợ; phát hành không thành công (EF-04.1, EF-04.3) |
| SC-04-03 | Gói tài trợ chưa có quyền lợi | Gói tài trợ tồn tại nhưng chưa có quyền lợi nào; organizer cần bổ sung hoặc xóa gói trống (EF-04.2) |

---

### 5. Preconditions

#### 5.1 Organizer đã xác thực

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`

#### 5.2 Hồ sơ ở trạng thái DRAFT

- Hồ sơ tài trợ đang ở trạng thái DRAFT

#### 5.3 Quyền sở hữu

- Organizer là tài khoản đại diện duy nhất của tổ chức BTC sở hữu hồ sơ

---

### 6. Postconditions

#### 6.1 Success

- Hồ sơ tài trợ chuyển sang trạng thái PUBLISHED
- Thời gian phát hành (published_at) được ghi nhận
- Hồ sơ xuất hiện trong kết quả tìm kiếm của doanh nghiệp (SF-02)

#### 6.2 Failure

- Hồ sơ vẫn ở trạng thái DRAFT
- Organizer được thông báo chi tiết các trường/điều kiện cần bổ sung

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0105: Hồ sơ PHẢI có ít nhất một hình thức tài trợ
- BR-0106: Hồ sơ PHẢI có ít nhất một gói tài trợ với giá trị tối thiểu > 0 và slot ≥ 1
- BR-0108: Hồ sơ chỉ được phát hành khi ĐẦY ĐỦ: tên sự kiện, loại hình, thời gian, địa điểm, quy mô, ngân sách, đối tượng khán giả, hình thức tài trợ, gói tài trợ có quyền lợi

---

### 10. Additional Information

**Assumptions:**

- Hồ sơ phát hành sẽ hiển thị cho tất cả doanh nghiệp trên trang tìm kiếm
- Organizer vẫn có thể chỉnh sửa hồ sơ sau khi phát hành (UC-02)
- Organizer có thể hủy phát hành thông qua UC-05

**Related Use Cases:**

- UC-02: Chỉnh sửa nội dung hồ sơ tài trợ (prerequisite — nội dung phải đầy đủ)
- UC-03: Quản lý gói tài trợ (prerequisite — gói tài trợ phải có quyền lợi)
- UC-05: Hủy phát hành hồ sơ tài trợ (sequential — sau khi đã PUBLISHED)
- UC-06: Tìm kiếm sự kiện để tài trợ (sequential — hồ sơ PUBLISHED khả dụng cho tìm kiếm)
