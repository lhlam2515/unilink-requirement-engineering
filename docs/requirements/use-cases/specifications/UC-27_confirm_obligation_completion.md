# UC-27: Xác nhận hoàn thành nghĩa vụ

**Brief Description**
> Authenticated User (bên đối tác — không phải bên thực hiện) xác nhận hoặc từ chối rằng nghĩa vụ đã được hoàn thành đúng cam kết, dựa trên bằng chứng đã nộp. Xác nhận chuyển nghĩa vụ sang CONFIRMED; từ chối chuyển sang DISPUTED cho phép bên thực hiện nộp lại.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Bên đối tác — xác nhận chéo) | Organizer xác nhận nghĩa vụ Sponsor, và ngược lại |
| Secondary | System | Chuyển trạng thái, gửi thông báo |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Nghĩa vụ đang ở trạng thái SUBMITTED
- Actor là bên ĐỐI TÁC (không phải bên thực hiện nghĩa vụ)

---

**Trigger**
> Bên đối tác nhấn "Xác nhận" hoặc "Từ chối" trên nghĩa vụ đã có bằng chứng SUBMITTED.

---

**Main Flow (Basic Path) — Xác nhận**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Mở chi tiết nghĩa vụ đang ở trạng thái SUBMITTED |
| 2 | System | Hiển thị bằng chứng: mô tả, file đính kèm (cho phép tải về và xem) |
| 3 | Authenticated User | Kiểm tra bằng chứng |
| 4 | Authenticated User | Nhấn "Xác nhận hoàn thành" |
| 5 | System | Chuyển nghĩa vụ sang trạng thái CONFIRMED, ghi nhận confirmed_at và confirmed_by |
| 6 | System | Gửi thông báo cho bên thực hiện "Nghĩa vụ [tên] đã được xác nhận hoàn thành" |
| 7 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-27.a: Từ chối xác nhận (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | Authenticated User | Nhấn "Từ chối" |
| 4b | System | Hiển thị form nhập lý do từ chối (bắt buộc) |
| 4c | Authenticated User | Nhập lý do từ chối (ví dụ: "Số tiền chuyển không khớp hợp đồng") |
| 4d | System | Chuyển nghĩa vụ sang trạng thái DISPUTED, ghi nhận disputed_at |
| 4e | System | Gửi thông báo cho bên thực hiện kèm lý do từ chối |
| 4f | System | Bên thực hiện có thể nộp bằng chứng mới (quay lại UC-26) |
| 4g | System | Use case kết thúc |

---

**Exception Flows**

> EF-27.1: Actor là bên thực hiện (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện actor là bên chịu trách nhiệm (tự xác nhận) |
| 1b | System | Từ chối "Bạn không thể tự xác nhận nghĩa vụ của mình" |

---

**Postconditions**

*Success (Xác nhận):*
- Nghĩa vụ chuyển sang CONFIRMED
- Bên thực hiện được thông báo

*Success (Từ chối):*
- Nghĩa vụ chuyển sang DISPUTED
- Bên thực hiện được thông báo kèm lý do, có thể nộp bằng chứng mới

*Failure:*
- Nghĩa vụ không thay đổi

---

**Business Rules**

- BR-0603: Chỉ BÊN ĐỐI TÁC mới có quyền xác nhận/từ chối. Organizer xác nhận nghĩa vụ Sponsor, Sponsor xác nhận nghĩa vụ Organizer. Lý do từ chối BẮT BUỘC

---

**Notes / Assumptions**

- Cơ chế kiểm soát chéo: ngăn tự xác nhận
- Nghĩa vụ DISPUTED cho phép nộp lại bằng chứng mới (UC-26)
- Liên kết: UC-25, UC-26
