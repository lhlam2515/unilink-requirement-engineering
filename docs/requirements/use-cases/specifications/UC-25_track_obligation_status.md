# Use-Case Specification: UC-25 — Theo dõi trạng thái nghĩa vụ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người theo dõi nghĩa vụ |
| Secondary | System | Tạo nghĩa vụ tự động, theo dõi deadline, gửi nhắc nhở |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) xem dashboard nghĩa vụ tài trợ cho một hợp đồng đã ký, bao gồm danh sách nghĩa vụ của mình và đối tác, trạng thái từng nghĩa vụ, deadline, và tiến trình tổng thể. Danh sách nghĩa vụ được tạo tự động khi hợp đồng ký kết.

---

### 2. Flow of Events

**Trigger**
> Actor truy cập trang "Nghĩa vụ tài trợ" của hợp đồng.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Truy cập trang "Nghĩa vụ tài trợ" của hợp đồng |
| 2 | System | Hiển thị dashboard nghĩa vụ với tiến trình tổng thể (tổng/hoàn thành/đang chờ/quá hạn) |
| 3 | System | Hiển thị danh sách nghĩa vụ của đối tác với trạng thái: PENDING, IN_PROGRESS, SUBMITTED, CONFIRMED, DISPUTED, OVERDUE |
| 4 | System | Hiển thị danh sách nghĩa vụ của bản thân với trạng thái tương ứng |
| 5 | System | Đánh dấu nghĩa vụ quá hạn bằng nhãn "QUÁ HẠN" (màu đỏ) |
| 6 | System | Hiển thị deadline cho từng nghĩa vụ |
| 7 | System | Use case kết thúc thành công — actor thấy toàn bộ trạng thái nghĩa vụ |

#### 2.2 Alternate Flows

##### AF-25.a: Actor xem chi tiết nghĩa vụ cụ thể
>
> *Triggered at Step 3 hoặc 4 of the Basic Flow when actor chọn một nghĩa vụ.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Authenticated User | Nhấn vào một nghĩa vụ cụ thể |
| 3b | System | Hiển thị chi tiết: mô tả, deadline, trạng thái, bằng chứng đã nộp (nếu có), lịch sử thay đổi |

#### 2.3 Exception Flows

##### EF-25.1: Chưa có nghĩa vụ nào
>
> *Triggered at Step 2 of the Basic Flow when hợp đồng chưa có nghĩa vụ.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện hợp đồng chưa có nghĩa vụ (lỗi tạo tự động) |
| 2b | System | Hiển thị thông báo "Chưa có nghĩa vụ nào cho hợp đồng này" |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-25-01 | Xem dashboard nghĩa vụ | Actor xem tiến trình tổng thể và trạng thái từng nghĩa vụ |
| SC-25-02 | Xem chi tiết nghĩa vụ | Actor xem chi tiết bao gồm bằng chứng và lịch sử (AF-25.a) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hợp đồng đã SIGNED

- Hợp đồng đang ở trạng thái SIGNED

#### 5.3 Nghĩa vụ đã được tạo

- Danh sách nghĩa vụ đã được tạo tự động (từ FR-0601)

#### 5.4 Actor là bên liên quan

- Actor là một trong hai bên liên quan

---

### 6. Postconditions

#### 6.1 Success

- Actor xem được toàn bộ nghĩa vụ và tiến trình

#### 6.2 Failure

- Không có dữ liệu hiển thị

---

### 7. Extension Points

| # | Extension Point | Extending UC | Điều kiện |
|---|----------------|--------------|-----------|
| 1 | Khi actor xem chi tiết một nghĩa vụ PENDING/IN_PROGRESS/DISPUTED | UC-26: Nộp bằng chứng hoàn thành nghĩa vụ | Actor là bên chịu trách nhiệm và muốn báo cáo hoàn thành |
| 2 | Khi actor xem chi tiết một nghĩa vụ SUBMITTED | UC-27: Xác nhận hoàn thành nghĩa vụ | Actor là bên đối tác và muốn xác nhận/từ chối |

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0601: Danh sách nghĩa vụ PHẢI được tạo tự động khi hợp đồng SIGNED. Nghĩa vụ doanh nghiệp: chuyển khoản/bàn giao hiện vật. Nghĩa vụ BTC: quảng bá, truyền thông, tổ chức, báo cáo
- BR-0605: Nghĩa vụ PENDING hoặc IN_PROGRESS quá deadline tự động chuyển sang OVERDUE. Hệ thống gửi nhắc nhở T-3 ngày, T-0, T+1 ngày qua in-app và email

---

### 10. Additional Information

**Assumptions:**

- Nghĩa vụ quá hạn được chuyển trạng thái tự động bởi scheduled job hàng ngày
- Nhắc nhở được gửi tự động: 3 ngày trước hạn, ngày hạn, 1 ngày sau hạn
- Actor có thể chuyển sang UC-26 để nộp bằng chứng hoặc UC-27 để xác nhận hoàn thành

**Related Use Cases:**

- UC-22: Ký hợp đồng điện tử (prerequisite — nghĩa vụ tạo tự động khi SIGNED)
- UC-26: Nộp bằng chứng hoàn thành nghĩa vụ (`<<extend>>` — nộp từ trang chi tiết)
- UC-27: Xác nhận hoàn thành nghĩa vụ (`<<extend>>` — xác nhận từ trang chi tiết)
- UC-28: Nộp báo cáo kết quả sự kiện (sequential — nghĩa vụ báo cáo của BTC)
