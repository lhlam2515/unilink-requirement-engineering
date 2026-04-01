# UC-23: Xuất hợp đồng dạng PDF

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) xuất hợp đồng đã ký dưới dạng file PDF để lưu trữ. PDF bao gồm toàn bộ nội dung hợp đồng, chữ ký điện tử của hai bên, ngày ký, mã hợp đồng, và dấu thời gian xuất.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người yêu cầu xuất PDF |
| Secondary | System | Tạo file PDF, đóng dấu thời gian |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hợp đồng đang ở trạng thái SIGNED
- Actor là một trong hai bên liên quan

---

**Trigger**
> Actor nhấn "Xuất PDF" trên trang hợp đồng đã ký.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Nhấn "Xuất PDF" trên trang hợp đồng đã ký |
| 2 | System | Tạo file PDF bao gồm: toàn bộ nội dung hợp đồng, chữ ký điện tử của hai bên, ngày ký |
| 3 | System | Thêm mã hợp đồng (contract_number) và dấu thời gian xuất |
| 4 | System | Thêm watermark "BẢN GỐC ĐIỆN TỬ" |
| 5 | System | Cung cấp file PDF để actor tải về với tên "[contract_number]_signed.pdf" |
| 6 | System | Use case kết thúc thành công — actor tải PDF |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-23.1: Hợp đồng chưa ký (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hợp đồng ở trạng thái DRAFTING hoặc CONFIRMED |
| 1b | System | Từ chối "Chỉ có thể xuất PDF cho hợp đồng đã ký" |

> EF-23.2: Lỗi tạo PDF (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Gặp lỗi khi tạo file PDF |
| 2b | System | Hiển thị thông báo "Không thể tạo PDF. Vui lòng thử lại sau." |

---

**Postconditions**

*Success:*
- Actor tải được file PDF hợp đồng đầy đủ nội dung và chữ ký

*Failure:*
- Không có file PDF được tạo
- Actor được thông báo lỗi

---

**Business Rules**

- BR-0507: Chỉ hợp đồng SIGNED mới xuất PDF. PDF là tài liệu chỉ đọc với watermark "BẢN GỐC ĐIỆN TỬ"

---

**Notes / Assumptions**

- PDF có thể xuất nhiều lần
- Cả hai bên đều có quyền xuất PDF
- Liên kết: UC-22
