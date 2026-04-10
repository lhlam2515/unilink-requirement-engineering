# SCR-012: User_ContractEdit_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-012 |
| **Screen Name** | User_ContractEdit_Screen |
| **Mục đích** | Authenticated User soạn thảo và chỉnh sửa điều khoản hợp đồng tài trợ, xác nhận nội dung |
| **Actor chính** | Authenticated User (Organizer hoặc Sponsor) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 4 — Soạn thảo và ký kết hợp đồng tài trợ |
| **Use case liên quan** | UC-20, UC-21 |

> **[UPDATED — BP03]** Navigation In thay đổi: hợp đồng chỉ được tạo SAU KHI thanh toán phí dịch vụ hoàn tất (2/2). Entry point chính từ SCR-027 (Paywall) thay vì trực tiếp từ SCR-011.

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Soạn thảo hợp đồng — edit mode khác với negotiation |
| Data scope riêng | ✅ Toàn bộ điều khoản hợp đồng |
| Action set riêng | ✅ Edit terms, confirm content |
| Navigation boundary | ✅ Context switch từ deal negotiation sang contract edit |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

- Mã hợp đồng (contract_number)
- Trạng thái (status): DRAFTING / CONFIRMED
- Version number (version_number)
- Trạng thái xác nhận: organizer_content_confirmed, sponsor_content_confirmed
- Lịch sử chỉnh sửa: người sửa, thời gian, trường thay đổi, giá trị cũ → mới
- Thông tin pre-filled từ deal context (thông tin BTC, doanh nghiệp, sự kiện, gói tài trợ)

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Validation | Ghi chú |
|--------|------|------------|---------|
| Thông tin nhà tài trợ | Text inputs | Bắt buộc | sponsor_info |
| Thông tin BTC | Text inputs | Bắt buộc | organizer_info |
| Ngày ký kết | Date picker | Bắt buộc | signing_date |
| Thời gian hiệu lực (bắt đầu ~ kết thúc) | Date range | Bắt buộc | validity_start, validity_end |
| Thời hạn thực hiện giao dịch | Date/Text | Bắt buộc | execution_deadline |
| Hình thức tài trợ | Select | Bắt buộc | CASH / IN_KIND / COMBINED |
| Giá trị tài trợ | Number | Bắt buộc, > 0 | sponsorship_value |
| Quyền lợi nhà tài trợ | Rich text / List | Bắt buộc | sponsor_benefits |
| Cam kết và trách nhiệm BTC | Rich text / List | Bắt buộc | organizer_commitments |
| Cam kết và trách nhiệm doanh nghiệp | Rich text / List | Bắt buộc | sponsor_commitments |
| Quyền lợi hai bên | Rich text / List | Tùy chọn | mutual_rights |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| **Lưu thay đổi** | Lưu nội dung, tăng version, reset xác nhận (UC-20 Main) | DRAFTING |
| **Xác nhận nội dung** | Ghi nhận xác nhận cho bên hiện tại (UC-21 Main) | DRAFTING |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả | Điều kiện |
|-----------|---------------------|-----------|
| Xem lịch sử chỉnh sửa | Hiển thị audit log | — |
| Chuyển sang ký | Chuyển đến SCR-013 (Contract Sign) | CONFIRMED |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0501: Hợp đồng chỉ tạo từ deal AGREED (sau khi thanh toán 2/2 hoàn tất — SF-12). Mỗi deal chỉ 1 hợp đồng `[UPDATED — BP03]`
- BR-0502: Chỉ chỉnh sửa khi DRAFTING. SIGNED không cho phép
- BR-0503: Lưu lịch sử mọi thay đổi: người, thời gian, trường, giá trị cũ/mới
- BR-0504: Chỉnh sửa sau khi có xác nhận → reset TẤT CẢ xác nhận

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| SCR-027 (Service Fee Paywall) | Tự động redirect sau khi 2/2 thanh toán hoàn tất `[UPDATED — BP03]` |
| SCR-011 (Deal Negotiation) | Link "Soạn thảo hợp đồng" khi deal AGREED |
| SCR-010 (Deal List) | Nhấn deal có contract |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | SCR-011 (Deal Negotiation) hoặc SCR-010 |
| CONFIRMED + nhấn "Ký hợp đồng" | SCR-013 (Contract Sign) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải hợp đồng |
| Save Success Toast | Lưu thành công |
| Confirm Content Toast | Xác nhận nội dung thành công |
| Confirmation Status Banner | 0/2, 1/2, 2/2 bên xác nhận |
| Both Confirmed Banner | Cả hai xác nhận → hiển thị CTA "Ký hợp đồng" |
| Edit Warning (CONFIRMED) | Cảnh báo reset xác nhận khi sửa (UC-20 EF-20.2) |
| Signed Error | Hợp đồng đã ký, không chỉnh sửa (UC-20 EF-20.1) |
| Lock Window Info | Đếm ngược 72h hard-lock kể từ khi PaywallSession COMPLETED (SF-05 FR-0507, BR-0509) `[UPDATED — BP03]` |
| Version Change Notification | Đối tác vừa chỉnh sửa → cần xác nhận lại (UC-21 AF-21.b) |

## UI Components liên quan

- Form sections — các nhóm điều khoản
- Rich text editor — quyền lợi, cam kết
- Confirmation progress — hiển thị trạng thái xác nhận 2 bên
- Audit log panel — lịch sử chỉnh sửa
- Lock banner — trạng thái hard-lock và đếm ngược 72h
- Toast notifications
- Alert banner — cảnh báo reset xác nhận

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-20 | Main-1 | Mở trang soạn thảo | Screen entry | Page load |
| UC-20 | Main-2 | Hiển thị HĐ pre-filled | Read-only + Input | Form sections |
| UC-20 | Main-3 | Chỉnh sửa điều khoản | Input | Form fields |
| UC-20 | Main-4~8 | Lưu + validate + log + version | Action | Save button |
| UC-20 | Main-9 | Reset xác nhận | Action | System processing |
| UC-20 | Main-10 | Thông báo đối tác | Action | System notification |
| UC-20 | EF-20.1 | HĐ đã ký | UI State | Error block |
| UC-20 | EF-20.2 | HĐ CONFIRMED, cảnh báo | UI State | Warning dialog |
| UC-21 | Main-1~2 | Xem + xác nhận nội dung | Action | CTA "Xác nhận" |
| UC-21 | Main-3~8 | Ghi nhận + kiểm tra 2 bên | Action + State | Confirmation progress |
| UC-21 | AF-21.a | Chỉ 1 bên | UI State | Partial (1/2) |
| UC-21 | AF-21.b | Đối tác sửa → reset | UI State | Alert notification |
| FR-0507 | Main | Hard-lock 72h | UI State | Lock banner / countdown |
