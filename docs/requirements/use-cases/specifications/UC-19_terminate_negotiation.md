# UC-19: Hủy bỏ thương thảo

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) hủy bỏ thương thảo đang diễn ra. Bên hủy phải cung cấp lý do. Hệ thống chuyển deal sang trạng thái TERMINATED và thông báo cho bên còn lại.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người hủy bỏ thương thảo |
| Secondary | System | Chuyển trạng thái, gửi thông báo |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái IN_PROGRESS
- Actor là một trong hai bên liên quan trong deal

---

**Trigger**
> Actor nhấn "Hủy bỏ thương thảo" trong trang thương thảo.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
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

---

**Alternate Flows**

> AF-19.a: Actor hủy thao tác (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Authenticated User | Nhấn "Không, tiếp tục thương thảo" |
| 5b | System | Giữ nguyên deal ở IN_PROGRESS. Use case kết thúc |

---

**Exception Flows**

> EF-19.1: Deal đã ở trạng thái AGREED (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện deal ở trạng thái AGREED |
| 1b | System | Từ chối "Không thể hủy thương thảo đã đồng thuận. Vui lòng liên hệ hỗ trợ." |

> EF-19.2: Lý do hủy quá ngắn (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện lý do dưới 10 ký tự |
| 4b | System | Hiển thị thông báo "Lý do hủy phải có ít nhất 10 ký tự" |
| 4c | Authenticated User | Viết lý do chi tiết hơn |

---

**Postconditions**

*Success:*
- Deal chuyển sang trạng thái TERMINATED
- Đối tác được thông báo kèm lý do hủy
- Tất cả tính năng trao đổi trong deal bị vô hiệu hóa (không thể gửi tin nhắn mới)

*Failure:*
- Deal vẫn ở trạng thái IN_PROGRESS
- Actor được thông báo lỗi

---

**Business Rules**

- BR-0406: Lý do hủy BẮT BUỘC (tối thiểu 10 ký tự). Deal chỉ hủy khi ở IN_PROGRESS. Deal đã AGREED không thể hủy qua giao diện thương thảo

---

**Notes / Assumptions**

- Hành động hủy không thể hoàn tác
- Lịch sử tin nhắn và ghi chú cuộc họp vẫn được lưu để tham khảo
- Liên kết: UC-14, UC-18
