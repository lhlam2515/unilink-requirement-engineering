# Use-Case Specification: UC-51 — Xem trước chi phí dịch vụ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Xem ước tính phí |
| Secondary | System | Tính phí dựa trên cấu hình hiện tại |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) xem ước tính phí dịch vụ kết nối dựa trên giá trị tài trợ đang thương thảo, trước khi đồng thuận lock-in. Giúp tránh bất ngờ về chi phí.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Xem ước tính phí dịch vụ" trong trang thương thảo hoặc trang tạo thỏa thuận nháp.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Xem ước tính phí dịch vụ" |
| 2 | System | Hiển thị form: Hình thức tài trợ (CASH/IN_KIND/COMBINED), Giá trị tiền mặt (nếu CASH/COMBINED) |
| 3 | Authenticated User | Nhập hình thức và giá trị tài trợ ước tính |
| 4 | System | Tính phí ước tính cho cả hai bên dựa trên FeeConfiguration hiện tại |
| 5 | System | Hiển thị: Phí CLB = [số tiền], Phí DN = [số tiền], Tổng phí nền tảng = [tổng] |
| 6 | System | Hiển thị ghi chú: "Đây là ước tính, phí thực tế sẽ được xác nhận khi lock-in" |
| 7 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-51.a: Tài trợ hoàn toàn hiện vật
>
> *Triggered at Step 3 of the Basic Flow when hình thức là IN_KIND.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Chọn IN_KIND |
| 3b | System | Hiển thị phí cố định: CLB = 50.000 VNĐ, DN = 500.000 VNĐ |
| 3c | System | Không yêu cầu nhập giá trị tiền mặt |

#### 2.3 Exception Flows

##### EF-51.1: Chưa nhập giá trị tiền mặt cho CASH/COMBINED
>
> *Triggered at Step 3 of the Basic Flow when thiếu giá trị tiền mặt.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Hiển thị "Vui lòng nhập giá trị tài trợ tiền mặt để ước tính phí" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-51-01 | Ước tính phí CASH | Actor nhập giá trị tiền mặt; xem phí ước tính |
| SC-51-02 | Ước tính phí IN_KIND | Actor chọn hiện vật; xem phí cố định (AF-51.a) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Deal đang IN_PROGRESS

- Deal đang ở trạng thái IN_PROGRESS

#### 5.3 Actor là bên liên quan

- Actor là một trong hai bên liên quan trong deal

---

### 6. Postconditions

#### 6.1 Success

- Actor nhìn thấy phí ước tính cho cả hai bên
- Không có thay đổi dữ liệu — chỉ hiển thị thông tin

#### 6.2 Failure

- Không hiển thị ước tính do thiếu dữ liệu đầu vào

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1201: Cơ cấu phí admin-configurable
- BR-1208: Cấu hình phí chỉ admin thay đổi, áp dụng cho deal mới

---

### 10. Additional Information

**Assumptions:**

- Đây là tính năng read-only, không ảnh hưởng đến deal hay thanh toán
- Phí ước tính có thể thay đổi nếu admin cập nhật cấu hình trước khi lock-in

**Related Use Cases:**

- UC-56: Tạo thỏa thuận nháp (sequential — ước tính phí trong quá trình thương thảo)
- UC-50: Thanh toán phí dịch vụ (sequential — phí thực tế sau lock-in)
