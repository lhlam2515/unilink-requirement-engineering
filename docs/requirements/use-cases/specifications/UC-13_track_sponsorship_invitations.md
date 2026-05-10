# Use-Case Specification: UC-13 — Theo dõi danh sách lời mời tài trợ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người theo dõi lời mời |
| Secondary | System | Truy xuất và hiển thị danh sách lời mời |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) xem và quản lý danh sách tất cả lời mời tài trợ đã gửi và đã nhận, lọc theo trạng thái và chiều lời mời.

---

### 2. Flow of Events

**Trigger**
> Actor truy cập trang "Lời mời tài trợ" trong dashboard cá nhân.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Truy cập trang "Lời mời tài trợ" trong dashboard |
| 2 | System | Hiển thị danh sách lời mời với bộ lọc: trạng thái (ALL/PENDING/ACCEPTED/DECLINED/EXPIRED) và chiều (SENT/RECEIVED/ALL) |
| 3 | Authenticated User | Áp dụng bộ lọc mong muốn |
| 4 | System | Truy xuất và hiển thị danh sách lời mời phân trang |
| 5 | System | Mỗi lời mời hiển thị: tên đối tác, tên sự kiện, trạng thái, ngày gửi, ngày phản hồi (nếu có) |
| 6 | System | Use case kết thúc thành công — actor thấy danh sách lời mời |

#### 2.2 Alternate Flows

##### AF-13.a: Actor xem chi tiết lời mời
>
> *Triggered at Step 5 of the Basic Flow when actor chọn một lời mời cụ thể.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Authenticated User | Nhấn vào một lời mời cụ thể trong danh sách |
| 5b | System | Hiển thị trang chi tiết lời mời với đầy đủ thông tin |
| 5c | System | Nếu lời mời PENDING và actor là bên nhận: hiển thị nút "Chấp nhận" / "Từ chối" (chuyển sang UC-12) |

#### 2.3 Exception Flows

##### EF-13.1: Không có lời mời nào
>
> *Triggered at Step 4 of the Basic Flow when không có lời mời nào khớp bộ lọc.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Không tìm thấy lời mời nào khớp bộ lọc |
| 4b | System | Hiển thị thông báo "Chưa có lời mời tài trợ nào" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-13-01 | Xem danh sách lời mời | Actor xem danh sách lời mời đã gửi/nhận với bộ lọc trạng thái |
| SC-13-02 | Danh sách trống | Không có lời mời nào khớp bộ lọc; thông báo danh sách trống (EF-13.1) |
| SC-13-03 | Xem chi tiết và phản hồi | Actor chọn lời mời PENDING đã nhận; chuyển sang phản hồi qua UC-12 (AF-13.a) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

---

### 6. Postconditions

#### 6.1 Success

- Actor xem được danh sách lời mời đã gửi/nhận với trạng thái hiện tại

#### 6.2 Failure

- Danh sách trống — actor được thông báo

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Khi actor xem chi tiết lời mời PENDING đã nhận (AF-13.a) | UC-12: Phản hồi lời mời tài trợ | Actor là bên nhận và muốn chấp nhận hoặc từ chối lời mời |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- Không có business rule riêng cho use case này

---

### 10. Additional Information

**Assumptions:**

- Danh sách lời mời hiển thị cả lời mời đã gửi và đã nhận
- Lời mời PENDING tự động hết hạn sau 30 ngày (1 tháng) kể từ ngày gửi (BR-0306)
- Actor có thể chuyển sang UC-12 để phản hồi lời mời PENDING

**Related Use Cases:**

- UC-11: Gửi lời mời tài trợ (prerequisite — lời mời phải tồn tại)
- UC-12: Phản hồi lời mời tài trợ (`<<extend>>` — phản hồi từ trang chi tiết lời mời)
