# Use-Case Specification: UC-21 — Xác nhận nội dung hợp đồng

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Mỗi bên xác nhận nội dung |
| Secondary | System | Kiểm tra song phương, chuyển trạng thái |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) xác nhận đồng ý với nội dung hợp đồng hiện tại. Khi CẢ HAI bên đều xác nhận, hệ thống chuyển hợp đồng sang trạng thái CONFIRMED và mở khóa tính năng ký chữ ký điện tử.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Xác nhận nội dung hợp đồng" trên trang soạn thảo hợp đồng.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Xem nội dung hợp đồng hiện tại |
| 2 | Authenticated User | Nhấn "Xác nhận nội dung hợp đồng" |
| 3 | System | Ghi nhận xác nhận cho bên hiện tại (organizer_content_confirmed hoặc sponsor_content_confirmed = true) |
| 4 | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã xác nhận nội dung hợp đồng" |
| 5 | System | Kiểm tra cả hai bên đã xác nhận chưa |
| 6 | System | Nếu cả hai đã xác nhận: chuyển hợp đồng sang trạng thái CONFIRMED |
| 7 | System | Mở khóa nút "Ký chữ ký điện tử" |
| 8 | System | Gửi thông báo cho cả hai bên "Nội dung hợp đồng đã được xác nhận, sẵn sàng ký kết" |
| 9 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-21.a: Chỉ một bên xác nhận
>
> *Triggered at Step 5 of the Basic Flow when chỉ một bên đã xác nhận.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện chỉ một bên đã xác nhận |
| 5b | System | Giữ hợp đồng ở DRAFTING, chờ bên còn lại |
| 5c | System | Use case kết thúc (chưa hoàn thành) |

##### AF-21.b: Đối tác chỉnh sửa sau khi actor đã xác nhận
>
> *Triggered bởi UC-20 when đối tác chỉnh sửa nội dung hợp đồng sau khi đã có xác nhận.*

| Step | Actor / System | Action |
|------|----------------|--------|
| – | System | Phát hiện bên đối tác chỉnh sửa nội dung hợp đồng |
| – | System | Reset TẤT CẢ xác nhận về false |
| – | System | Thông báo cho bên đã xác nhận "Nội dung đã thay đổi, cần xác nhận lại" |
| – | System | Quy trình xác nhận bắt đầu lại |

#### 2.3 Exception Flows

Không có exception flow đặc biệt cho use case này.

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-21-01 | Cả hai xác nhận | Cả hai bên xác nhận; hợp đồng chuyển sang CONFIRMED |
| SC-21-02 | Một bên xác nhận | Chỉ một bên; chờ đối tác (AF-21.a) |
| SC-21-03 | Xác nhận bị reset | Đối tác chỉnh sửa sau khi đã xác nhận; tất cả xác nhận được reset (AF-21.b) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hợp đồng đang DRAFTING

- Hợp đồng đang ở trạng thái DRAFTING

#### 5.3 Actor là bên liên quan

- Actor là một trong hai bên liên quan

---

### 6. Postconditions

#### 6.1 Success (cả hai xác nhận)

- Hợp đồng chuyển sang trạng thái CONFIRMED
- Nút "Ký chữ ký điện tử" được mở khóa (UC-22)

#### 6.2 Success (một bên xác nhận)

- Hợp đồng vẫn ở DRAFTING, chờ bên còn lại

#### 6.3 Nếu có chỉnh sửa sau

- Tất cả xác nhận bị reset, quy trình bắt đầu lại

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0504: Khi bất kỳ bên nào chỉnh sửa sau khi có xác nhận: TẤT CẢ xác nhận PHẢI được reset

---

### 10. Additional Information

**Assumptions:**

- Xác nhận nội dung là bước bắt buộc trước khi ký — đảm bảo hai bên đồng ý cùng phiên bản
- Nếu cần chỉnh sửa thêm: quay lại UC-20, xác nhận bị reset

**Related Use Cases:**

- UC-20: Chỉnh sửa điều khoản hợp đồng (`<<include>>` — UC-21 bao gồm UC-20)
- UC-22: Ký hợp đồng điện tử (sequential — sau khi xác nhận nội dung)
