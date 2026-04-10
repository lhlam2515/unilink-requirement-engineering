# SCR-029: Admin_PlatformRevenueReport_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-029 |
| **Screen Name** | Admin_PlatformRevenueReport_Screen |
| **Mục đích** | Admin xem báo cáo tổng hợp doanh thu phí dịch vụ của nền tảng theo khoảng thời gian: tổng thu, tổng hoàn tiền, doanh thu ròng, và danh sách giao dịch chi tiết |
| **Actor chính** | Admin |
| **Quy trình nghiệp vụ** | BP-03 / Quản trị doanh thu (SF-14) |
| **Use case liên quan** | UC-54 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Báo cáo doanh thu — reporting context |
| Data scope riêng | ✅ PlatformRevenueLog, aggregate data |
| Action set riêng | ✅ Lọc thời gian, lọc loại giao dịch |
| Navigation boundary | ✅ Admin panel → revenue report |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

### KPI Summary Cards

- Tổng thu (INCOME)
- Tổng hoàn tiền (REFUND)
- Doanh thu ròng (= Σ INCOME − Σ REFUND)

### Danh sách giao dịch chi tiết

Mỗi giao dịch hiển thị:

- Deal ID
- Bên nộp (payer): Organizer / Sponsor
- Số tiền (amount)
- Loại giao dịch: INCOME / REFUND
- Ngày giao dịch (recorded_at)

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Khoảng thời gian | Date range picker | Mặc định: tháng hiện tại |
| Loại giao dịch | Select filter | ALL / INCOME / REFUND |
| Vai trò bên nộp | Select filter | ALL / Organizer / Sponsor |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Lọc báo cáo** | Cập nhật dữ liệu và KPI theo bộ lọc |

## Hành động phụ (Secondary Actions)

Không có hành động phụ ở phiên bản đầu.

> **Ghi chú:** Xuất file CSV/Excel có thể bổ sung trong phiên bản sau.

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-1404: Mọi ServiceFeeTransaction chuyển trạng thái PHẢI được ghi vào PlatformRevenueLog
- Doanh thu ròng = Σ INCOME − Σ REFUND

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Admin Menu | Menu "Báo cáo doanh thu nền tảng" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Quay lại / Breadcrumb | Admin Dashboard / Menu |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải dữ liệu báo cáo |
| Empty State | Chưa có giao dịch trong khoảng thời gian đã chọn (UC-54 AF-54.a) |
| Data Loaded | Hiển thị KPI + danh sách giao dịch |
| Filter Applied | Bộ lọc đang active — hiển thị indicator |

## UI Components liên quan

- **KPI summary cards** — tổng thu, tổng hoàn tiền, doanh thu ròng
- **Data table** — danh sách giao dịch chi tiết với pagination
- **Date range picker** — chọn khoảng thời gian
- **Filter bar** — loại giao dịch, vai trò bên nộp
- **Empty state illustration** — không có dữ liệu

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-54 | Main-1 | Truy cập trang báo cáo doanh thu | Screen entry | Page load |
| UC-54 | Main-2 | Hiển thị báo cáo mặc định (tháng hiện tại) | Read-only data | KPI cards + data table |
| UC-54 | Main-3 | Hiển thị tổng quan: tổng thu, hoàn tiền, ròng | Read-only data | KPI summary cards |
| UC-54 | Main-4 | Hiển thị danh sách giao dịch | Read-only data | Data table |
| UC-54 | Main-5~6 | Lọc theo bộ lọc | Input + Action | Filter bar + date range |
| UC-54 | AF-54.a | Không có giao dịch | UI State | Empty State |
