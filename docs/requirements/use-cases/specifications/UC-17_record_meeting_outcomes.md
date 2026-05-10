# Use-Case Specification: UC-17 — Ghi nhận kết quả cuộc họp

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người ghi nhận kết quả |
| Secondary | System | Lưu trữ, hiển thị cho cả hai bên |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) ghi nhận kết quả sau cuộc họp thương thảo vào notebook chung của deal, bao gồm tóm tắt nội dung, các quyết định đã thống nhất, và action items tiếp theo. Notebook được lưu trong deal context để tham khảo khi soạn hợp đồng; hệ thống không tạo nội dung tự động từ video meeting.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Ghi nhận kết quả" sau cuộc họp hoặc trong trang chi tiết meeting.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Mở trang chi tiết cuộc họp đã CONFIRMED |
| 2 | Authenticated User | Nhấn "Ghi nhận kết quả" |
| 3 | System | Hiển thị form ghi nhận: tóm tắt nội dung, quyết định đã thống nhất, action items |
| 4 | Authenticated User | Nhập tóm tắt nội dung cuộc họp |
| 5 | Authenticated User | Nhập danh sách quyết định đã thống nhất |
| 6 | Authenticated User | Nhập danh sách action items tiếp theo |
| 7 | Authenticated User | Nhấn "Lưu" |
| 8 | System | Lưu ghi chú cuộc họp, ghi nhận noted_by và noted_at |
| 9 | System | Hiển thị ghi chú cho cả hai bên trong deal context |
| 10 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-17.a: Chỉnh sửa ghi chú đã lưu
>
> *Triggered at Step 2 of the Basic Flow when cuộc họp đã có ghi chú trước đó.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện cuộc họp đã có ghi chú trước đó |
| 2b | System | Hiển thị ghi chú hiện tại với khả năng chỉnh sửa |
| 2c | Authenticated User | Cập nhật nội dung ghi chú |
| 2d | System | Lưu thay đổi. Use case kết thúc |

#### 2.3 Exception Flows

Không có exception flow đặc biệt cho use case này.

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-17-01 | Ghi nhận mới | Actor ghi nhận tóm tắt, quyết định, action items sau cuộc họp CONFIRMED |
| SC-17-02 | Chỉnh sửa ghi nhận | Actor cập nhật ghi chú cuộc họp đã có (AF-17.a) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Cuộc họp đã CONFIRMED

- Cuộc họp đã ở trạng thái CONFIRMED (đã diễn ra hoặc đang diễn ra)

#### 5.3 Actor là bên liên quan

- Actor là một trong hai bên liên quan trong deal

---

### 6. Postconditions

#### 6.1 Success

- Ghi chú cuộc họp được lưu và hiển thị cho cả hai bên
- Thông tin này có thể tham khảo khi soạn hợp đồng (SF-05)

#### 6.2 Failure

- Ghi chú không được lưu

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- Không có business rule riêng cho use case này

---

### 10. Additional Information

**Assumptions:**

- Cả hai bên đều có thể ghi nhận kết quả, nhưng mỗi meeting chỉ có MỘT notebook chung (bên ghi sau sẽ chỉnh sửa bản đã có)
- Notebook cuộc họp là nguồn tham khảo quan trọng cho giai đoạn soạn thảo hợp đồng
- Hệ thống không tạo nội dung tự động từ video meeting; actor tự nhập thông tin

**Related Use Cases:**

- UC-15: Đặt lịch họp thương thảo (prerequisite — meeting phải tồn tại)
- UC-16: Phản hồi đề xuất lịch họp (`<<extend>>` base — UC-17 mở rộng UC-16)
- UC-20: Chỉnh sửa điều khoản hợp đồng (sequential — tham khảo kết quả họp)
