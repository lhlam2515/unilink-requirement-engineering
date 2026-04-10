# UC-54: Xem báo cáo doanh thu nền tảng

**Brief Description**
> Admin xem báo cáo tổng hợp doanh thu phí dịch vụ của nền tảng theo khoảng thời gian. Báo cáo bao gồm tổng thu, tổng hoàn tiền, doanh thu ròng, và danh sách chi tiết các giao dịch phí.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Admin | Xem và lọc báo cáo doanh thu |
| Secondary | System | Tổng hợp dữ liệu từ PlatformRevenueLog |

---

**Preconditions**

- Admin đã đăng nhập với quyền quản trị
- Có ít nhất một PlatformRevenueLog trong hệ thống

---

**Trigger**
> Admin truy cập trang "Báo cáo doanh thu nền tảng".

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Admin | Truy cập trang "Báo cáo doanh thu nền tảng" |
| 2 | System | Hiển thị báo cáo mặc định cho tháng hiện tại |
| 3 | System | Hiển thị tổng quan: Tổng thu (INCOME), Tổng hoàn tiền (REFUND), Doanh thu ròng |
| 4 | System | Hiển thị danh sách giao dịch chi tiết: deal ID, bên nộp, số tiền, loại (INCOME/REFUND), ngày |
| 5 | Admin | Có thể lọc theo: khoảng thời gian, loại giao dịch, vai trò bên nộp (Organizer/Sponsor) |
| 6 | System | Cập nhật báo cáo theo bộ lọc |
| 7 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-54.a: Không có giao dịch trong khoảng thời gian (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện không có giao dịch trong khoảng thời gian đã chọn |
| 2b | System | Hiển thị empty state: "Chưa có giao dịch phí dịch vụ trong khoảng thời gian này" |
| 2c | System | Hiển thị tổng quan = 0 |

---

**Exception Flows**

Không có exception flow đặc biệt cho use case này.

---

**Postconditions**

*Success:*
- Admin nhìn thấy báo cáo doanh thu với dữ liệu chính xác
- Không có thay đổi dữ liệu — chỉ hiển thị thông tin

---

**Business Rules**

- BR-1404: Mọi ServiceFeeTransaction chuyển trạng thái PHẢI được ghi vào PlatformRevenueLog
- Doanh thu ròng = Σ INCOME − Σ REFUND

---

**Notes / Assumptions**

- Báo cáo mang tính tham chiếu, không thay thế phần mềm kế toán chính thức
- Không hỗ trợ xuất file (CSV/Excel) ở phiên bản đầu — có thể bổ sung
- Liên kết: UC-50 (thanh toán phí → ghi revenue log), UC-55 (đối soát thủ công)
