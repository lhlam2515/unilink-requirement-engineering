# UC-14: Trao đổi tin nhắn trong thương vụ

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) gửi và nhận tin nhắn văn bản trong phạm vi một deal (thương vụ) cụ thể. Hỗ trợ giao tiếp real-time, đính kèm file tài liệu, và theo dõi trạng thái đã đọc/chưa đọc. [UPDATED — BP03] Tin nhắn được áp dụng bộ lọc Data Masking (SF-13) che giấu thông tin liên hệ (SĐT, email, link MXH) khi deal chưa hoàn tất thanh toán phí dịch vụ. Hệ thống cũng phát hiện hành vi cố tình lách bộ lọc (anti-bypass).

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người gửi/nhận tin nhắn |
| Secondary | System | Xử lý real-time, gửi thông báo, xác thực file đính kèm |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái IN_PROGRESS, AWAITING_PAYMENT, hoặc AGREED
- Actor là một trong hai bên liên quan trong deal

---

**Trigger**
> Actor mở trang thương thảo của deal và nhập tin nhắn hoặc đính kèm file.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Mở trang thương thảo của deal |
| 2 | System | Hiển thị lịch sử tin nhắn theo thứ tự thời gian, đánh dấu trạng thái đã đọc/chưa đọc. Nếu deal.contact_unlocked = false: hiển thị nội dung đã masking (thông tin liên hệ bị che giấu) |
| 3 | Authenticated User | Nhập nội dung tin nhắn |
| 4 | Authenticated User | Nhấn "Gửi" |
| 5 | System | Lưu nội dung GỐC vào database. Nếu deal.contact_unlocked = false: áp dụng bộ lọc Data Masking — phát hiện SĐT, email, URL MXH, số tài khoản ngân hàng và thay thế bằng "***" khi hiển thị |
| 6 | System | Chạy anti-bypass detection song song — kiểm tra hành vi lách bộ lọc (viết số bằng chữ, chen ký tự đặc biệt, biến thể Unicode). Nếu phát hiện: xem EF-14.4 |
| 7 | System | Hiển thị tin nhắn (đã masking nếu cần) cho đối tác trong real-time |
| 8 | System | Nếu đối tác không đang online: gửi thông báo in-app |
| 9 | System | Cập nhật delivery_status sang DELIVERED khi đối tác nhận, READ khi đối tác đọc |
| 10 | System | Use case kết thúc thành công — tin nhắn đã gửi |

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

> EF-14.4: Phát hiện lách bộ lọc Data Masking (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện hành vi bypass (viết SĐT bằng chữ, chen ký tự đặc biệt, v.v.) |
| 6b | System | Chặn tin nhắn (status = FLAGGED), KHÔNG gửi đến đối tác |
| 6c | System | Cảnh báo người gửi: "Tin nhắn bị chặn: phát hiện hành vi trao đổi thông tin liên hệ trước khi thanh toán. Vi phạm: [N]/3" |
| 6d | System | Ghi BypassViolationLog (→ UC-53 admin review) |
| 6e | System | Nếu vi phạm ≥ 3: khóa tạm thời tính năng nhắn tin trong deal này |

---

**Postconditions**

*Success:*
- Tin nhắn được gửi và hiển thị cho đối tác
- Nếu deal chưa mở khóa: nội dung hiển thị đã masking (thông tin liên hệ bị che)
- File đính kèm (nếu có) được lưu trữ và có thể tải về
- Trạng thái delivery được theo dõi

*Success (sau khi deal mở khóa — 2/2 thanh toán):*
- Tất cả tin nhắn cũ hiển thị nội dung gốc không masking
- Tin nhắn mới không bị masking

*Failure:*
- Tin nhắn không được gửi
- Actor được thông báo lỗi cụ thể

*Failure (bypass detected):*
- Tin nhắn bị FLAGGED, không gửi đến đối tác
- Vi phạm được ghi log cho admin review (UC-53)

---

**Business Rules**

- BR-0401: Chỉ hai bên liên quan trong deal mới có quyền gửi tin nhắn
- BR-0402: Tin nhắn chỉ gửi được khi deal ở trạng thái IN_PROGRESS, AWAITING_PAYMENT, hoặc AGREED
- BR-0403: File đính kèm: định dạng PDF, DOCX, XLSX, JPEG, PNG, WebP. Tối đa 10MB/file, 5 file/tin nhắn
- BR-1301: Data Masking áp dụng cho SĐT, email, URL MXH, số tài khoản khi contact_unlocked = false
- BR-1303: Zero Tolerance — vi phạm bypass: 1-2 cảnh báo, 3 khóa tạm tính năng nhắn tin

---

**Notes / Assumptions**

- Hệ thống hỗ trợ real-time (WebSocket hoặc tương đương)
- Lịch sử tin nhắn được lưu vĩnh viễn trong deal context
- Data Masking chỉ áp dụng cho text message, KHÔNG quét nội dung file đính kèm
- Nội dung gốc LUÔN được lưu trong database; masking chỉ áp dụng khi hiển thị
- Liên kết: UC-15, UC-18, UC-19, UC-50 (mở khóa masking sau 2/2), UC-53 (admin review bypass)
