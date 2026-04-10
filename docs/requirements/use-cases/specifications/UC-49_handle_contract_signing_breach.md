# UC-49: Xử lý vi phạm ký kết hợp đồng

**Brief Description**
> Authenticated User (bên đã hoàn tất nghĩa vụ) báo cáo đối tác khi hợp đồng đã quá hạn ký 72 giờ nhưng chưa đủ 2 chữ ký. Hệ thống tạm thời đóng băng tài khoản bên bị tố cáo, gửi cảnh báo cuối cùng và cho thêm 24 giờ ân hạn để hoàn tất ký kết. Nếu hết ân hạn mà vẫn chưa đủ 2 chữ ký, hệ thống đóng thương vụ và kích hoạt chế tài.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (bên đã hoàn tất nghĩa vụ) | Người gửi report |
| Secondary | System | Kiểm tra deadline, đóng băng, gửi cảnh báo, thực thi chế tài |
| Secondary | Reported Party | Bên bị báo cáo |
| Secondary | Admin | Xử lý chế tài hoặc ngoại lệ nếu cần |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Contract đã vào trạng thái hard-lock sau 2/2 payment
- Đã qua signing_deadline_at + 72 giờ nhưng contract chưa SIGNED
- Actor là bên đã hoàn tất nghĩa vụ trong deal đó

---

**Trigger**
> Actor nhấn "Tố cáo đối tác" trên màn hình hợp đồng.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Nhấn "Tố cáo đối tác" |
| 2 | System | Kiểm tra contract đã quá hạn 72 giờ và chưa đủ 2 chữ ký |
| 3 | System | Hiển thị form report với lý do và bằng chứng tùy chọn |
| 4 | Authenticated User | Nhập lý do báo cáo và gửi report |
| 5 | System | Tạo contract breach case và chuyển trạng thái sang REPORTED |
| 6 | System | Tạm thời đóng băng tài khoản của bên bị tố cáo |
| 7 | System | Gửi cảnh báo cuối cùng, cho thêm 24 giờ ân hạn |
| 8 | System | Ghi nhận final_warning_expires_at |
| 9 | System | Use case kết thúc — chờ hết ân hạn hoặc bên vi phạm ký xong |

---

**Alternate Flows**

> AF-49.a: Bên vi phạm hoàn tất chữ ký trong 24 giờ ân hạn

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | Reported Party | Hoàn tất chữ ký còn thiếu |
| 7b | System | Chuyển contract sang SIGNED và đóng breach case |
| 7c | System | Gỡ đóng băng nếu chính sách cho phép |

---

**Exception Flows**

> EF-49.1: Chưa quá hạn 72 giờ

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Từ chối report với thông báo "Chưa đến thời điểm tố cáo" |

> EF-49.2: Report thiếu lý do

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Từ chối gửi report |
| 4b | System | Hiển thị lỗi "Lý do tố cáo là bắt buộc" |

---

**Postconditions**

*Success (report hợp lệ):*
- Contract breach case được tạo
- Bên bị tố cáo bị đóng băng tạm thời
- Final warning 24h được kích hoạt

*Success (hết ân hạn mà không ký):*
- Thương vụ bị đóng
- Chế tài tương ứng được thực thi

*Failure:*
- Không tạo report
- Trạng thái contract/deal không đổi

---

**Business Rules**

- BR-1406: Chỉ bên đã hoàn tất nghĩa vụ mới được mở quyền tố cáo khi quá hạn ký 72 giờ
- BR-1407: Sau report hợp lệ phải cấp thêm 24 giờ ân hạn
- BR-1408: Hết ân hạn mà chưa đủ 2 chữ ký thì phải đóng thương vụ và kích hoạt chế tài

---

**Notes / Assumptions**

- Use case này là nhánh hậu hard-lock, thay thế hoàn toàn cơ chế hủy đồng thuận trước đây
- Liên kết: SF-14, UC-22, UC-24, UC-25
