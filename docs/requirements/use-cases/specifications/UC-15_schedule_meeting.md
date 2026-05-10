# Use-Case Specification: UC-15 — Đặt lịch họp thương thảo

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người đề xuất lịch họp |
| Secondary | System | Xác thực, gửi thông báo, nhắc nhở |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) đề xuất lịch họp/meeting với đối tác trong phạm vi thương vụ, bao gồm ngày giờ, thời lượng, chủ đề, và hình thức họp (online/trực tiếp). Hệ thống chỉ ghi nhận lịch hẹn và gửi nhắc nhở; hệ thống không tổ chức hoặc host cuộc gọi.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Đặt lịch họp" trong trang thương thảo của deal.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Đặt lịch họp" trong trang thương thảo |
| 2 | System | Hiển thị form đặt lịch: ngày giờ, thời lượng, chủ đề, ghi chú, hình thức họp (ONLINE/IN_PERSON), link/địa điểm |
| 3 | Authenticated User | Nhập ngày giờ họp, thời lượng dự kiến (phút), và chủ đề (bắt buộc) |
| 4 | Authenticated User | Chọn hình thức họp và nhập link/địa điểm (nếu có) |
| 5 | Authenticated User | Nhấn "Gửi đề xuất" |
| 6 | System | Xác thực: ngày giờ phải trong tương lai (≥ 1 giờ từ hiện tại) |
| 7 | System | Tạo meeting với trạng thái PROPOSED, ghi nhận proposed_at |
| 8 | System | Gửi thông báo in-app và email cho đối tác về đề xuất lịch họp |
| 9 | System | Use case kết thúc thành công — đề xuất đã gửi, chờ đối tác phản hồi |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-15.1: Ngày giờ không hợp lệ
>
> *Triggered at Step 6 of the Basic Flow when thời gian dưới 1 giờ từ hiện tại.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện ngày giờ họp trong quá khứ hoặc dưới 1 giờ từ hiện tại |
| 6b | System | Từ chối với thông báo "Thời gian họp phải ít nhất 1 giờ kể từ hiện tại" |
| 6c | Authenticated User | Chọn thời gian hợp lệ và thử gửi lại |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-15-01 | Đề xuất lịch họp thành công | Actor đề xuất lịch họp hợp lệ; đối tác được thông báo |
| SC-15-02 | Thời gian không hợp lệ | Thời gian dưới 1 giờ từ hiện tại; đề xuất không thành công (EF-15.1) |

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

- Meeting được tạo với trạng thái PROPOSED
- Đối tác được thông báo và có thể phản hồi (UC-16)
- Khi meeting CONFIRMED, hệ thống gửi nhắc nhở 30 phút trước giờ họp

#### 6.2 Failure

- Meeting không được tạo
- Actor được thông báo lỗi xác thực

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0404: Lịch họp chỉ đề xuất với ngày giờ trong tương lai (≥ 1 giờ). Hệ thống gửi nhắc nhở 30 phút trước giờ họp CONFIRMED

---

### 10. Additional Information

**Assumptions:**

- Hệ thống chỉ đóng vai trò ghi nhận lịch hẹn và gửi nhắc nhở; không cung cấp giao diện video call hay tổ chức cuộc họp trực tuyến
- Hệ thống không tích hợp lịch bên ngoài (Google Calendar, v.v.) ở phiên bản đầu
- Đối tác phản hồi đề xuất qua UC-16
- Kết quả họp được ghi nhận qua UC-17

**Related Use Cases:**

- UC-14: Trao đổi tin nhắn trong thương vụ (sequential — cùng deal context)
- UC-16: Phản hồi đề xuất lịch họp (sequential — đối tác phản hồi)
- UC-17: Ghi nhận kết quả cuộc họp (sequential — sau khi họp xong)
