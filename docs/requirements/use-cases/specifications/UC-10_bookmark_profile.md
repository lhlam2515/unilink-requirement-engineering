# Use-Case Specification: UC-10 — Lưu hồ sơ quan tâm

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Sponsor hoặc Organizer) | Người lưu/bỏ lưu bookmark |
| Secondary | System | Quản lý bookmark, kiểm tra trùng lặp |

---

### 1. Brief Description

> Authenticated User (Sponsor hoặc Organizer) lưu bookmark hồ sơ tài trợ sự kiện hoặc hồ sơ doanh nghiệp vào danh sách quan tâm để tham khảo sau. Có thể xem danh sách đã bookmark và xóa bookmark khi không còn quan tâm.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn nút "Bookmark" / "Lưu" trên trang chi tiết hoặc kết quả tìm kiếm.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn nút "Bookmark" trên hồ sơ quan tâm |
| 2 | System | Kiểm tra actor chưa bookmark hồ sơ này trước đó |
| 3 | System | Tạo bookmark mới với target_type (PROPOSAL hoặc BUSINESS) và target_id |
| 4 | System | Ghi nhận thời gian bookmark (bookmarked_at) |
| 5 | System | Cập nhật giao diện: hiển thị trạng thái "Đã lưu" |
| 6 | System | Use case kết thúc thành công — hồ sơ đã được lưu vào danh sách quan tâm |

#### 2.2 Alternate Flows

##### AF-10.a: Actor xem danh sách bookmark
>
> *Triggered at Step 1 of the Basic Flow when actor truy cập danh sách quan tâm.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Truy cập trang "Danh sách quan tâm" trong dashboard |
| 1b | System | Hiển thị danh sách hồ sơ đã bookmark với thông tin tóm tắt |
| 1c | System | Đánh dấu các hồ sơ đã bị hủy phát hành với trạng thái "Hồ sơ không còn khả dụng" |
| 1d | System | Use case kết thúc |

##### AF-10.b: Actor bỏ bookmark
>
> *Triggered at Step 1 of the Basic Flow when actor chọn bỏ lưu.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Nhấn "Bỏ lưu" trên hồ sơ đã bookmark |
| 1b | System | Xóa bookmark khỏi danh sách |
| 1c | System | Cập nhật giao diện: hiển thị nút "Bookmark" thay vì "Đã lưu" |
| 1d | System | Use case kết thúc |

#### 2.3 Exception Flows

##### EF-10.1: Hồ sơ đã được bookmark trước đó
>
> *Triggered at Step 2 of the Basic Flow when actor đã bookmark hồ sơ này.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện actor đã bookmark hồ sơ này |
| 2b | System | Không tạo bookmark trùng lặp, giữ nguyên trạng thái "Đã lưu" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-10-01 | Bookmark thành công | Actor lưu hồ sơ vào danh sách quan tâm |
| SC-10-02 | Xem danh sách bookmark | Actor xem danh sách hồ sơ đã lưu, bao gồm cả hồ sơ "không khả dụng" (AF-10.a) |
| SC-10-03 | Bỏ bookmark | Actor xóa hồ sơ khỏi danh sách quan tâm (AF-10.b) |
| SC-10-04 | Bookmark trùng lặp | Actor bookmark hồ sơ đã lưu trước đó; không tạo bản trùng (EF-10.1) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hồ sơ mục tiêu khả dụng

- Hồ sơ mục tiêu đang ở trạng thái PUBLISHED (sự kiện) hoặc ACTIVE (doanh nghiệp)

---

### 6. Postconditions

#### 6.1 Success

- Bookmark được tạo/xóa thành công
- Danh sách quan tâm được cập nhật

#### 6.2 Failure

- Không có thay đổi nào đối với bookmark

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0205: Mỗi actor chỉ có thể bookmark một hồ sơ MỘT LẦN (không trùng lặp). Bookmark hồ sơ bị hủy phát hành được giữ lại nhưng đánh dấu "không khả dụng"

---

### 10. Additional Information

**Assumptions:**

- Sponsor bookmark hồ sơ tài trợ sự kiện, Organizer bookmark hồ sơ doanh nghiệp
- Bookmark hồ sơ bị hủy phát hành không bị xóa tự động — chỉ đánh dấu trạng thái

**Related Use Cases:**

- UC-08: Xem chi tiết hồ sơ tài trợ sự kiện (`<<extend>>` base — UC-10 mở rộng UC-08)
- UC-09: Xem chi tiết hồ sơ doanh nghiệp (`<<extend>>` base — UC-10 mở rộng UC-09)
- UC-06: Tìm kiếm sự kiện để tài trợ (sequential — bookmark từ kết quả tìm kiếm)
- UC-07: Tìm kiếm doanh nghiệp để mời tài trợ (sequential — bookmark từ kết quả tìm kiếm)
