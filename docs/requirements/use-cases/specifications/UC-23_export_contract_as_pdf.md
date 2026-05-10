# Use-Case Specification: UC-23 — Xuất hợp đồng dạng PDF

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người yêu cầu xuất PDF |
| Secondary | System | Tạo file PDF, đóng dấu thời gian |

---

### 1. Brief Description

> Authenticated User (Organizer hoặc Sponsor) xuất hợp đồng đã ký dưới dạng file PDF để lưu trữ. PDF bao gồm toàn bộ nội dung hợp đồng, chữ ký điện tử của hai bên, ngày ký, mã hợp đồng, và dấu thời gian xuất.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Xuất PDF" trên trang hợp đồng đã ký.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Authenticated User | Nhấn "Xuất PDF" trên trang hợp đồng đã ký |
| 2 | System | Tạo file PDF bao gồm: toàn bộ nội dung hợp đồng, chữ ký điện tử của hai bên, ngày ký |
| 3 | System | Thêm mã hợp đồng (contract_number) và dấu thời gian xuất |
| 4 | System | Thêm watermark "BẢN GỐC ĐIỆN TỬ" |
| 5 | System | Cung cấp file PDF để actor tải về với tên "[contract_number]_signed.pdf" |
| 6 | System | Use case kết thúc thành công — actor tải PDF |

#### 2.2 Alternate Flows

Không có alternate flow cho use case này.

#### 2.3 Exception Flows

##### EF-23.1: Hợp đồng chưa ký
>
> *Triggered at Step 1 of the Basic Flow when hợp đồng chưa ở trạng thái SIGNED.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hợp đồng ở trạng thái DRAFTING hoặc CONFIRMED |
| 1b | System | Từ chối "Chỉ có thể xuất PDF cho hợp đồng đã ký" |

##### EF-23.2: Lỗi tạo PDF
>
> *Triggered at Step 2 of the Basic Flow when lỗi kỹ thuật.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Gặp lỗi khi tạo file PDF |
| 2b | System | Hiển thị "Không thể tạo PDF. Vui lòng thử lại sau." |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-23-01 | Xuất PDF thành công | Actor tải về file PDF hợp đồng đầy đủ nội dung và chữ ký |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Hợp đồng đã SIGNED

- Hợp đồng đang ở trạng thái SIGNED

#### 5.3 Actor là bên liên quan

- Actor là một trong hai bên liên quan

---

### 6. Postconditions

#### 6.1 Success

- Actor tải được file PDF hợp đồng đầy đủ nội dung và chữ ký

#### 6.2 Failure

- Không có file PDF được tạo
- Actor được thông báo lỗi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0507: Chỉ hợp đồng SIGNED mới xuất PDF. PDF là tài liệu chỉ đọc với watermark "BẢN GỐC ĐIỆN TỬ"

---

### 10. Additional Information

**Assumptions:**

- PDF có thể xuất nhiều lần
- Cả hai bên đều có quyền xuất PDF

**Related Use Cases:**

- UC-22: Ký hợp đồng điện tử (`<<extend>>` base — UC-23 mở rộng UC-22 sau khi SIGNED)
