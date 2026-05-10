# Use-Case Specification: UC-49 — Xử lý vi phạm ký kết hợp đồng

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (bên đã hoàn tất nghĩa vụ) | Người gửi report |
| Secondary | System | Kiểm tra deadline, đóng băng, gửi cảnh báo, thực thi chế tài |
| Secondary | Reported Party | Bên bị báo cáo |
| Secondary | Admin | Xử lý chế tài hoặc ngoại lệ nếu cần |

---

### 1. Brief Description

> Authenticated User (bên đã hoàn tất nghĩa vụ) báo cáo đối tác khi hợp đồng đã quá hạn ký 72 giờ nhưng chưa đủ 2 chữ ký. Hệ thống tạm thời đóng băng tài khoản bên bị tố cáo, gửi cảnh báo cuối cùng và cho thêm 24 giờ ân hạn để hoàn tất ký kết. Nếu hết ân hạn mà vẫn chưa đủ 2 chữ ký, hệ thống đóng thương vụ và kích hoạt chế tài.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Tố cáo đối tác" trên màn hình hợp đồng.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Tố cáo đối tác" |
| 2 | System | Kiểm tra contract đã quá hạn 72 giờ và chưa đủ 2 chữ ký |
| 3 | System | Hiển thị form report với lý do và bằng chứng tùy chọn |
| 4 | Authenticated User | Nhập lý do báo cáo và gửi report |
| 5 | System | Tạo contract breach case và chuyển trạng thái sang REPORTED |
| 6 | System | Tạm thời đóng băng tài khoản của bên bị tố cáo |
| 7 | System | Gửi cảnh báo cuối cùng, cho thêm 24 giờ ân hạn |
| 8 | System | Ghi nhận final_warning_expires_at |
| 9 | System | Use case kết thúc — chờ hết ân hạn hoặc bên vi phạm ký xong |

#### 2.2 Alternate Flows

##### AF-49.a: Bên vi phạm hoàn tất chữ ký trong 24 giờ ân hạn
>
> *Triggered at Step 7 of the Basic Flow when bên vi phạm ký xong trong ân hạn.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | Reported Party | Hoàn tất chữ ký còn thiếu |
| 7b | System | Chuyển contract sang SIGNED và đóng breach case |
| 7c | System | Gỡ đóng băng nếu chính sách cho phép |

#### 2.3 Exception Flows

##### EF-49.1: Chưa quá hạn 72 giờ
>
> *Triggered at Step 2 of the Basic Flow when chưa đến thời điểm tố cáo.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Từ chối report với thông báo "Chưa đến thời điểm tố cáo" |

##### EF-49.2: Report thiếu lý do
>
> *Triggered at Step 4 of the Basic Flow when lý do trống.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Từ chối gửi report |
| 4b | System | Hiển thị lỗi "Lý do tố cáo là bắt buộc" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-49-01 | Report hợp lệ | Bên đã hoàn tất nghĩa vụ report đối tác; đóng băng tạm + 24 giờ ân hạn |
| SC-49-02 | Hoàn tất trong ân hạn | Bên vi phạm ký xong trong 24 giờ; contract SIGNED, breach case đóng (AF-49.a) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Contract đã vào hard-lock

- Contract đã vào trạng thái hard-lock sau 2/2 payment

#### 5.3 Đã quá hạn ký 72 giờ

- Đã qua signing_deadline_at + 72 giờ nhưng contract chưa SIGNED

#### 5.4 Actor đã hoàn tất nghĩa vụ

- Actor là bên đã hoàn tất nghĩa vụ trong deal đó

---

### 6. Postconditions

#### 6.1 Success (report hợp lệ)

- Contract breach case được tạo
- Bên bị tố cáo bị đóng băng tạm thời
- Final warning 24h được kích hoạt

#### 6.2 Success (hết ân hạn mà không ký)

- Thương vụ bị đóng
- Chế tài tương ứng được thực thi

#### 6.3 Failure

- Không tạo report
- Trạng thái contract/deal không đổi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1406: Chỉ bên đã hoàn tất nghĩa vụ mới được mở quyền tố cáo khi quá hạn ký 72 giờ
- BR-1407: Sau report hợp lệ phải cấp thêm 24 giờ ân hạn
- BR-1408: Hết ân hạn mà chưa đủ 2 chữ ký thì phải đóng thương vụ và kích hoạt chế tài

---

### 10. Additional Information

**Assumptions:**

- Use case này là nhánh hậu hard-lock, thay thế hoàn toàn cơ chế hủy đồng thuận trước đây

**Related Use Cases:**

- UC-22: Ký hợp đồng điện tử (`<<extend>>` base — UC-49 mở rộng UC-22)
- UC-24: Yêu cầu hóa đơn VAT (sequential — nếu contract SIGNED sau ân hạn)
- UC-25: Theo dõi trạng thái nghĩa vụ (sequential — sau khi contract SIGNED)
