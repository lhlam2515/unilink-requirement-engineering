# UC-25: Theo dõi trạng thái nghĩa vụ

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) xem dashboard nghĩa vụ tài trợ cho một hợp đồng đã ký, bao gồm danh sách nghĩa vụ của mình và đối tác, trạng thái từng nghĩa vụ, deadline, và tiến trình tổng thể. Danh sách nghĩa vụ được tạo tự động khi hợp đồng ký kết.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người theo dõi nghĩa vụ |
| Secondary | System | Tạo nghĩa vụ tự động, theo dõi deadline, gửi nhắc nhở |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hợp đồng đang ở trạng thái SIGNED
- Danh sách nghĩa vụ đã được tạo tự động (từ FR-0601)
- Actor là một trong hai bên liên quan

---

**Trigger**
> Actor truy cập trang "Nghĩa vụ tài trợ" của hợp đồng.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Truy cập trang "Nghĩa vụ tài trợ" của hợp đồng |
| 2 | System | Hiển thị dashboard nghĩa vụ với tiến trình tổng thể (tổng/hoàn thành/đang chờ/quá hạn) |
| 3 | System | Hiển thị danh sách nghĩa vụ của đối tác với trạng thái: PENDING, IN_PROGRESS, SUBMITTED, CONFIRMED, DISPUTED, OVERDUE |
| 4 | System | Hiển thị danh sách nghĩa vụ của bản thân với trạng thái tương ứng |
| 5 | System | Đánh dấu nghĩa vụ quá hạn bằng nhãn "QUÁ HẠN" (màu đỏ) |
| 6 | System | Hiển thị deadline cho từng nghĩa vụ |
| 7 | System | Use case kết thúc thành công — actor thấy toàn bộ trạng thái nghĩa vụ |

---

**Alternate Flows**

> AF-25.a: Actor xem chi tiết nghĩa vụ cụ thể (triggered at Step 3 hoặc 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Nhấn vào một nghĩa vụ cụ thể |
| 3b | System | Hiển thị chi tiết: mô tả, deadline, trạng thái, bằng chứng đã nộp (nếu có), lịch sử thay đổi |

---

**Exception Flows**

> EF-25.1: Chưa có nghĩa vụ nào (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hợp đồng chưa có nghĩa vụ (lỗi tạo tự động) |
| 2b | System | Hiển thị thông báo "Chưa có nghĩa vụ nào cho hợp đồng này" |

---

**Postconditions**

*Success:*
- Actor xem được toàn bộ nghĩa vụ và tiến trình

*Failure:*
- Không có dữ liệu hiển thị

---

**Business Rules**

- BR-0601: Danh sách nghĩa vụ PHẢI được tạo tự động khi hợp đồng SIGNED. Nghĩa vụ doanh nghiệp: chuyển khoản/bàn giao hiện vật. Nghĩa vụ BTC: quảng bá, truyền thông, tổ chức, báo cáo
- BR-0605: Nghĩa vụ PENDING hoặc IN_PROGRESS quá deadline tự động chuyển sang OVERDUE. Hệ thống gửi nhắc nhở T-3 ngày, T-0, T+1 ngày qua in-app và email

---

**Notes / Assumptions**

- Nghĩa vụ quá hạn được chuyển trạng thái tự động bởi scheduled job hàng ngày
- Nhắc nhở được gửi tự động: 3 ngày trước hạn, ngày hạn, 1 ngày sau hạn
- Actor có thể chuyển sang UC-26 để nộp bằng chứng hoặc UC-27 để xác nhận hoàn thành
- Liên kết: UC-22, UC-26, UC-27, UC-28
