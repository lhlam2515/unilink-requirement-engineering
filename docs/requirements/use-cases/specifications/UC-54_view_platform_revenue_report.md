# Use-Case Specification: UC-54 — Xem báo cáo doanh thu nền tảng

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Xem và lọc báo cáo doanh thu |
| Secondary | System | Tổng hợp dữ liệu từ PlatformRevenueLog |

---

### 1. Brief Description

> Admin xem báo cáo tổng hợp doanh thu phí dịch vụ của nền tảng theo khoảng thời gian. Báo cáo bao gồm tổng thu, tổng hoàn tiền, doanh thu ròng, và danh sách chi tiết các giao dịch phí.

---

### 2. Flow of Events

**Trigger**
> Admin truy cập trang "Báo cáo doanh thu nền tảng".

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Admin | Truy cập trang "Báo cáo doanh thu nền tảng" |
| 2 | System | Hiển thị báo cáo mặc định cho tháng hiện tại |
| 3 | System | Hiển thị tổng quan: Tổng thu (INCOME), Tổng hoàn tiền (REFUND), Doanh thu ròng |
| 4 | System | Hiển thị danh sách giao dịch: deal ID, bên nộp, số tiền, loại, ngày |
| 5 | Admin | Lọc theo: khoảng thời gian, loại giao dịch, vai trò bên nộp |
| 6 | System | Cập nhật báo cáo theo bộ lọc |
| 7 | System | Use case kết thúc thành công |

#### 2.2 Alternate Flows

##### AF-54.a: Không có giao dịch trong khoảng thời gian
>
> *Triggered at Step 2 of the Basic Flow when không có dữ liệu.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Hiển thị empty state, tổng quan = 0 |

#### 2.3 Exception Flows

Không có exception flow đặc biệt cho use case này.

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-54-01 | Xem doanh thu tháng | Admin xem tổng quan và chi tiết giao dịch tháng hiện tại |
| SC-54-02 | Lọc theo khoảng thời gian | Admin lọc báo cáo theo tùy chọn |

---

### 5. Preconditions

#### 5.1 Admin đã xác thực

- Admin đã đăng nhập với quyền quản trị

---

### 6. Postconditions

#### 6.1 Success

- Admin xem được báo cáo doanh thu chính xác
- Không có thay đổi dữ liệu — chỉ hiển thị

#### 6.2 Failure

- Không áp dụng (use case chỉ đọc dữ liệu)

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-1404: Mọi ServiceFeeTransaction chuyển trạng thái PHẢI được ghi vào PlatformRevenueLog
- Doanh thu ròng = Σ INCOME − Σ REFUND

---

### 10. Additional Information

**Assumptions:**

- Báo cáo mang tính tham chiếu, không thay thế phần mềm kế toán
- Không hỗ trợ xuất file (CSV/Excel) ở phiên bản đầu

**Related Use Cases:**

- UC-50: Thanh toán phí dịch vụ (sequential — ghi revenue log)
- UC-55: Đối soát thủ công (sequential — ảnh hưởng doanh thu)
