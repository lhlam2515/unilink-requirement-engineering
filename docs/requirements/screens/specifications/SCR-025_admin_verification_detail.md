# SCR-025: Admin_VerificationDetail_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-025 |
| **Screen Name** | Admin_VerificationDetail_Screen |
| **Mục đích** | Admin xem chi tiết hồ sơ xác thực tổ chức và thực hiện quyết định: phê duyệt, từ chối, hoặc yêu cầu bổ sung thông tin |
| **Actor chính** | Admin |
| **Quy trình nghiệp vụ** | BP-02 / Bước 5–6 — Kiểm duyệt và ra quyết định |
| **Use case liên quan** | UC-42, UC-43, UC-44, UC-45 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Xem xét chi tiết và ra quyết định kiểm duyệt |
| Data scope riêng | ✅ Chi tiết hồ sơ: thông tin, tài liệu, lịch sử — data set khác list |
| Action set riêng | ✅ Preview/download tài liệu, phê duyệt, từ chối, yêu cầu bổ sung |
| Navigation boundary | ✅ Context switch rõ ràng từ list → detail |
| Independently testable | ✅ Có thể viết acceptance criteria riêng |

> **Lưu ý thiết kế**: UC-43 (phê duyệt), UC-44 (từ chối), UC-45 (yêu cầu bổ sung) là **actions/modals** trên screen này, KHÔNG phải screens riêng. Lý do: cùng context (chi tiết hồ sơ), mỗi action là confirm dialog hoặc form modal đơn giản.

---

## Dữ liệu hiển thị (Read-only Data)

### Thông tin cơ bản

- Tên tổ chức, vai trò (Organizer/Sponsor), email, địa chỉ liên hệ
- Tên trường (Organizer) hoặc Lĩnh vực (Sponsor)

### Thông tin bổ sung

- Fanpage/website (Organizer) hoặc Mã số thuế (Sponsor)

### Tài liệu minh chứng

- Danh sách tài liệu: tên file, loại, kích thước, ngày tải lên
- Nút "Xem trước" (PDF viewer / image viewer) và "Tải về" cho mỗi tài liệu

### Lịch sử xác thực

- Tất cả lần gửi và xử lý theo thứ tự thời gian
- Mỗi entry: ngày, hành động (SUBMITTED/APPROVED/REJECTED/INFO_REQUESTED), lý do (nếu có)

---

## Dữ liệu nhập (Input Fields)

### Xác nhận phê duyệt (UC-43 — modal)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Ghi chú (Admin) | Textarea | Tùy chọn |

### Form từ chối (UC-44 — modal)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Lý do từ chối | Textarea | Bắt buộc (BR-1004) |

### Form yêu cầu bổ sung (UC-45 — modal)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Chi tiết cần bổ sung | Textarea | Bắt buộc (BR-1005) |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Phê duyệt** | Mở confirm dialog → System chuyển VERIFIED + mở khóa quyền → redirect SCR-024 (UC-43) |
| **Từ chối** | Mở form modal (nhập lý do bắt buộc) → System chuyển REJECTED → redirect SCR-024 (UC-44) |
| **Yêu cầu bổ sung** | Mở form modal (nhập chi tiết bắt buộc) → System chuyển INFO_REQUIRED → redirect SCR-024 (UC-45) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Xem trước tài liệu | Mở preview (PDF viewer / image viewer) — AF-42.a |
| Tải về tài liệu | Download file — AF-42.b |
| Quay lại danh sách | Chuyển về SCR-024 |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-1002: Admin có quyền xem/tải tất cả tài liệu. Lịch sử hiển thị TẤT CẢ lần gửi/xử lý
- BR-1003: Phê duyệt → VERIFIED + mở khóa quyền + retention 7 ngày
- BR-1004: Từ chối → rejection_reason bắt buộc, không cần tạo lại tài khoản
- BR-1005: Yêu cầu bổ sung → info_request_details bắt buộc
- BR-1006: Gửi thông báo email + in-app cho 6 sự kiện

---

## Quy tắc xác thực (Validation Rules)

- Lý do từ chối (UC-44): bắt buộc, không được bỏ trống
- Chi tiết yêu cầu bổ sung (UC-45): bắt buộc, không được bỏ trống

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-024 (Verification List) | Nhấn vào hồ sơ trong danh sách |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Phê duyệt thành công | SCR-024 + thông báo "Đã phê duyệt" |
| Từ chối thành công | SCR-024 + thông báo "Đã từ chối" |
| Yêu cầu bổ sung thành công | SCR-024 + thông báo "Đã gửi yêu cầu" |
| Nhấn "Quay lại" | SCR-024 |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải chi tiết hồ sơ |
| Document Preview | Admin nhấn "Xem trước" tài liệu (AF-42.a) |
| Approve Confirm Modal | Admin nhấn "Phê duyệt" (UC-43 Main-1~2) |
| Reject Form Modal | Admin nhấn "Từ chối" (UC-44 Main-1~2) |
| Info Request Form Modal | Admin nhấn "Yêu cầu bổ sung" (UC-45 Main-1~2) |
| Error State — Hồ sơ không tồn tại | Hồ sơ bị xóa hoặc ID sai (EF-42.1) |
| Error State — Đã xử lý trước đó | Admin khác đã xử lý (EF-43.1, EF-44.2, EF-45.2) |
| Error State — Không nhập lý do | Từ chối không nhập lý do (EF-44.1, EF-45.1) |

## UI Components liên quan

- Organization info card — thông tin cơ bản + bổ sung
- Document list with preview/download — danh sách tài liệu minh chứng
- PDF viewer / Image viewer — xem trước tài liệu
- Verification history timeline — lịch sử xác thực
- Confirm dialog (phê duyệt) — notes tùy chọn + confirm/cancel
- Reject form modal — lý do bắt buộc + confirm/cancel
- Info request form modal — chi tiết bắt buộc + confirm/cancel
- Action button group — Phê duyệt / Từ chối / Yêu cầu bổ sung

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-42 | Main-1 | Admin nhấn vào hồ sơ | Navigation In | Entry từ SCR-024 |
| UC-42 | Main-2~3 | System truy xuất, hiển thị thông tin cơ bản | Read-only | Org info card |
| UC-42 | Main-4 | Hiển thị thông tin bổ sung theo vai trò | Read-only | Additional info section |
| UC-42 | Main-5 | Hiển thị danh sách tài liệu + preview/download | Read-only | Document list |
| UC-42 | Main-6 | Hiển thị lịch sử xác thực | Read-only | History timeline |
| UC-42 | AF-42.a | Xem trước tài liệu | Component | PDF/Image viewer |
| UC-42 | AF-42.b | Tải về tài liệu | Action | Download button |
| UC-42 | AF-42.c | Hiển thị MST (Sponsor) | Read-only | MST field in info card |
| UC-42 | AF-42.d | Hiển thị lịch sử gửi lại | Read-only | History timeline entries |
| UC-42 | EF-42.1 | Hồ sơ không tồn tại | UI State | Error → redirect SCR-024 |
| UC-43 | Main-1~2 | Admin nhấn "Phê duyệt", system hiển thị xác nhận | Action | Approve confirm modal |
| UC-43 | Main-3 | Admin nhập ghi chú, xác nhận | Input + Action | Modal form + confirm |
| UC-43 | Main-4~12 | System xử lý phê duyệt | Action | System → redirect SCR-024 |
| UC-43 | EF-43.1 | Đã xử lý bởi admin khác | UI State | Error modal |
| UC-44 | Main-1~2 | Admin nhấn "Từ chối", system hiển thị form | Action | Reject form modal |
| UC-44 | Main-3~4 | Admin nhập lý do, xác nhận | Input + Action | Textarea + confirm |
| UC-44 | Main-5~11 | System xử lý từ chối | Action | System → redirect SCR-024 |
| UC-44 | EF-44.1 | Không nhập lý do | UI State | Inline error in modal |
| UC-44 | EF-44.2 | Đã xử lý trước đó | UI State | Error modal |
| UC-45 | Main-1~2 | Admin nhấn "Yêu cầu bổ sung", system hiển thị form | Action | Info request form modal |
| UC-45 | Main-3~4 | Admin nhập chi tiết, xác nhận | Input + Action | Textarea + confirm |
| UC-45 | Main-5~11 | System xử lý yêu cầu | Action | System → redirect SCR-024 |
| UC-45 | EF-45.1 | Không nhập chi tiết | UI State | Inline error in modal |
| UC-45 | EF-45.2 | Đã xử lý trước đó | UI State | Error modal |
