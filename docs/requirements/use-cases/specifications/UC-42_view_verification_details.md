# UC-42: Xem chi tiết hồ sơ xác thực

**Brief Description**
> Admin xem toàn bộ thông tin chi tiết của một hồ sơ xác thực tổ chức, bao gồm thông tin cơ bản, thông tin bổ sung, tài liệu minh chứng (có thể xem/tải về), và lịch sử các lần gửi/xử lý hồ sơ trước đó. Đây là bước bắt buộc trước khi thực hiện quyết định phê duyệt, từ chối, hoặc yêu cầu bổ sung.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Người xem xét hồ sơ để đánh giá |
| Secondary | System | Truy xuất và hiển thị chi tiết hồ sơ |

---

**Preconditions**

- Admin đã đăng nhập vào hệ thống với vai trò `admin`
- Hồ sơ xác thực tồn tại trong hệ thống

---

**Trigger**
> Admin nhấn vào một hồ sơ trong danh sách chờ duyệt (UC-41).

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Admin | Nhấn vào hồ sơ cần xem chi tiết trong danh sách |
| 2 | System | Truy xuất thông tin đầy đủ của hồ sơ xác thực |
| 3 | System | Hiển thị thông tin cơ bản: tên tổ chức, vai trò, email, địa chỉ liên hệ |
| 4 | System | Hiển thị thông tin bổ sung theo vai trò (fanpage, MST, giấy tờ...) |
| 5 | System | Hiển thị danh sách tài liệu minh chứng đã tải lên với nút preview/download |
| 6 | System | Hiển thị lịch sử xác thực (tất cả lần gửi và xử lý trước đó theo thứ tự thời gian) |
| 7 | System | Use case kết thúc thành công — chi tiết hồ sơ đã được hiển thị |

---

**Alternate Flows**

> AF-42.a: Xem trước tài liệu minh chứng (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Admin | Nhấn nút "Xem trước" trên một tài liệu |
| 5b | System | Hiển thị preview tài liệu (PDF viewer hoặc image viewer) |

> AF-42.b: Tải về tài liệu minh chứng (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5c | Admin | Nhấn nút "Tải về" trên một tài liệu |
| 5d | System | Tải file tài liệu về máy Admin |

> AF-42.c: Hồ sơ Sponsor có mã số thuế (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Hiển thị mã số thuế của doanh nghiệp |
| 4b | Admin | Đối chiếu MST thủ công với nguồn bên ngoài hệ thống |

> AF-42.d: Hồ sơ đã từng bị xử lý trước đó (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Hiển thị lịch sử: lần gửi trước + quyết định (REJECTED/INFO_REQUIRED) + lý do + lần gửi hiện tại |

---

**Exception Flows**

> EF-42.1: Hồ sơ không tồn tại (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Không tìm thấy hồ sơ xác thực với ID được yêu cầu |
| 2b | System | Hiển thị "Hồ sơ không tồn tại hoặc đã bị xóa" |
| 2c | System | Chuyển Admin quay lại danh sách (UC-41) |

---

**Postconditions**

*Success:*

- Admin đã xem đầy đủ thông tin hồ sơ xác thực
- Admin sẵn sàng ra quyết định: phê duyệt (UC-43), từ chối (UC-44), hoặc yêu cầu bổ sung (UC-45)

*Failure:*

- Không hiển thị được chi tiết hồ sơ
- Admin quay lại danh sách hồ sơ

---

**Business Rules**

- BR-1002: Admin phải có quyền xem và tải về tất cả tài liệu minh chứng. Lịch sử xác thực hiển thị TẤT CẢ lần gửi/xử lý theo thứ tự thời gian

---

**Notes / Assumptions**

- Việc kiểm tra MST doanh nghiệp được thực hiện THỦ CÔNG — Admin đối chiếu bên ngoài hệ thống
- UC-42 là base UC cho ba extend: UC-43 (Phê duyệt), UC-44 (Từ chối), UC-45 (Yêu cầu bổ sung)
- Liên kết: UC-41 (<<include>> từ danh sách), UC-43, UC-44, UC-45 (<<extend>>)
