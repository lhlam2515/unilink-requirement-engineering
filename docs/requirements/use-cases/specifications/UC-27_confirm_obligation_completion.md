# Use-Case Specification: UC-27 — Xác nhận hoàn thành nghĩa vụ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Bên đối tác — xác nhận chéo) | Organizer xác nhận nghĩa vụ Sponsor, và ngược lại |
| Secondary | System | Chuyển trạng thái, gửi thông báo |

---

### 1. Brief Description

> Authenticated User (bên đối tác — không phải bên thực hiện) xác nhận hoặc từ chối rằng nghĩa vụ đã được hoàn thành đúng cam kết, dựa trên bằng chứng đã nộp. Xác nhận chuyển nghĩa vụ sang CONFIRMED; từ chối chuyển sang DISPUTED cho phép bên thực hiện nộp lại.

---

### 2. Flow of Events

**Trigger**
> Bên đối tác nhấn "Xác nhận" hoặc "Từ chối" trên nghĩa vụ đã có bằng chứng SUBMITTED.

#### 2.1 Basic Flow — Xác nhận

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Mở chi tiết nghĩa vụ đang ở trạng thái SUBMITTED |
| 2 | System | Hiển thị bằng chứng: mô tả, file đính kèm (cho phép tải về và xem) |
| 3 | Authenticated User | Kiểm tra bằng chứng |
| 4 | Authenticated User | Nhấn "Xác nhận hoàn thành" |
| 5 | System | Chuyển nghĩa vụ sang trạng thái CONFIRMED, ghi nhận confirmed_at và confirmed_by |
| 6 | System | Gửi thông báo cho bên thực hiện "Nghĩa vụ [tên] đã được xác nhận hoàn thành" |
| 7 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-27.a: Từ chối xác nhận
>
> *Triggered at Step 4 of the Basic Flow when bên đối tác chọn từ chối.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Authenticated User | Nhấn "Từ chối" |
| 4b | System | Hiển thị form nhập lý do từ chối (bắt buộc) |
| 4c | Authenticated User | Nhập lý do từ chối (ví dụ: "Số tiền chuyển không khớp hợp đồng") |
| 4d | System | Chuyển nghĩa vụ sang trạng thái DISPUTED, ghi nhận disputed_at |
| 4e | System | Gửi thông báo cho bên thực hiện kèm lý do từ chối |
| 4f | System | Bên thực hiện có thể nộp bằng chứng mới (quay lại UC-26) |
| 4g | System | Use case kết thúc |

#### 2.3 Exception Flows

##### EF-27.1: Actor là bên thực hiện
>
> *Triggered at Step 1 of the Basic Flow when actor cố tự xác nhận.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện actor là bên chịu trách nhiệm (tự xác nhận) |
| 1b | System | Từ chối "Bạn không thể tự xác nhận nghĩa vụ của mình" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-27-01 | Xác nhận hoàn thành | Bên đối tác xác nhận; nghĩa vụ CONFIRMED |
| SC-27-02 | Từ chối xác nhận | Bên đối tác từ chối kèm lý do; nghĩa vụ DISPUTED (AF-27.a) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Nghĩa vụ đang SUBMITTED

- Nghĩa vụ đang ở trạng thái SUBMITTED

#### 5.3 Actor là bên đối tác

- Actor là bên ĐỐI TÁC (không phải bên thực hiện nghĩa vụ)

---

### 6. Postconditions

#### 6.1 Success (Xác nhận)

- Nghĩa vụ chuyển sang CONFIRMED
- Bên thực hiện được thông báo

#### 6.2 Success (Từ chối)

- Nghĩa vụ chuyển sang DISPUTED
- Bên thực hiện được thông báo kèm lý do, có thể nộp bằng chứng mới

#### 6.3 Failure

- Nghĩa vụ không thay đổi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0603: Chỉ BÊN ĐỐI TÁC mới có quyền xác nhận/từ chối. Organizer xác nhận nghĩa vụ Sponsor, Sponsor xác nhận nghĩa vụ Organizer. Lý do từ chối BẮT BUỘC

---

### 10. Additional Information

**Assumptions:**

- Cơ chế kiểm soát chéo: ngăn tự xác nhận
- Nghĩa vụ DISPUTED cho phép nộp lại bằng chứng mới (UC-26)

**Related Use Cases:**

- UC-25: Theo dõi trạng thái nghĩa vụ (`<<extend>>` base — UC-27 mở rộng UC-25)
- UC-26: Nộp bằng chứng hoàn thành nghĩa vụ (sequential — bằng chứng phải tồn tại)
