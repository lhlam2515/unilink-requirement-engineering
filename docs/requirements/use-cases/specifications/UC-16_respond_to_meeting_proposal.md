# Use-Case Specification: UC-16 — Phản hồi đề xuất lịch họp

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Bên nhận đề xuất) | Người phản hồi đề xuất lịch họp |
| Secondary | System | Cập nhật trạng thái, gửi thông báo |

---

### 1. Brief Description

> Authenticated User (bên nhận đề xuất) phản hồi lịch họp đã đề xuất bằng cách chấp nhận, từ chối, hoặc đề xuất thời gian khác. Hệ thống thông báo kết quả cho bên đề xuất.

---

### 2. Flow of Events

**Trigger**
> Bên nhận nhấn "Chấp nhận", "Từ chối", hoặc "Đề xuất lại" trên lịch họp đã đề xuất.

#### 2.1 Basic Flow — Chấp nhận

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Xem chi tiết đề xuất lịch họp |
| 2 | System | Hiển thị thông tin: ngày giờ, thời lượng, chủ đề, ghi chú, hình thức |
| 3 | Authenticated User | Nhấn "Chấp nhận" |
| 4 | System | Chuyển meeting sang trạng thái CONFIRMED |
| 5 | System | Ghi nhận responded_at |
| 6 | System | Gửi thông báo cho bên đề xuất "Đề xuất lịch họp đã được chấp nhận" |
| 7 | System | Lên lịch nhắc nhở tự động 30 phút trước giờ họp cho cả hai bên |
| 8 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-16.a: Từ chối đề xuất
>
> *Triggered at Step 3 of the Basic Flow when bên nhận chọn từ chối.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Nhấn "Từ chối" |
| 3b | Authenticated User | Nhập ghi chú phản hồi (tùy chọn) |
| 3c | System | Chuyển meeting sang trạng thái DECLINED |
| 3d | System | Gửi thông báo cho bên đề xuất kèm ghi chú. Use case kết thúc |

##### AF-16.b: Đề xuất thời gian khác
>
> *Triggered at Step 3 of the Basic Flow when bên nhận muốn đề xuất lại.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Nhấn "Đề xuất lại" |
| 3b | System | Hiển thị form chọn ngày giờ mới |
| 3c | Authenticated User | Chọn ngày giờ mới và nhập ghi chú (tùy chọn) |
| 3d | System | Chuyển meeting gốc sang trạng thái RESCHEDULED |
| 3e | System | Tạo meeting mới với thời gian đề xuất mới, liên kết (rescheduled_from_id) đến meeting gốc |
| 3f | System | Gửi thông báo cho bên đề xuất về đề xuất thời gian mới |
| 3g | System | Use case kết thúc — bên đề xuất nhận đề xuất mới để phản hồi |

#### 2.3 Exception Flows

##### EF-16.1: Đề xuất thời gian mới không hợp lệ
>
> *Triggered at Step 3c trong AF-16.b when thời gian mới dưới 1 giờ từ hiện tại.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3c-a | System | Phát hiện ngày giờ mới trong quá khứ hoặc dưới 1 giờ từ hiện tại |
| 3c-b | System | Từ chối với thông báo "Thời gian họp phải ít nhất 1 giờ kể từ hiện tại" |
| 3c-c | Authenticated User | Chọn thời gian hợp lệ |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-16-01 | Chấp nhận đề xuất | Bên nhận chấp nhận; meeting chuyển sang CONFIRMED, nhắc nhở 30 phút trước |
| SC-16-02 | Từ chối đề xuất | Bên nhận từ chối có kèm ghi chú (AF-16.a) |
| SC-16-03 | Đề xuất thời gian khác | Bên nhận đề xuất lại thời gian mới; meeting gốc RESCHEDULED (AF-16.b) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Meeting đang PROPOSED

- Meeting đang ở trạng thái PROPOSED

#### 5.3 Actor là bên nhận đề xuất

- Actor là bên nhận đề xuất (không phải bên đề xuất)

---

### 6. Postconditions

#### 6.1 Success (Chấp nhận)

- Meeting chuyển sang CONFIRMED
- Nhắc nhở 30 phút trước giờ họp được lên lịch

#### 6.2 Success (Từ chối)

- Meeting chuyển sang DECLINED

#### 6.3 Success (Đề xuất lại)

- Meeting gốc chuyển sang RESCHEDULED
- Meeting mới được tạo với trạng thái PROPOSED

#### 6.4 Failure

- Meeting không thay đổi trạng thái

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Sau khi cuộc họp được CONFIRMED và diễn ra | UC-17: Ghi nhận kết quả cuộc họp | Actor muốn ghi nhận kết quả cuộc họp |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0404: Thời gian đề xuất lại phải trong tương lai (≥ 1 giờ). Nhắc nhở 30 phút trước giờ họp CONFIRMED

---

### 10. Additional Information

**Assumptions:**

- Quá trình đề xuất lại có thể lặp lại nhiều lần cho đến khi hai bên đồng ý
- Sau khi meeting CONFIRMED, cả hai bên có thể ghi nhận kết quả qua UC-17

**Related Use Cases:**

- UC-15: Đặt lịch họp thương thảo (prerequisite — meeting phải tồn tại)
- UC-17: Ghi nhận kết quả cuộc họp (`<<extend>>` — sau khi họp xong)
