# SCR-023: User_OrganizationProfile_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-023 |
| **Screen Name** | User_OrganizationProfile_Screen |
| **Mục đích** | Authenticated User xem, bổ sung, chỉnh sửa hồ sơ tổ chức và gửi hồ sơ xác thực. Đây là hub quản lý toàn bộ thông tin tổ chức — bao gồm thông tin cơ bản, thông tin bổ sung, tài liệu minh chứng, và trạng thái xác thực |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-02 / Bước 3–4 — Bổ sung thông tin và gửi hồ sơ xác thực |
| **Use case liên quan** | UC-38, UC-39, UC-40 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Quản lý hồ sơ tổ chức — mục tiêu duy nhất và rõ ràng |
| Data scope riêng | ✅ Hồ sơ tổ chức: thông tin cơ bản, bổ sung, tài liệu minh chứng, trạng thái |
| Action set riêng | ✅ Xem, chỉnh sửa, bổ sung tài liệu, gửi xác thực |
| Navigation boundary | ✅ Trang riêng /organization/profile, entry từ menu chính |
| Independently testable | ✅ Có thể viết acceptance criteria cho view, edit, upload, submit riêng |

> **Lưu ý thiết kế**: UC-38 (bổ sung), UC-39 (chỉnh sửa), UC-40 (gửi xác thực) được gộp vào CÙNG MỘT screen vì hoạt động trên cùng data scope (hồ sơ tổ chức), trong cùng context, và thường được thực hiện liên tiếp. Chỉnh sửa và bổ sung chuyển screen sang edit mode, gửi xác thực là CTA action.

---

## Dữ liệu hiển thị (Read-only Data)

### Khu vực trạng thái

- Trạng thái xác thực hiện tại (verification_status): UNVERIFIED / PENDING_REVIEW / VERIFIED / REJECTED / INFO_REQUIRED
- Badge trạng thái với màu sắc phân biệt
- Lý do từ chối (nếu REJECTED) — từ lần kiểm duyệt gần nhất
- Chi tiết yêu cầu bổ sung (nếu INFO_REQUIRED) — từ Admin
- Thời gian còn lại để bổ sung thông tin (14 ngày deadline — BR-0902)

### Khu vực thông tin cơ bản

- Tên tổ chức (org_name)
- Vai trò (Organizer / Sponsor) — read-only, không thể thay đổi
- Email liên hệ
- Địa chỉ liên hệ
- Tên trường (nếu Organizer) hoặc Lĩnh vực (nếu Sponsor)

### Khu vực thông tin bổ sung

- Đường dẫn fanpage/website (Organizer) hoặc Mã số thuế (Sponsor)

### Khu vực tài liệu minh chứng

- Danh sách tài liệu đã tải lên: tên file, kích thước, ngày tải, trạng thái
- Preview thumbnail (cho ảnh)

---

## Dữ liệu nhập (Input Fields)

### Mode chỉnh sửa (Edit Mode — UC-39)

#### Thông tin cơ bản

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Tên tổ chức | Text input | Bắt buộc |
| Tên trường (Organizer) / Lĩnh vực (Sponsor) | Text input | Bắt buộc |
| Địa chỉ liên hệ | Text input | Bắt buộc |

#### Thông tin bổ sung (UC-38)

**Organizer:**

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Giấy quyết định thành lập CLB | File upload | PDF, JPEG, PNG, WebP. Max 10MB |
| Giấy giới thiệu đoàn trường | File upload | PDF, JPEG, PNG, WebP. Max 10MB |
| Đường dẫn fanpage/website | Text input (URL) | Tùy chọn |

**Sponsor:**

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Mã số thuế | Text input | Tùy chọn |
| Giấy tờ chứng thực hoạt động | File upload | PDF, JPEG, PNG, WebP. Max 10MB |
| Giấy phép kinh doanh | File upload | PDF, JPEG, PNG, WebP. Max 10MB |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Chỉnh sửa** | Chuyển sang Edit Mode — hiển thị form chỉnh sửa (UC-39 Main-3~5) |
| **Lưu thay đổi** (trong Edit Mode) | System cập nhật hồ sơ → hiển thị thông báo thành công (UC-39 Main-7~10) |
| **Gửi hồ sơ xác thực** | System kiểm tra đầy đủ → chuyển PENDING_REVIEW → thông báo (UC-40 Main-3~12) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Bổ sung thông tin | Chuyển sang chế độ bổ sung — hiển thị form + upload area (UC-38 Main-3~4) |
| Tải lên tài liệu | Upload file → hiển thị trong danh sách (UC-38 Main-5~10) |
| Xóa tài liệu đã tải | Xóa file khỏi danh sách (trước khi gửi xác thực) |
| Hủy chỉnh sửa | Quay lại View Mode, bỏ thay đổi chưa lưu |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0901: Tài liệu minh chứng phải có định dạng PDF, JPEG, PNG, hoặc WebP. Kích thước tối đa 10MB/file
- BR-0902: Thông tin và tài liệu bổ sung cần hoàn tất trong vòng 14 ngày. Hệ thống gửi nhắc nhở khi còn 3 ngày
- BR-0903: Quyền chỉnh sửa phụ thuộc verification_status (UNVERIFIED: full edit; PENDING_REVIEW: edit + đánh dấu; VERIFIED: không chỉnh sửa; REJECTED/INFO_REQUIRED: full edit)
- BR-0904: Hồ sơ chỉ được gửi khi đầy đủ thông tin bắt buộc và email đã xác minh
- BR-0905: Chỉ cho phép gửi khi status là UNVERIFIED/REJECTED/INFO_REQUIRED

---

## Quy tắc xác thực (Validation Rules)

- Tên tổ chức: bắt buộc, không được bỏ trống
- Tên trường (Organizer) / Lĩnh vực (Sponsor): bắt buộc
- Địa chỉ liên hệ: bắt buộc
- File upload: chỉ PDF, JPEG, PNG, WebP; max 10MB mỗi file
- Gửi xác thực: email phải đã xác minh (email_verified = true)

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Dashboard / Menu chính | Menu "Hồ sơ tổ chức" |
| Thông báo (email/in-app) | Link trong thông báo yêu cầu bổ sung hoặc từ chối |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Lưu thay đổi thành công | Quay lại View Mode trên cùng screen |
| Gửi xác thực thành công | Cùng screen, status cập nhật thành PENDING_REVIEW |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| View Mode | Mặc định khi truy cập — hiển thị thông tin read-only |
| Edit Mode | Nhấn "Chỉnh sửa" hoặc "Bổ sung thông tin" |
| Loading State | Đang tải hồ sơ hoặc đang upload file |
| Upload Progress | File đang tải lên — hiển thị progress bar |
| Status Banner — UNVERIFIED | Hiển thị hướng dẫn bổ sung thông tin và gửi xác thực |
| Status Banner — PENDING_REVIEW | Hiển thị cảnh báo "đang chờ kiểm duyệt" khi chỉnh sửa (AF-39.a) |
| Status Banner — REJECTED | Hiển thị lý do từ chối + hướng dẫn chỉnh sửa và gửi lại (AF-39.b) |
| Status Banner — INFO_REQUIRED | Hiển thị chi tiết yêu cầu bổ sung từ Admin (AF-39.c) |
| Status Banner — VERIFIED | Hiển thị "Hồ sơ đã xác thực", ẩn nút chỉnh sửa (EF-39.1) |
| Error State — File quá lớn | File vượt 10MB (EF-38.1) |
| Error State — Định dạng sai | File không phải PDF/JPEG/PNG/WebP (EF-38.2) |
| Error State — Quá hạn bổ sung | Quá 14 ngày (EF-38.3) |
| Error State — Email chưa xác minh | email_verified = false khi gửi xác thực (EF-40.2) |
| Error State — Thiếu thông tin bắt buộc | Gửi xác thực khi chưa đủ thông tin (EF-40.1) |
| Error State — Trạng thái không cho phép | PENDING_REVIEW khi gửi lại (EF-40.3) hoặc VERIFIED (EF-40.4) |
| Success State | Thông báo "Cập nhật thành công" hoặc "Hồ sơ đã được gửi" |

## UI Components liên quan

- Verification status badge — trạng thái xác thực hiện tại
- Status info banner — thông tin chi tiết theo trạng thái (lý do từ chối, yêu cầu bổ sung)
- Organization info card — hiển thị thông tin tổ chức
- Edit form — form chỉnh sửa thông tin (toggle View/Edit mode)
- File upload area — khu vực kéo thả hoặc chọn file
- Document list — danh sách tài liệu đã tải lên với preview/delete
- Progress bar — tiến trình upload file
- CTA "Gửi hồ sơ xác thực" — nút chính, disabled khi chưa đủ điều kiện
- Countdown timer — đếm ngược deadline 14 ngày (nếu applicable)

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-38 | Main-1~2 | Truy cập trang hồ sơ, system hiển thị thông tin | Read-only | View Mode |
| UC-38 | Main-3~4 | Chọn "Bổ sung thông tin", system hiển thị form | Action | Edit Mode toggle + upload area |
| UC-38 | Main-5 | Nhập thông tin/chọn file | Input | Form fields + file picker |
| UC-38 | Main-6~7 | System xác thực file | Action | Upload validation |
| UC-38 | Main-8~10 | System lưu tài liệu, hiển thị trong danh sách | UI State | Document list update |
| UC-38 | EF-38.1 | File quá 10MB | UI State | Error alert |
| UC-38 | EF-38.2 | Định dạng không hợp lệ | UI State | Error alert |
| UC-38 | EF-38.3 | Quá hạn 14 ngày | UI State | Warning banner |
| UC-39 | Main-1~2 | Truy cập trang, system hiển thị thông tin | Read-only | View Mode |
| UC-39 | Main-3~5 | Nhấn "Chỉnh sửa", system kiểm tra quyền, hiển thị form | Action | Edit Mode toggle |
| UC-39 | Main-6~7 | Cập nhật thông tin, nhấn "Lưu" | Input + Action | Form fields + CTA |
| UC-39 | Main-8~10 | System xác thực, cập nhật, thông báo | Action | System xử lý + success toast |
| UC-39 | AF-39.a | PENDING_REVIEW — cảnh báo và đánh dấu | UI State | Status banner warning |
| UC-39 | AF-39.b | REJECTED — hiển thị lý do từ chối | UI State | Status info banner |
| UC-39 | AF-39.c | INFO_REQUIRED — hiển thị yêu cầu bổ sung | UI State | Status info banner |
| UC-39 | EF-39.1 | VERIFIED — không thể chỉnh sửa | UI State | Edit button disabled/hidden |
| UC-39 | EF-39.2 | Thiếu trường bắt buộc | UI State | Inline field errors |
| UC-40 | Main-1~2 | Truy cập hồ sơ, system hiển thị nút gửi | Read-only | CTA "Gửi hồ sơ xác thực" |
| UC-40 | Main-3 | Nhấn "Gửi hồ sơ xác thực" | Action | CTA button |
| UC-40 | Main-4~6 | System xác thực email, thông tin, trạng thái | Action | System validation |
| UC-40 | Main-7~12 | Tạo request, chuyển PENDING_REVIEW, thông báo | Action | Status update + success state |
| UC-40 | EF-40.1 | Thiếu thông tin bắt buộc | UI State | Error listing + link UC-38/39 |
| UC-40 | EF-40.2 | Email chưa xác minh | UI State | Warning + resend verification link |
| UC-40 | EF-40.3 | Đang PENDING_REVIEW | UI State | CTA disabled + info message |
| UC-40 | EF-40.4 | Đã VERIFIED | UI State | CTA hidden + verified badge |
