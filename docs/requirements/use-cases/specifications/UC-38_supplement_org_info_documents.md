# UC-38: Bổ sung thông tin và tài liệu minh chứng

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) bổ sung thông tin tùy chọn và tải lên tài liệu minh chứng cho hồ sơ tổ chức sau khi hoàn tất đăng ký cơ bản. Tài liệu minh chứng khác nhau theo vai trò và phải đạt yêu cầu về định dạng và kích thước.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người bổ sung hồ sơ tổ chức |
| Secondary | System | Xác thực file upload, lưu trữ tài liệu |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Actor đã hoàn tất đăng ký cơ bản (UC-34 hoặc UC-35)
- Hồ sơ tổ chức chưa ở trạng thái VERIFIED (hoặc đã bị REJECTED/INFO_REQUIRED)

---

**Trigger**
> Actor truy cập trang "Hồ sơ tổ chức" và chọn "Bổ sung thông tin".

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Truy cập trang "Hồ sơ tổ chức" |
| 2 | System | Hiển thị thông tin hiện tại của tổ chức và danh sách tài liệu đã tải lên (nếu có) |
| 3 | Authenticated User | Chọn "Bổ sung thông tin" |
| 4 | System | Hiển thị form bổ sung thông tin tùy chọn và khu vực upload tài liệu minh chứng tương ứng với vai trò |
| 5 | Authenticated User | Nhập thông tin tùy chọn và/hoặc chọn file tài liệu minh chứng để tải lên |
| 6 | System | Xác thực định dạng file (chỉ chấp nhận PDF, JPEG, PNG, WebP) theo BR-0901 |
| 7 | System | Xác thực kích thước file không vượt quá 10MB theo BR-0901 |
| 8 | System | Kiểm tra trùng tên file trong cùng loại tài liệu theo BR-0908. Nếu không trùng, lưu trữ tài liệu và gán document_id (UUID) duy nhất |
| 9 | System | Cập nhật hồ sơ tổ chức với thông tin bổ sung |
| 10 | System | Hiển thị tài liệu đã tải lên trong danh sách minh chứng với trạng thái thành công |
| 11 | System | Use case kết thúc thành công — hồ sơ đã được bổ sung thông tin và tài liệu |

---

**Alternate Flows**

> AF-38.a: Tài liệu minh chứng cho Organizer (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Hiển thị form với: Giấy quyết định thành lập CLB (file), Giấy giới thiệu đoàn trường (file), Đường dẫn fanpage/website (text, optional) |
| – | – | Tiếp tục từ Step 5 |

> AF-38.b: Tài liệu minh chứng cho Sponsor (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4b | System | Hiển thị form với: Mã số thuế (text, optional), Giấy tờ chứng thực hoạt động (file), Giấy phép kinh doanh (file) |
| – | – | Tiếp tục từ Step 5 |

> AF-38.c: Bổ sung nhiều tài liệu (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Authenticated User | Chọn thêm file tài liệu khác để tải lên |
| – | – | Hệ thống lặp lại Step 6–10 cho mỗi file |

> AF-38.d: File trùng tên trong cùng loại tài liệu (triggered at Step 8)

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | System | Phát hiện đã tồn tại file có cùng tên trong loại tài liệu đang upload |
| 8b | System | Hiển thị xác nhận "File [tên file] đã tồn tại. Bạn muốn thay thế file cũ không?" |
| 8c | Authenticated User | Chọn "Thay thế" → System xóa file cũ và lưu file mới với cùng document_id |
| 8d | Authenticated User | Hoặc chọn "Hủy" → Quay lại Step 5, file cũ giữ nguyên |

> AF-38.e: Đổi tên file khi upload (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5b | Authenticated User | Chỉnh sửa tên hiển thị (display_name) của file trước khi xác nhận upload |
| 5c | System | Lưu file với display_name do người dùng chỉ định (giữ nguyên file gốc) |

---

**Exception Flows**

> EF-38.1: File vượt quá kích thước cho phép (triggered at Step 7)

| Step | Actor / System | Action |
|------|----------------|--------|
| 7a | System | Phát hiện file có kích thước vượt quá 10MB |
| 7b | System | Hiển thị "Kích thước file vượt quá giới hạn 10MB" |
| 7c | Authenticated User | Chọn file khác có kích thước phù hợp |

> EF-38.2: Định dạng file không hợp lệ (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện file có định dạng không được hỗ trợ |
| 6b | System | Hiển thị "Chỉ chấp nhận định dạng PDF, JPEG, PNG, WebP" |
| 6c | Authenticated User | Chọn file có định dạng phù hợp |

> EF-38.3: Đã quá hạn 14 ngày bổ sung (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện đã quá 14 ngày kể từ ngày tạo tài khoản (BR-0902) |
| 3b | System | Hiển thị cảnh báo "Đã quá hạn bổ sung thông tin. Vui lòng liên hệ hỗ trợ." |

---

**Postconditions**

*Success:*

- Tài liệu minh chứng đã được lưu trữ với document_id duy nhất
- Thông tin tùy chọn đã được cập nhật trong hồ sơ tổ chức
- Hồ sơ sẵn sàng để gửi xác thực (UC-40)

*Failure:*

- Tài liệu không được lưu trữ
- Actor được thông báo về lỗi cụ thể (kích thước, định dạng)

---

**Business Rules**

- BR-0901: Tài liệu minh chứng phải có định dạng PDF, JPEG, PNG, hoặc WebP. Kích thước tối đa 10MB/file
- BR-0902: Thông tin và tài liệu bổ sung cần hoàn tất trong vòng 14 ngày. Hệ thống gửi nhắc nhở khi còn 3 ngày
- BR-0908: Khi upload file trùng tên trong cùng loại tài liệu: hệ thống hỏi xác nhận thay thế. Nếu tên file khác thì bổ sung. Người dùng có thể đổi tên hiển thị file trước khi upload

---

**Notes / Assumptions**

- Actor có thể bổ sung thông tin nhiều lần (nhiều phiên khác nhau) miễn là trong thời hạn 14 ngày
- Tài liệu minh chứng sẽ được xóa tự động sau 7 ngày kể từ khi hồ sơ được xử lý hoàn tất (FR-0905 — nhúng vào system behavior)
- Liên kết: UC-34 / UC-35 (đăng ký), UC-39 (chỉnh sửa), UC-40 (gửi xác thực)
