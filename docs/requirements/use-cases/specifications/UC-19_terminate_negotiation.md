# Use-Case Specification: UC-19 — Hủy bỏ thương thảo

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người hủy bỏ thương thảo |
| Secondary | System | Chuyển trạng thái, gửi thông báo |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) hủy bỏ thương thảo đang diễn ra. Bên hủy phải cung cấp lý do. Hệ thống chuyển deal sang trạng thái TERMINATED và thông báo cho bên còn lại.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Hủy bỏ thương thảo" trong trang thương thảo.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Hủy bỏ thương thảo" |
| 2 | System | Hiển thị form nhập lý do hủy (bắt buộc, tối thiểu 10 ký tự) |
| 3 | Authenticated User | Nhập lý do hủy bỏ |
| 4 | System | Xác thực lý do ≥ 10 ký tự |
| 5 | System | Hiển thị xác nhận "Bạn có chắc muốn hủy bỏ thương thảo? Hành động này không thể hoàn tác." |
| 6 | Authenticated User | Xác nhận hủy bỏ |
| 7 | System | Chuyển deal sang trạng thái TERMINATED, ghi nhận terminated_by và terminated_at |
| 8 | System | Gửi thông báo cho đối tác bao gồm lý do hủy |
| 9 | System | Vô hiệu hóa tính năng nhắn tin và các thao tác trong deal |
| 10 | System | Use case kết thúc thành công — thương thảo đã bị hủy |

#### 2.2 Alternate Flows

##### AF-19.a: Actor hủy thao tác
>
> *Triggered at Step 5 of the Basic Flow when actor chọn không hủy.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Authenticated User | Nhấn "Không, tiếp tục thương thảo" |
| 5b | System | Giữ nguyên deal ở IN_PROGRESS. Use case kết thúc |

#### 2.3 Exception Flows

##### EF-19.1: Deal đã ở trạng thái AGREED
>
> *Triggered at Step 1 of the Basic Flow when deal đã ở AGREED.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện deal ở trạng thái AGREED |
| 1b | System | Từ chối "Không thể hủy thương thảo đã đồng thuận. Vui lòng liên hệ hỗ trợ." |

##### EF-19.2: Lý do hủy quá ngắn
>
> *Triggered at Step 4 of the Basic Flow when lý do dưới 10 ký tự.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện lý do dưới 10 ký tự |
| 4b | System | Hiển thị thông báo "Lý do hủy phải có ít nhất 10 ký tự" |
| 4c | Authenticated User | Viết lý do chi tiết hơn |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-19-01 | Hủy thành công | Actor nhập lý do ≥ 10 ký tự và xác nhận; deal TERMINATED |
| SC-19-02 | Actor hủy thao tác | Actor không xác nhận; deal giữ nguyên IN_PROGRESS (AF-19.a) |
| SC-19-03 | Deal đã đồng thuận | Deal ở trạng thái AGREED; hủy không được phép (EF-19.1) |

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

- Deal chuyển sang trạng thái TERMINATED
- Đối tác được thông báo kèm lý do hủy
- Tất cả tính năng trao đổi trong deal bị vô hiệu hóa (không thể gửi tin nhắn mới)

#### 6.2 Failure

- Deal vẫn ở trạng thái IN_PROGRESS
- Actor được thông báo lỗi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0406: Lý do hủy BẮT BUỘC (tối thiểu 10 ký tự). Deal chỉ hủy khi ở IN_PROGRESS. Deal đã AGREED không thể hủy qua giao diện thương thảo

---

### 10. Additional Information

**Assumptions:**

- Hành động hủy không thể hoàn tác
- Lịch sử tin nhắn và ghi chú cuộc họp vẫn được lưu để tham khảo

**Related Use Cases:**

- UC-14: Trao đổi tin nhắn trong thương vụ (`<<extend>>` base — UC-19 mở rộng UC-14)
- UC-18: Xác nhận đồng thuận ký kết (sequential — đối lập với UC-19)
