# Use-Case Specification: UC-31 — Báo cáo đánh giá vi phạm

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người báo cáo vi phạm |
| Secondary | System | Tạo report, gửi cho Admin |
| Secondary | Admin | Xem xét và xử lý report |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) báo cáo một đánh giá mà họ cho là vi phạm (ngôn ngữ không phù hợp, spam, thông tin sai lệch). Hệ thống tạo report và gửi cho Admin xem xét.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Báo cáo đánh giá" trên một đánh giá cụ thể trong trang hồ sơ hoặc trang đánh giá.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Báo cáo đánh giá" trên đánh giá vi phạm |
| 2 | System | Hiển thị form báo cáo: lý do báo cáo (bắt buộc) |
| 3 | Authenticated User | Nhập lý do báo cáo (ví dụ: "Nội dung sai sự thật", "Ngôn ngữ không phù hợp") |
| 4 | Authenticated User | Nhấn "Gửi báo cáo" |
| 5 | System | Tạo report với trạng thái PENDING |
| 6 | System | Ghi nhận reported_by và reported_at |
| 7 | System | Gửi report cho Admin xem xét |
| 8 | System | Hiển thị xác nhận "Báo cáo đã được gửi. Chúng tôi sẽ xem xét trong vòng 48 giờ." |
| 9 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-31.a: Admin xử lý report — giữ đánh giá
>
> *Triggered bởi Admin when đánh giá không vi phạm.*

| Step | Actor / System | Action |
|------|----------------|--------|
| – | Admin | Xem xét đánh giá bị báo cáo |
| – | Admin | Nhận thấy đánh giá không vi phạm |
| – | Admin | Chuyển report sang RESOLVED_KEPT |
| – | System | Giữ nguyên đánh giá, không thay đổi |

##### AF-31.b: Admin xử lý report — xóa đánh giá
>
> *Triggered bởi Admin when đánh giá xác nhận vi phạm.*

| Step | Actor / System | Action |
|------|----------------|--------|
| – | Admin | Xem xét và xác nhận đánh giá vi phạm |
| – | Admin | Chuyển report sang RESOLVED_REMOVED |
| – | System | Đặt moderation_status của đánh giá sang REMOVED |
| – | System | Đánh giá không còn hiển thị công khai |
| – | System | Tính lại điểm uy tín tổng hợp (loại bỏ đánh giá đã REMOVED) |

#### 2.3 Exception Flows

##### EF-31.1: Đánh giá đã bị báo cáo trước đó bởi cùng actor
>
> *Triggered at Step 5 of the Basic Flow when actor đã báo cáo đánh giá này trước đó.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện actor đã báo cáo đánh giá này trước đó |
| 5b | System | Thông báo "Bạn đã báo cáo đánh giá này. Báo cáo đang được xem xét." |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-31-01 | Báo cáo thành công | Actor gửi báo cáo đánh giá; Admin sẽ xem xét trong 48 giờ |
| SC-31-02 | Admin giữ đánh giá | Admin xem xét và quyết định đánh giá không vi phạm (AF-31.a) |
| SC-31-03 | Admin xóa đánh giá | Admin xem xét và xác nhận vi phạm; đánh giá bị REMOVED (AF-31.b) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Đánh giá đang hiển thị

- Đánh giá đang hiển thị công khai (moderation_status = APPROVED)

---

### 6. Postconditions

#### 6.1 Success

- Report được tạo và gửi cho Admin
- Admin xem xét trong vòng 48 giờ
- Nếu vi phạm: đánh giá bị REMOVED, điểm uy tín được tính lại
- Nếu không vi phạm: đánh giá được giữ nguyên

#### 6.2 Failure

- Report không được tạo

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0707: Đánh giá FLAGGED cần Admin review trong 48 giờ. Đánh giá REMOVED không hiển thị và không tính vào điểm tổng hợp

---

### 10. Additional Information

**Assumptions:**

- Nhiều actor có thể báo cáo cùng một đánh giá — mỗi report được xem xét riêng
- Admin có thể xem danh sách report pending từ admin panel

**Related Use Cases:**

- UC-29: Gửi đánh giá đối tác (sequential — đánh giá phải tồn tại)
- UC-30: Xem điểm uy tín (`<<extend>>` base — UC-31 mở rộng UC-30)
