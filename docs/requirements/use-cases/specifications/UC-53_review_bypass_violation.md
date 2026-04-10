# UC-53: Xem xét vi phạm lách bộ lọc

**Brief Description**
> Admin xem xét các trường hợp vi phạm lách bộ lọc Data Masking (anti-bypass) do hệ thống phát hiện tự động. Admin đánh giá nội dung tin nhắn bị FLAGGED, quyết định xác nhận vi phạm, đánh dấu false positive, hoặc leo thang xử lý (khóa tài khoản vĩnh viễn theo chính sách Zero Tolerance).

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Xem xét và quyết định xử lý vi phạm |
| Secondary | System | Phát hiện vi phạm, ghi log, thực thi chế tài |

---

**Preconditions**

- Admin đã đăng nhập vào hệ thống với quyền quản trị
- Có ít nhất một BypassViolationLog chưa được review (admin_reviewed = false)

---

**Trigger**
> Admin truy cập danh sách vi phạm Data Masking hoặc nhận thông báo vi phạm mới.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Admin | Truy cập trang "Quản lý vi phạm Data Masking" |
| 2 | System | Hiển thị danh sách các BypassViolationLog chưa review, sắp xếp theo thời gian mới nhất |
| 3 | Admin | Chọn một vi phạm để xem chi tiết |
| 4 | System | Hiển thị: nội dung tin nhắn gốc, phương pháp phát hiện (REGEX/DICTIONARY/HEURISTIC), deal context, lịch sử vi phạm của user, số lần vi phạm, hành động đã áp dụng tự động |
| 5 | Admin | Đánh giá nội dung và chọn quyết định: "Xác nhận vi phạm" |
| 6 | System | Cập nhật BypassViolationLog: admin_reviewed = true, admin_decision = CONFIRMED_VIOLATION |
| 7 | System | Ghi nhận admin_reviewed_by và admin_reviewed_at |
| 8 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-53.a: Đánh dấu False Positive (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Admin | Chọn "False Positive — Chặn nhầm" |
| 5b | System | Cập nhật admin_decision = FALSE_POSITIVE |
| 5c | System | Gỡ FLAGGED cho tin nhắn, hiển thị lại nội dung gốc cho người nhận |
| 5d | System | Giảm violation_count của user (nếu > 0) |
| 5e | System | Gỡ khóa tạm thời tính năng nhắn tin nếu đã bị khóa do vi phạm này |

> AF-53.b: Leo thang — Khóa tài khoản vĩnh viễn (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Admin | Chọn "Leo thang — Khóa tài khoản vĩnh viễn" |
| 5b | System | Hiển thị xác nhận: "Hành động này sẽ khóa vĩnh viễn tài khoản [tên]. Tiếp tục?" |
| 5c | Admin | Xác nhận khóa và nhập lý do |
| 5d | System | Cập nhật admin_decision = ESCALATED |
| 5e | System | Khóa tài khoản vi phạm vĩnh viễn |
| 5f | System | Gửi thông báo cho user bị khóa: "Tài khoản đã bị khóa vĩnh viễn do vi phạm chính sách Zero Tolerance" |

---

**Exception Flows**

> EF-53.1: Vi phạm đã được review trước đó (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện vi phạm đã được review bởi admin khác |
| 3b | System | Hiển thị trạng thái "Đã xử lý bởi [admin]" với quyết định trước đó |
| 3c | Admin | Có thể xem chi tiết nhưng không thể thay đổi quyết định |

---

**Postconditions**

*Success (xác nhận vi phạm):*
- BypassViolationLog được đánh dấu reviewed
- Chế tài tự động tiếp tục áp dụng (FLAGGED giữ nguyên)

*Success (false positive):*
- Tin nhắn được gỡ FLAGGED và hiển thị lại
- Violation count giảm

*Success (leo thang):*
- Tài khoản bị khóa vĩnh viễn
- User bị thông báo

*Failure:*
- Vi phạm vẫn chờ review

---

**Business Rules**

- BR-1303: Zero Tolerance — vi phạm 1-2: cảnh báo, 3: khóa tạm, 3+: admin review → có thể khóa vĩnh viễn
- BR-1303: Thuật toán ưu tiên false positive hơn false negative

---

**Notes / Assumptions**

- Hệ thống phát hiện vi phạm tự động (FR-1303) và áp dụng chế tài ban đầu — UC này là bước review của admin
- Appeal từ user (kháng cáo false positive) là ngoài phạm vi UC này — xử lý qua kênh hỗ trợ
- Liên kết: UC-14 (nhắn tin — nơi masking và bypass detection hoạt động)
