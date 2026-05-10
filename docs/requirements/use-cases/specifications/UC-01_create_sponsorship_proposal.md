# Use-Case Specification: UC-01 — Tạo hồ sơ tài trợ sự kiện

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người khởi tạo hồ sơ tài trợ |
| Secondary | System | Gán mã định danh, ghi nhận thời gian tạo |

---

### 1. Brief Description

> Organizer khởi tạo một hồ sơ tài trợ sự kiện mới trên hệ thống. Hệ thống tạo hồ sơ với trạng thái ban đầu DRAFT, gán mã định danh duy nhất, và chuyển organizer đến trang chỉnh sửa để tiếp tục soạn thảo nội dung.

---

### 2. Flow of Events

**Trigger**
> Organizer chọn chức năng "Tạo hồ sơ tài trợ mới" từ dashboard hoặc menu quản lý hồ sơ.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Organizer | Chọn chức năng "Tạo hồ sơ tài trợ mới" |
| 2 | System | Tạo hồ sơ mới với trạng thái DRAFT |
| 3 | System | Gán proposal_id (UUID) duy nhất cho hồ sơ |
| 4 | System | Gán hồ sơ cho tổ chức BTC mà tài khoản organizer đại diện |
| 5 | System | Ghi nhận thời gian tạo (created_at) và người tạo (created_by) |
| 6 | System | Chuyển organizer đến trang chỉnh sửa hồ sơ tài trợ |
| 7 | System | Use case kết thúc thành công — hồ sơ đã được khởi tạo ở trạng thái DRAFT |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-01.1: Không thể tạo hồ sơ do lỗi hệ thống
>
> *Triggered at Step 2 of the Basic Flow when hệ thống gặp lỗi.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện lỗi khi khởi tạo hồ sơ (database, service không khả dụng) |
| 2b | System | Hiển thị thông báo lỗi "Không thể tạo hồ sơ tài trợ. Vui lòng thử lại sau." |
| 2c | Organizer | Có thể thử tạo lại hoặc quay về dashboard |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-01-01 | Tạo hồ sơ thành công | Organizer khởi tạo hồ sơ tài trợ mới; hồ sơ được tạo ở trạng thái DRAFT |
| SC-01-02 | Tạo hồ sơ thất bại | Quá trình tạo hồ sơ gặp lỗi; organizer được thông báo và có thể thử lại |

---

### 5. Preconditions

#### 5.1 Organizer đã xác thực

- Organizer đã đăng nhập vào hệ thống với vai trò `organizer`

#### 5.2 Tổ chức BTC hợp lệ

- Organizer là tài khoản đại diện duy nhất của một tổ chức BTC hợp lệ trên hệ thống

---

### 6. Postconditions

#### 6.1 Success

- Hồ sơ tài trợ mới được tạo với trạng thái DRAFT
- Hồ sơ có proposal_id duy nhất và được gắn với tổ chức BTC
- Thời gian tạo và danh tính người tạo được ghi nhận
- Organizer đang ở trang chỉnh sửa hồ sơ, sẵn sàng soạn thảo nội dung

#### 6.2 Failure

- Không có hồ sơ nào được tạo
- Organizer được thông báo về lỗi và ở lại trang trước đó

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0101: Mỗi hồ sơ tài trợ khi khởi tạo PHẢI có trạng thái ban đầu là DRAFT
- BR-0102: Mỗi hồ sơ tài trợ PHẢI được gán cho đúng một tổ chức BTC. Một BTC có thể có nhiều hồ sơ tài trợ đồng thời

---

### 10. Additional Information

**Assumptions:**

- Hồ sơ được tạo ở trạng thái DRAFT — chưa có nội dung chi tiết tại bước này (cho phép tạo bản nháp trống)
- Organizer tiếp tục soạn thảo nội dung thông qua UC-02 (Chỉnh sửa nội dung hồ sơ tài trợ) và UC-03 (Quản lý gói tài trợ)

**Related Use Cases:**

- UC-02: Chỉnh sửa nội dung hồ sơ tài trợ (sequential — bước tiếp theo)
- UC-03: Quản lý gói tài trợ (sequential — bước tiếp theo)
- UC-04: Phát hành hồ sơ tài trợ (sequential — sau khi nội dung đầy đủ)
