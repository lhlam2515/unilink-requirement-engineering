# UC-56: Tạo thỏa thuận nháp

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) tạo Thỏa thuận nháp (Draft Agreement) tóm tắt các điều khoản đã thống nhất trong quá trình thương thảo. Đối tác xem và xác nhận hoặc từ chối thỏa thuận. Thỏa thuận nháp đã xác nhận là cơ sở để tính phí dịch vụ và soạn hợp đồng.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Tạo thỏa thuận nháp |
| Secondary | Authenticated User (đối tác) | Xác nhận hoặc từ chối thỏa thuận |
| Secondary | System | Thông báo, lưu trữ, kiểm tra trạng thái |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái IN_PROGRESS
- Actor là một trong hai bên liên quan trong deal
- Không có thỏa thuận nháp PENDING_CONFIRMATION hoặc CONFIRMED nào tồn tại cho deal này

---

**Trigger**
> Actor nhấn "Tạo thỏa thuận nháp" trong trang thương thảo.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Nhấn "Tạo thỏa thuận nháp" trong trang thương thảo |
| 2 | System | Hiển thị form tạo thỏa thuận: Hình thức tài trợ, Giá trị tiền mặt (nếu có), Mô tả hiện vật (nếu có), Điều khoản chính |
| 3 | Authenticated User | Điền đầy đủ thông tin và nhấn "Gửi thỏa thuận" |
| 4 | System | Xác thực dữ liệu đầu vào |
| 5 | System | Tạo DraftAgreement với status = PENDING_CONFIRMATION |
| 6 | System | Gửi thông báo cho đối tác: "[Bên tạo] đã tạo thỏa thuận nháp. Vui lòng xem xét." |
| 7 | Đối tác | Xem nội dung thỏa thuận nháp |
| 8 | Đối tác | Nhấn "Xác nhận thỏa thuận" |
| 9 | System | Chuyển DraftAgreement sang CONFIRMED, ghi nhận confirmed_by và confirmed_at |
| 10 | System | Mở khóa nút "Xác nhận đồng thuận" (UC-18) |
| 11 | System | Gửi thông báo cho bên tạo: "Đối tác đã xác nhận thỏa thuận nháp" |
| 12 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-56.a: Đối tác từ chối thỏa thuận (triggered at Step 8)

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | Đối tác | Nhấn "Từ chối" và nhập ghi chú lý do |
| 8b | System | Chuyển DraftAgreement sang REJECTED, lưu rejection_note |
| 8c | System | Gửi thông báo cho bên tạo: "Đối tác đã từ chối thỏa thuận. Lý do: [ghi chú]" |
| 8d | System | Bên tạo có thể tạo thỏa thuận nháp mới (quay lại Step 1) |

> AF-56.b: Tài trợ kết hợp COMBINED (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Chọn hình thức COMBINED |
| 3b | System | Yêu cầu nhập CẢ giá trị tiền mặt VÀ mô tả hiện vật |
| 3c | Authenticated User | Điền đầy đủ cả hai phần |
| 3d | System | Tiếp tục Step 4 bình thường |

---

**Exception Flows**

> EF-56.1: Đã có thỏa thuận nháp đang chờ (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện đã có DraftAgreement PENDING_CONFIRMATION |
| 1b | System | Từ chối tạo mới: "Đã có thỏa thuận nháp đang chờ đối tác xác nhận" |

> EF-56.2: Thiếu giá trị tiền mặt cho hình thức CASH (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện sponsorship_type = CASH nhưng sponsorship_value trống |
| 4b | System | Hiển thị lỗi "Vui lòng nhập giá trị tài trợ tiền mặt" |

> EF-56.3: Thiếu mô tả hiện vật cho hình thức IN_KIND (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Phát hiện sponsorship_type = IN_KIND nhưng in_kind_description trống |
| 4b | System | Hiển thị lỗi "Vui lòng nhập mô tả hiện vật" |

---

**Postconditions**

*Success (xác nhận):*
- DraftAgreement ở trạng thái CONFIRMED
- Nút "Xác nhận đồng thuận" (UC-18) được mở khóa
- Thông tin sponsorship_type và sponsorship_value sẵn sàng cho tính phí (UC-50)

*Success (từ chối):*
- DraftAgreement ở trạng thái REJECTED
- Bên tạo có thể tạo lại thỏa thuận mới

*Failure:*
- Không tạo được DraftAgreement do thiếu dữ liệu

---

**Business Rules**

- BR-0407: Thỏa thuận nháp BẮT BUỘC trước khi đồng thuận. Mỗi deal chỉ có MỘT thỏa thuận CONFIRMED tại một thời điểm
- BR-1201: Thỏa thuận nháp xác định sponsorship_type và sponsorship_value — cơ sở tính phí dịch vụ

---

**Notes / Assumptions**

- Thỏa thuận nháp có thể bị từ chối nhiều lần — bên tạo sửa và gửi lại
- Bên nào cũng có thể tạo thỏa thuận nháp (không giới hạn Organizer hay Sponsor)
- Liên kết: UC-18 (đồng thuận — yêu cầu thỏa thuận CONFIRMED), UC-51 (xem trước phí), UC-50 (thanh toán phí)
