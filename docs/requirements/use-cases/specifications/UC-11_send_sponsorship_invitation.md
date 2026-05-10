# Use-Case Specification: UC-11 — Gửi lời mời tài trợ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người gửi lời mời |
| Secondary | System | Xác thực, tạo lời mời, gửi thông báo tự động |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) gửi lời mời tài trợ cho đối tác dựa trên một hồ sơ tài trợ sự kiện cụ thể. Lời mời bao gồm tin nhắn giới thiệu và hệ thống tự động gửi thông báo cho bên nhận qua in-app và email.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Gửi lời mời tài trợ" từ trang chi tiết hồ sơ đối tác hoặc hồ sơ sự kiện.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Gửi lời mời tài trợ" trên trang chi tiết hồ sơ |
| 2 | System | Hiển thị form gửi lời mời: tin nhắn giới thiệu (bắt buộc), gói tài trợ ưu tiên (tùy chọn) |
| 3 | Authenticated User | Nhập tin nhắn giới thiệu (tối thiểu 20 ký tự) |
| 4 | Authenticated User | Chọn gói tài trợ ưu tiên (nếu muốn) |
| 5 | Authenticated User | Nhấn "Gửi" |
| 6 | System | Xác thực: hồ sơ tài trợ đang PUBLISHED, chưa có lời mời PENDING cho cặp này, tin nhắn ≥ 20 ký tự |
| 7 | System | Tạo lời mời với trạng thái PENDING, ghi nhận sent_at |
| 8 | System | Gửi thông báo in-app cho bên nhận (trong vòng 30 giây) |
| 9 | System | Gửi email thông báo cho bên nhận (trong vòng 5 phút) — bao gồm tên bên gửi, tên sự kiện, tin nhắn giới thiệu, liên kết đến chi tiết lời mời |
| 10 | System | Hiển thị xác nhận "Lời mời tài trợ đã được gửi thành công" |
| 11 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-11.1: Đã tồn tại lời mời PENDING cho cặp này
>
> *Triggered at Step 6 of the Basic Flow when đã có lời mời PENDING.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện đã tồn tại lời mời PENDING giữa hai bên cho cùng hồ sơ |
| 6b | System | Từ chối với thông báo "Đã tồn tại lời mời đang chờ xử lý cho đối tác này" |
| 6c | Authenticated User | Có thể xem lời mời hiện có hoặc chờ bên nhận phản hồi |

##### EF-11.2: Tin nhắn giới thiệu quá ngắn
>
> *Triggered at Step 6 of the Basic Flow when tin nhắn dưới 20 ký tự.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện tin nhắn giới thiệu dưới 20 ký tự |
| 6b | System | Từ chối với thông báo "Tin nhắn giới thiệu phải có ít nhất 20 ký tự" |
| 6c | Authenticated User | Nhập tin nhắn dài hơn và thử gửi lại |

##### EF-11.3: Hồ sơ tài trợ không còn PUBLISHED
>
> *Triggered at Step 6 of the Basic Flow when hồ sơ đã bị hủy phát hành.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện hồ sơ tài trợ đã bị hủy phát hành (DRAFT) |
| 6b | System | Từ chối với thông báo "Hồ sơ tài trợ không còn khả dụng" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-11-01 | Gửi lời mời thành công | Actor gửi lời mời với tin nhắn hợp lệ; lời mời PENDING được tạo, bên nhận được thông báo |
| SC-11-02 | Lời mời trùng lặp | Đã tồn tại lời mời PENDING cho cùng cặp đối tác; lời mời không được tạo (EF-11.1) |
| SC-11-03 | Tin nhắn không hợp lệ | Tin nhắn giới thiệu dưới 20 ký tự; lời mời không được gửi (EF-11.2) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hồ sơ tài trợ đang PUBLISHED

- Hồ sơ tài trợ liên quan đang ở trạng thái PUBLISHED

#### 5.3 Chưa có lời mời PENDING

- Chưa tồn tại lời mời PENDING giữa hai bên cho cùng hồ sơ

---

### 6. Postconditions

#### 6.1 Success

- Lời mời tài trợ được tạo với trạng thái PENDING
- Bên nhận được thông báo qua in-app (≤ 30 giây) và email (≤ 5 phút)
- Lời mời xuất hiện trong danh sách lời mời đã gửi của actor

#### 6.2 Failure

- Lời mời không được tạo
- Actor được thông báo lỗi cụ thể

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0301: Lời mời chỉ gửi đến hồ sơ tài trợ ở trạng thái PUBLISHED
- BR-0302: Mỗi cặp (proposal_id + target_business_id) chỉ có MỘT lời mời PENDING tại một thời điểm
- BR-0303: Tin nhắn giới thiệu BẮT BUỘC, tối thiểu 20 ký tự
- BR-0304: Thông báo gửi qua in-app (≤ 30 giây) và email (≤ 5 phút)

---

### 10. Additional Information

**Assumptions:**

- Cả Organizer và Sponsor đều có thể gửi lời mời (hai chiều)
- Lời mời PENDING sẽ tự động hết hạn sau 30 ngày (1 tháng) kể từ ngày gửi (xử lý tự động bởi System, xem BR-0306)

**Related Use Cases:**

- UC-08: Xem chi tiết hồ sơ tài trợ sự kiện (`<<extend>>` base — UC-11 mở rộng UC-08)
- UC-09: Xem chi tiết hồ sơ doanh nghiệp (`<<extend>>` base — UC-11 mở rộng UC-09)
- UC-12: Phản hồi lời mời tài trợ (sequential — bên nhận phản hồi)
- UC-13: Theo dõi danh sách lời mời tài trợ (sequential — theo dõi trạng thái)
