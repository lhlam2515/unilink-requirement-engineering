# UC-31: Báo cáo đánh giá vi phạm

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) báo cáo một đánh giá mà họ cho là vi phạm (ngôn ngữ không phù hợp, spam, thông tin sai lệch). Hệ thống tạo report và gửi cho Admin xem xét.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người báo cáo vi phạm |
| Secondary | System | Tạo report, gửi cho Admin |
| Secondary | Admin | Xem xét và xử lý report |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Đánh giá đang hiển thị công khai (moderation_status = APPROVED)

---

**Trigger**
> Actor nhấn "Báo cáo đánh giá" trên một đánh giá cụ thể trong trang hồ sơ hoặc trang đánh giá.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Nhấn "Báo cáo đánh giá" trên đánh giá vi phạm |
| 2 | System | Hiển thị form báo cáo: lý do báo cáo (bắt buộc) |
| 3 | Authenticated User | Nhập lý do báo cáo (ví dụ: "Nội dung sai sự thật", "Ngôn ngữ không phù hợp") |
| 4 | Authenticated User | Nhấn "Gửi báo cáo" |
| 5 | System | Tạo report với trạng thái PENDING |
| 6 | System | Ghi nhận reported_by và reported_at |
| 7 | System | Gửi report cho Admin xem xét |
| 8 | System | Hiển thị xác nhận "Báo cáo đã được gửi. Chúng tôi sẽ xem xét trong vòng 48 giờ." |
| 9 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-31.a: Admin xử lý report — giữ đánh giá (triggered bởi Admin)

| Step | Actor / System | Action |
|------|----------------|--------|
| – | Admin | Xem xét đánh giá bị báo cáo |
| – | Admin | Nhận thấy đánh giá không vi phạm |
| – | Admin | Chuyển report sang RESOLVED_KEPT |
| – | System | Giữ nguyên đánh giá, không thay đổi |

> AF-31.b: Admin xử lý report — xóa đánh giá (triggered bởi Admin)

| Step | Actor / System | Action |
|------|----------------|--------|
| – | Admin | Xem xét và xác nhận đánh giá vi phạm |
| – | Admin | Chuyển report sang RESOLVED_REMOVED |
| – | System | Đặt moderation_status của đánh giá sang REMOVED |
| – | System | Đánh giá không còn hiển thị công khai |
| – | System | Tính lại điểm uy tín tổng hợp (loại bỏ đánh giá đã REMOVED) |

---

**Exception Flows**

> EF-31.1: Đánh giá đã bị báo cáo trước đó bởi cùng actor (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Phát hiện actor đã báo cáo đánh giá này trước đó |
| 5b | System | Thông báo "Bạn đã báo cáo đánh giá này. Báo cáo đang được xem xét." |

---

**Postconditions**

*Success:*
- Report được tạo và gửi cho Admin
- Admin xem xét trong vòng 48 giờ
- Nếu vi phạm: đánh giá bị REMOVED, điểm uy tín được tính lại
- Nếu không vi phạm: đánh giá được giữ nguyên

*Failure:*
- Report không được tạo

---

**Business Rules**

- BR-0707: Đánh giá FLAGGED cần Admin review trong 48 giờ. Đánh giá REMOVED không hiển thị và không tính vào điểm tổng hợp

---

**Notes / Assumptions**

- Nhiều actor có thể báo cáo cùng một đánh giá — mỗi report được xem xét riêng
- Admin có thể xem danh sách report pending từ admin panel
- Liên kết: UC-29, UC-30
