# Use-Case Specification: UC-22 — Ký hợp đồng điện tử

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Mỗi bên ký chữ ký điện tử |
| Secondary | System | Lưu chữ ký, kiểm tra song phương, tạo nghĩa vụ tự động |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) ký chữ ký điện tử lên hợp đồng đã được xác nhận nội dung (CONFIRMED). Hệ thống hỗ trợ chữ ký vẽ tay hoặc gõ tên. Khi CẢ HAI bên đã ký, hợp đồng chuyển sang trạng thái SIGNED và hệ thống tự động tạo danh sách nghĩa vụ tài trợ. [UPDATED — BP03] Hợp đồng PHẢI được ký trong 72 giờ kể từ khi hard-lock (2/2 thanh toán). Hệ thống hiển thị đếm ngược thời hạn ký.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Ký hợp đồng" trên trang hợp đồng đã xác nhận.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Ký hợp đồng" trên trang hợp đồng CONFIRMED. Hệ thống hiển thị đếm ngược thời hạn ký (còn lại từ 72 giờ) |
| 2 | System | Hiển thị giao diện ký: chọn hình thức ký (vẽ tay hoặc gõ tên) |
| 3 | Authenticated User | Vẽ chữ ký hoặc gõ tên đầy đủ |
| 4 | Authenticated User | Nhấn "Xác nhận ký" |
| 5 | System | Lưu chữ ký (signature_data), ghi nhận signed = true và signed_at |
| 6 | System | Kiểm tra cả hai bên đã ký chưa |
| 7 | System | Nếu cả hai đã ký: chuyển hợp đồng sang trạng thái SIGNED |
| 8 | System | Tự động tạo danh sách nghĩa vụ tài trợ từ điều khoản hợp đồng (FR-0601 / SF-06) |
| 9 | System | Gửi thông báo cho cả hai bên "Hợp đồng đã được ký kết thành công" |
| 10 | System | Use case kết thúc thành công — hợp đồng đã ký kết |

#### 2.2 Alternate Flows

##### AF-22.a: Chỉ một bên ký
>
> *Triggered at Step 6 of the Basic Flow when chỉ một bên đã ký.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện chỉ một bên đã ký |
| 6b | System | Giữ hợp đồng ở CONFIRMED, chờ bên còn lại |
| 6c | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã ký hợp đồng, chờ bạn ký" |
| 6d | System | Use case kết thúc (chưa hoàn thành) |

#### 2.3 Exception Flows

##### EF-22.1: Hợp đồng chưa CONFIRMED
>
> *Triggered at Step 1 of the Basic Flow when hợp đồng ở trạng thái DRAFTING.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hợp đồng ở trạng thái DRAFTING |
| 1b | System | Từ chối "Cần xác nhận nội dung bởi cả hai bên trước khi ký" |

##### EF-22.2: Actor đã ký trước đó
>
> *Triggered at Step 1 of the Basic Flow when actor đã ký hợp đồng này.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện actor đã ký hợp đồng này |
| 1b | System | Thông báo "Bạn đã ký hợp đồng này. Đang chờ đối tác ký." |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-22-01 | Cả hai ký | Cả hai bên ký trong 72 giờ; hợp đồng SIGNED, nghĩa vụ được tạo tự động |
| SC-22-02 | Một bên ký | Chỉ một bên; chờ đối tác trong thời hạn 72 giờ (AF-22.a) |
| SC-22-03 | Quá hạn 72 giờ | Chưa đủ 2 chữ ký trong 72 giờ; phát sinh vi phạm UC-49 |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hợp đồng đã CONFIRMED

- Hợp đồng đang ở trạng thái CONFIRMED

#### 5.3 Trong giai đoạn hard-lock

- Hợp đồng đang trong giai đoạn hard-lock (cancel_action_enabled = false)
- signing_deadline_at chưa qua (còn trong 72 giờ)

#### 5.4 Actor chưa ký

- Actor là một trong hai bên liên quan
- Actor chưa ký hợp đồng này

---

### 6. Postconditions

#### 6.1 Success (cả hai ký)

- Hợp đồng chuyển sang trạng thái SIGNED với chữ ký của hai bên
- Ngày ký (signing_date) được ghi nhận
- Danh sách nghĩa vụ được tạo tự động (UC-25)
- Đếm ngược 72 giờ dừng lại
- Cả hai bên được thông báo

#### 6.2 Success (một bên ký)

- Hợp đồng vẫn ở CONFIRMED, chờ bên còn lại
- Đếm ngược 72 giờ tiếp tục

#### 6.3 Failure (quá hạn 72 giờ)

- Phát sinh sự kiện vi phạm (SF-14 / UC-49)
- Hợp đồng không thay đổi

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Sau khi hợp đồng chuyển sang SIGNED | UC-23: Xuất hợp đồng dạng PDF | Actor muốn tải PDF hợp đồng đã ký |
| 2 | Sau khi hợp đồng chuyển sang SIGNED | UC-24: Yêu cầu hóa đơn VAT phí dịch vụ | Sponsor cần hóa đơn VAT cho phí dịch vụ kết nối đã thanh toán |
| 3 | Quá hạn ký 72 giờ mà chưa đủ 2 chữ ký | UC-49: Xử lý vi phạm ký kết hợp đồng | Actor muốn report đối tác trì hoãn ký |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0505: Chữ ký điện tử chỉ thực hiện khi hợp đồng CONFIRMED. Mỗi bên ký MỘT LẦN, sau khi ký không thể rút lại
- BR-0506: Hợp đồng chuyển sang SIGNED khi VÀ CHỈ KHI cả hai bên đều đã ký
- BR-0510: Hợp đồng PHẢI hoàn tất 2 chữ ký trong 72 giờ kể từ hard-lock

---

### 10. Additional Information

**Assumptions:**

- Phiên bản đầu sử dụng chữ ký điện tử đơn giản (vẽ/gõ), không phải chữ ký số PKI
- Sau khi SIGNED: có thể xuất PDF (UC-23)
- Nghĩa vụ tài trợ được tạo tự động — theo dõi qua UC-25 đến UC-28
- Nếu hết 72 giờ mà chưa đủ 2 chữ ký: UC-49 xử lý vi phạm

**Related Use Cases:**

- UC-21: Xác nhận nội dung hợp đồng (prerequisite — hợp đồng phải CONFIRMED)
- UC-23: Xuất hợp đồng PDF (sequential — sau khi SIGNED)
- UC-24: Yêu cầu hóa đơn VAT (`<<extend>>` — sau khi SIGNED)
- UC-25: Theo dõi trạng thái nghĩa vụ (sequential — nghĩa vụ được tạo tự động)
- UC-49: Xử lý vi phạm ký kết hợp đồng (`<<extend>>` — quá hạn 72h)
- UC-50: Thanh toán phí dịch vụ (prerequisite — hard-lock sau 2/2 thanh toán)
