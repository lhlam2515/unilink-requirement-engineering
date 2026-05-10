# Use-Case Specification: UC-53 — Xem xét vi phạm lách bộ lọc

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Xem xét và quyết định xử lý vi phạm |
| Secondary | System | Phát hiện vi phạm, ghi log, thực thi chế tài |

---

### 1. Brief Description

> Admin xem xét các trường hợp vi phạm lách bộ lọc Data Masking (anti-bypass) do hệ thống phát hiện tự động. Admin đánh giá nội dung tin nhắn bị FLAGGED, quyết định xác nhận vi phạm, đánh dấu false positive, hoặc leo thang xử lý (khóa tài khoản vĩnh viễn theo chính sách Zero Tolerance).

---

### 2. Flow of Events

**Trigger**
> Admin truy cập danh sách vi phạm Data Masking hoặc nhận thông báo vi phạm mới.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Admin | Truy cập trang "Quản lý vi phạm Data Masking" |
| 2 | System | Hiển thị danh sách BypassViolationLog chưa review, sắp xếp mới nhất |
| 3 | Admin | Chọn một vi phạm để xem chi tiết |
| 4 | System | Hiển thị: nội dung gốc, phương pháp phát hiện, deal context, lịch sử vi phạm, hành động tự động đã áp dụng |
| 5 | Admin | Đánh giá và chọn "Xác nhận vi phạm" |
| 6 | System | Cập nhật: admin_reviewed = true, admin_decision = CONFIRMED_VIOLATION |
| 7 | System | Ghi nhận admin_reviewed_by và admin_reviewed_at |
| 8 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-53.a: Đánh dấu False Positive
>
> *Triggered at Step 5 of the Basic Flow when admin xác định chặn nhầm.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Admin | Chọn "False Positive — Chặn nhầm" |
| 5b | System | Cập nhật admin_decision = FALSE_POSITIVE |
| 5c | System | Gỡ FLAGGED, hiển thị lại nội dung gốc cho người nhận |
| 5d | System | Giảm violation_count (nếu > 0) |
| 5e | System | Gỡ khóa tạm thời nhắn tin nếu đã bị khóa do vi phạm này |

##### AF-53.b: Leo thang — Khóa tài khoản vĩnh viễn
>
> *Triggered at Step 5 of the Basic Flow when admin quyết định leo thang.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Admin | Chọn "Leo thang — Khóa tài khoản vĩnh viễn" |
| 5b | System | Hiển thị xác nhận kèm lý do bắt buộc |
| 5c | Admin | Xác nhận và nhập lý do |
| 5d | System | Cập nhật admin_decision = ESCALATED |
| 5e | System | Khóa tài khoản vi phạm vĩnh viễn |
| 5f | System | Thông báo user: "Tài khoản đã bị khóa vĩnh viễn do vi phạm chính sách Zero Tolerance" |

#### 2.3 Exception Flows

##### EF-53.1: Vi phạm đã được review trước đó
>
> *Triggered at Step 3 of the Basic Flow when vi phạm đã review.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Hiển thị "Đã xử lý bởi [admin]" với quyết định trước đó |
| 3b | Admin | Xem chi tiết nhưng không thể thay đổi |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-53-01 | Xác nhận vi phạm | Admin xác nhận vi phạm thực sự |
| SC-53-02 | False positive | Admin gỡ FLAGGED, khôi phục nội dung (AF-53.a) |
| SC-53-03 | Leo thang khóa vĩnh viễn | Admin khóa tài khoản vĩnh viễn (AF-53.b) |

---

### 5. Preconditions

#### 5.1 Admin đã xác thực

- Admin đã đăng nhập với quyền quản trị

#### 5.2 Có vi phạm chưa review

- Có ít nhất một BypassViolationLog chưa review (admin_reviewed = false)

---

### 6. Postconditions

#### 6.1 Success (xác nhận vi phạm)

- BypassViolationLog đã review, chế tài giữ nguyên

#### 6.2 Success (false positive)

- Tin nhắn gỡ FLAGGED, violation_count giảm

#### 6.3 Success (leo thang)

- Tài khoản bị khóa vĩnh viễn, user thông báo

#### 6.4 Failure

- Vi phạm vẫn chờ review

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1303: Zero Tolerance — vi phạm 1-2: cảnh báo, 3: khóa tạm, 3+: admin review → có thể khóa vĩnh viễn. Thuật toán ưu tiên false positive hơn false negative

---

### 10. Additional Information

**Assumptions:**

- Hệ thống phát hiện vi phạm tự động (FR-1303) và áp dụng chế tài ban đầu — UC này là bước review của admin
- Appeal từ user (kháng cáo) là ngoài phạm vi — xử lý qua kênh hỗ trợ

**Related Use Cases:**

- UC-14: Nhắn tin trong thương thảo (sequential — nơi masking và bypass detection hoạt động)
