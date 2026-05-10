# Use-Case Specification: UC-56 — Tạo thỏa thuận nháp

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Tạo thỏa thuận nháp |
| Secondary | Authenticated User (đối tác) | Xác nhận hoặc từ chối thỏa thuận |
| Secondary | System | Thông báo, lưu trữ, kiểm tra trạng thái |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) tạo Thỏa thuận nháp (Draft Agreement) tóm tắt các điều khoản đã thống nhất trong quá trình thương thảo. Đối tác xem và xác nhận hoặc từ chối thỏa thuận. Thỏa thuận nháp đã xác nhận là cơ sở để tính phí dịch vụ và soạn hợp đồng.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Tạo thỏa thuận nháp" trong trang thương thảo.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Tạo thỏa thuận nháp" trong trang thương thảo |
| 2 | System | Hiển thị form tạo thỏa thuận: Hình thức tài trợ, Giá trị tiền mặt (nếu có), Mô tả hiện vật (nếu có), Điều khoản chính |
| 3 | Authenticated User | Điền đầy đủ thông tin và nhấn "Gửi thỏa thuận" |
| 4 | System | Xác thực dữ liệu đầu vào |
| 5 | System | Tạo DraftAgreement với status = PENDING_CONFIRMATION |
| 6 | System | Gửi thông báo cho đối tác: "[Bên tạo] đã tạo thỏa thuận nháp. Vui lòng xem xét." |
| 7 | Đối tác | Xem nội dung thỏa thuận nháp |
| 8 | Đối tác | Nhấn "Xác nhận thỏa thuận" |
| 9 | System | Chuyển DraftAgreement sang CONFIRMED, ghi nhận confirmed_by và confirmed_at |
| 10 | System | Mở khóa nút "Xác nhận đồng thuận" (UC-18) |
| 11 | System | Gửi thông báo cho bên tạo: "Đối tác đã xác nhận thỏa thuận nháp" |
| 12 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-56.a: Đối tác từ chối thỏa thuận
>
> *Triggered at Step 8 of the Basic Flow when đối tác chọn từ chối.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | Đối tác | Nhấn "Từ chối" và nhập ghi chú lý do |
| 8b | System | Chuyển DraftAgreement sang REJECTED, lưu rejection_note |
| 8c | System | Gửi thông báo cho bên tạo: "Đối tác đã từ chối thỏa thuận. Lý do: [ghi chú]" |
| 8d | System | Bên tạo có thể tạo thỏa thuận nháp mới (quay lại Step 1) |

##### AF-56.b: Tài trợ kết hợp COMBINED
>
> *Triggered at Step 3 of the Basic Flow when actor chọn hình thức COMBINED.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Chọn hình thức COMBINED |
| 3b | System | Yêu cầu nhập CẢ giá trị tiền mặt VÀ mô tả hiện vật |
| 3c | Authenticated User | Điền đầy đủ cả hai phần |
| 3d | System | Tiếp tục Step 4 bình thường |

#### 2.3 Exception Flows

##### EF-56.1: Đã có thỏa thuận nháp đang chờ
>
> *Triggered at Step 1 of the Basic Flow when đã có DraftAgreement PENDING_CONFIRMATION.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện đã có DraftAgreement PENDING_CONFIRMATION |
| 1b | System | Từ chối tạo mới: "Đã có thỏa thuận nháp đang chờ đối tác xác nhận" |

##### EF-56.2: Thiếu giá trị tiền mặt cho hình thức CASH
>
> *Triggered at Step 4 of the Basic Flow when sponsorship_type = CASH nhưng giá trị trống.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện sponsorship_type = CASH nhưng sponsorship_value trống |
| 4b | System | Hiển thị lỗi "Vui lòng nhập giá trị tài trợ tiền mặt" |

##### EF-56.3: Thiếu mô tả hiện vật cho hình thức IN_KIND
>
> *Triggered at Step 4 of the Basic Flow when sponsorship_type = IN_KIND nhưng mô tả trống.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện sponsorship_type = IN_KIND nhưng in_kind_description trống |
| 4b | System | Hiển thị lỗi "Vui lòng nhập mô tả hiện vật" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-56-01 | Tạo và xác nhận thành công | Actor tạo thỏa thuận; đối tác xác nhận → CONFIRMED |
| SC-56-02 | Đối tác từ chối | Đối tác từ chối thỏa thuận kèm lý do; bên tạo có thể tạo lại (AF-56.a) |
| SC-56-03 | Thiếu dữ liệu bắt buộc | Thiếu giá trị tiền mặt hoặc mô tả hiện vật; thỏa thuận không được tạo (EF-56.2, EF-56.3) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Deal đang IN_PROGRESS

- Deal đang ở trạng thái IN_PROGRESS

#### 5.3 Actor là bên liên quan

- Actor là một trong hai bên liên quan trong deal

#### 5.4 Chưa có thỏa thuận active

- Không có thỏa thuận nháp PENDING_CONFIRMATION hoặc CONFIRMED nào tồn tại cho deal này

---

### 6. Postconditions

#### 6.1 Success (xác nhận)

- DraftAgreement ở trạng thái CONFIRMED
- Nút "Xác nhận đồng thuận" (UC-18) được mở khóa
- Thông tin sponsorship_type và sponsorship_value sẵn sàng cho tính phí (UC-50)

#### 6.2 Success (từ chối)

- DraftAgreement ở trạng thái REJECTED
- Bên tạo có thể tạo lại thỏa thuận mới

#### 6.3 Failure

- Không tạo được DraftAgreement do thiếu dữ liệu

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0407: Thỏa thuận nháp BẮT BUỘC trước khi đồng thuận. Mỗi deal chỉ có MỘT thỏa thuận CONFIRMED tại một thời điểm
- BR-1201: Thỏa thuận nháp xác định sponsorship_type và sponsorship_value — cơ sở tính phí dịch vụ

---

### 10. Additional Information

**Assumptions:**

- Thỏa thuận nháp có thể bị từ chối nhiều lần — bên tạo sửa và gửi lại
- Bên nào cũng có thể tạo thỏa thuận nháp (không giới hạn Organizer hay Sponsor)

**Related Use Cases:**

- UC-18: Xác nhận đồng thuận ký kết (sequential — yêu cầu thỏa thuận CONFIRMED)
- UC-51: Xem trước phí dịch vụ (sequential — sau khi thỏa thuận CONFIRMED)
- UC-50: Thanh toán phí dịch vụ (sequential — sau khi đồng thuận)
