# Use-Case Specification: UC-29 — Gửi đánh giá đối tác

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người gửi đánh giá |
| Secondary | System | Kiểm duyệt nội dung, tính điểm tổng hợp, gửi thông báo |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) gửi đánh giá về đối tác sau khi hợp đồng tài trợ kết thúc. Đánh giá bao gồm điểm uy tín, điểm chất lượng hợp tác, và nhận xét văn bản. Hệ thống tự động tính lại điểm uy tín tổng hợp của đối tác.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Đánh giá đối tác" trên trang hợp đồng đã kết thúc, hoặc từ thông báo nhắc đánh giá.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Đánh giá đối tác" |
| 2 | System | Hiển thị form đánh giá: điểm uy tín (1-5 sao), điểm chất lượng hợp tác (1-5 sao), nhận xét (tùy chọn, tối đa 1000 ký tự) |
| 3 | Authenticated User | Chấm điểm uy tín (1-5) |
| 4 | Authenticated User | Chấm điểm chất lượng hợp tác (1-5) |
| 5 | Authenticated User | Nhập nhận xét văn bản (tùy chọn) |
| 6 | Authenticated User | Nhấn "Gửi đánh giá" |
| 7 | System | Xác thực: điểm 1-5, nhận xét ≤ 1000 ký tự |
| 8 | System | Lưu đánh giá, ghi nhận submitted_at |
| 9 | System | Kiểm duyệt tự động nội dung nhận xét (bộ lọc từ khóa) |
| 10 | System | Nếu nội dung hợp lệ: đặt moderation_status = APPROVED |
| 11 | System | Tính lại điểm uy tín tổng hợp cho đối tác (trung bình cộng tất cả đánh giá APPROVED) |
| 12 | System | Gửi thông báo cho đối tác "Bạn đã nhận được đánh giá mới" |
| 13 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-29.a: Nội dung nhận xét bị đánh dấu vi phạm
>
> *Triggered at Step 9 of the Basic Flow when bộ lọc tự động phát hiện nội dung không phù hợp.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 9a | System | Bộ lọc tự động phát hiện từ ngữ không phù hợp |
| 9b | System | Đặt moderation_status = FLAGGED — đánh giá không hiển thị công khai |
| 9c | System | Gửi yêu cầu review cho Admin |
| 9d | System | Đánh giá chờ Admin xem xét trong vòng 48 giờ |
| 9e | System | Điểm đánh giá KHÔNG được tính vào điểm tổng hợp cho đến khi APPROVED |

#### 2.3 Exception Flows

##### EF-29.1: Hợp đồng chưa kết thúc
>
> *Triggered at Step 1 of the Basic Flow when hợp đồng vẫn đang hiệu lực.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hợp đồng vẫn đang hiệu lực (validity_end > hôm nay) |
| 1b | System | Từ chối "Chỉ có thể đánh giá sau khi hợp đồng kết thúc" |

##### EF-29.2: Actor đã đánh giá trước đó
>
> *Triggered at Step 1 of the Basic Flow when actor đã gửi đánh giá cho hợp đồng này.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện actor đã gửi đánh giá cho đối tác trong hợp đồng này |
| 1b | System | Từ chối "Bạn đã đánh giá đối tác cho hợp đồng này rồi" |

##### EF-29.3: Điểm đánh giá không hợp lệ
>
> *Triggered at Step 7 of the Basic Flow when điểm ngoài khoảng 1-5.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | System | Phát hiện điểm ngoài khoảng 1-5 |
| 7b | System | Từ chối "Điểm đánh giá phải nằm trong khoảng 1-5" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-29-01 | Đánh giá thành công | Actor gửi đánh giá hợp lệ; điểm uy tín tổng hợp được cập nhật |
| SC-29-02 | Nội dung bị flagged | Bộ lọc phát hiện nội dung vi phạm; chờ Admin review (AF-29.a) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hợp đồng đã kết thúc

- Hợp đồng đã kết thúc (validity_end < ngày hiện tại HOẶC tất cả nghĩa vụ đều CONFIRMED)

#### 5.3 Chưa đánh giá

- Actor chưa gửi đánh giá cho đối tác trong hợp đồng này

#### 5.4 Actor là bên liên quan

- Actor là một trong hai bên liên quan

---

### 6. Postconditions

#### 6.1 Success

- Đánh giá được lưu thành công
- Nếu APPROVED: điểm uy tín tổng hợp được cập nhật, đánh giá hiển thị công khai
- Nếu FLAGGED: chờ Admin review
- Đối tác được thông báo

#### 6.2 Failure

- Đánh giá không được lưu
- Actor được thông báo lỗi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0701: Đánh giá chỉ gửi khi hợp đồng kết thúc (validity_end < hôm nay HOẶC tất cả nghĩa vụ CONFIRMED)
- BR-0702: Mỗi bên chỉ MỘT đánh giá cho mỗi hợp đồng
- BR-0703: Điểm 1-5 (số nguyên), nhận xét tùy chọn ≤ 1000 ký tự
- BR-0705: Điểm tổng hợp = trung bình cộng đánh giá APPROVED. Cập nhật real-time
- BR-0707: Đánh giá mới kiểm duyệt tự động. FLAGGED cần Admin review trong 48 giờ

---

### 10. Additional Information

**Assumptions:**

- Hệ thống gửi nhắc nhở đánh giá tự động: lần 1 vào ngày kết thúc HĐ, lần 2 sau 7 ngày (BR-0704)
- Đánh giá REMOVED không hiển thị và không tính vào điểm tổng hợp

**Related Use Cases:**

- UC-25: Theo dõi trạng thái nghĩa vụ (prerequisite — nghĩa vụ phải CONFIRMED)
- UC-28: Nộp báo cáo kết quả sự kiện (sequential — thường nộp trước khi đánh giá)
- UC-30: Xem điểm uy tín (sequential — điểm được cập nhật sau đánh giá)
- UC-31: Báo cáo đánh giá vi phạm (sequential — đối tác có thể báo cáo)
