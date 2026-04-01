# UC-14: Trao đổi tin nhắn trong thương vụ

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) gửi và nhận tin nhắn văn bản trong phạm vi một deal (thương vụ) cụ thể. Hỗ trợ giao tiếp real-time, đính kèm file tài liệu, và theo dõi trạng thái đã đọc/chưa đọc.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người gửi/nhận tin nhắn |
| Secondary | System | Xử lý real-time, gửi thông báo, xác thực file đính kèm |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái IN_PROGRESS hoặc AGREED
- Actor là một trong hai bên liên quan trong deal

---

**Trigger**
> Actor mở trang thương thảo của deal và nhập tin nhắn hoặc đính kèm file.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Mở trang thương thảo của deal |
| 2 | System | Hiển thị lịch sử tin nhắn theo thứ tự thời gian, đánh dấu trạng thái đã đọc/chưa đọc |
| 3 | Authenticated User | Nhập nội dung tin nhắn |
| 4 | Authenticated User | Nhấn "Gửi" |
| 5 | System | Lưu tin nhắn, ghi nhận sent_at và delivery_status = SENT |
| 6 | System | Hiển thị tin nhắn cho đối tác trong real-time |
| 7 | System | Nếu đối tác không đang online: gửi thông báo in-app |
| 8 | System | Cập nhật delivery_status sang DELIVERED khi đối tác nhận, READ khi đối tác đọc |
| 9 | System | Use case kết thúc thành công — tin nhắn đã gửi |

---

**Alternate Flows**

> AF-14.a: Gửi tin nhắn kèm file đính kèm (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Đính kèm file vào tin nhắn (tối đa 5 file) |
| 3b | System | Xác thực định dạng (PDF, DOCX, XLSX, JPEG, PNG, WebP) và kích thước (≤ 10MB/file) |
| 3c | System | Upload file thành công, tạo attachment_id |
| 3d | System | Hiển thị file đính kèm trong luồng tin nhắn, đối tác có thể tải về |
| 3e | System | Tiếp tục tại Step 5 |

---

**Exception Flows**

> EF-14.1: Deal đã kết thúc (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện deal ở trạng thái TERMINATED |
| 4b | System | Từ chối gửi tin nhắn với thông báo "Thương thảo đã kết thúc" |

> EF-14.2: File đính kèm không hợp lệ (triggered at Step 3b trong AF-14.a)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3b-a | System | Phát hiện file không đúng định dạng hoặc vượt quá 10MB |
| 3b-b | System | Từ chối upload với thông báo "Định dạng file không được hỗ trợ" hoặc "File vượt quá giới hạn 10MB" |
| 3b-c | Authenticated User | Chọn file khác hoặc gửi tin nhắn không có đính kèm |

> EF-14.3: Vượt quá giới hạn số file đính kèm (triggered at Step 3a trong AF-14.a)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a-a | System | Phát hiện đã đính kèm hơn 5 file |
| 3a-b | System | Từ chối với thông báo "Tối đa 5 file cho mỗi tin nhắn" |

---

**Postconditions**

*Success:*
- Tin nhắn được gửi và hiển thị cho đối tác
- File đính kèm (nếu có) được lưu trữ và có thể tải về
- Trạng thái delivery được theo dõi

*Failure:*
- Tin nhắn không được gửi
- Actor được thông báo lỗi cụ thể

---

**Business Rules**

- BR-0401: Chỉ hai bên liên quan trong deal mới có quyền gửi tin nhắn
- BR-0402: Tin nhắn chỉ gửi được khi deal ở trạng thái IN_PROGRESS hoặc AGREED
- BR-0403: File đính kèm: định dạng PDF, DOCX, XLSX, JPEG, PNG, WebP. Tối đa 10MB/file, 5 file/tin nhắn

---

**Notes / Assumptions**

- Hệ thống hỗ trợ real-time (WebSocket hoặc tương đương)
- Lịch sử tin nhắn được lưu vĩnh viễn trong deal context
- Liên kết: UC-15, UC-18, UC-19
