# 📝 Ghi chú Lịch sử Thay đổi: Tích hợp Quy trình Thanh toán (BP03)

**Ngày ghi nhận:** 10/04/2026
**Sự kiện gốc:** Bổ sung quy trình `BP03_payment_for_services` (Commit `163ebd4`)
**Mục tiêu:** Chuyển đổi luồng vận hành từ *Thương thảo $\rightarrow$ Ký kết* sang *Thương thảo $\rightarrow$ Thanh toán phí $\rightarrow$ Mở khóa thông tin $\rightarrow$ Ký kết*.

---

## 🚀 1. Các Use-case (UC) thay đổi

### ➕ Bổ sung mới

Các use-case này được xây dựng để hiện thực hóa cơ chế thu phí và đối soát:

| Mã UC | Tên Use-case | Mục đích |
| :--- | :--- | :--- |
| **UC-50** | Pay service fee | Thực hiện thanh toán phí dịch vụ qua cổng QR/Ví điện tử. |
| **UC-51** | Preview service fee | Xem chi tiết phí dịch vụ dự kiến trước khi thanh toán. |
| **UC-52** | Issue service fee VAT invoice | Tự động phát hành hóa đơn VAT cho phí quản lý chiến dịch. |
| **UC-24** | Request VAT invoice | Cho phép doanh nghiệp yêu cầu xuất hóa đơn VAT. |
| **UC-53** | Review bypass violation | Quản trị viên xem xét và xử lý các trường hợp lách luật trao đổi thông tin. |
| **UC-54** | View platform revenue report | Theo dõi báo cáo doanh thu từ phí dịch vụ của nền tảng. |
| **UC-55** | Manual payment reconciliation | Đối soát thủ công các giao dịch thanh toán. |
| **UC-56** | Create draft agreement | Tạo thỏa thuận nháp để khóa nội dung trước khi thanh toán. |
| **UC-49** | Handle contract signing breach | Xử lý các vi phạm trong quá trình ký kết sau thanh toán. |

### 🔄 Cập nhật/Sửa đổi

Điều chỉnh các luồng hiện có để tích hợp điều kiện thanh toán và bảo mật:

* **`UC-18` (Confirm mutual agreement):** Thêm bước tạo thỏa thuận nháp và kiểm tra trạng thái thanh toán trước khi xác nhận cuối cùng.
* **`UC-22` (Sign contract electronically):** Bổ sung điều kiện bắt buộc: Chỉ cho phép ký khi trạng thái thanh toán đạt **2/2 (Double Handshake)**.
* **`UC-14` (Exchange messages in deal):** Tích hợp cơ chế che giấu dữ liệu (**Data Masking**) đối với thông tin định danh.
* **`UC-20`, `UC-21`:** Cập nhật luồng chỉnh sửa và xác nhận nội dung hợp đồng để phù hợp với giai đoạn Lock-in.

---

## ⚙️ 2. Các System Features (SF) thay đổi

### ➕ Bổ sung mới

Thiết lập các module kỹ thuật cốt lõi để vận hành BP03:

* **`SF-12` (Service Fee Paywall):** Hệ thống tính toán phí và chặn quyền truy cập thông tin liên hệ cho đến khi hoàn tất thanh toán.
* **`SF-13` (Data Masking Unlocking):** Thuật toán tự động ẩn/hiện thông tin định danh (SĐT, Email) dựa trên trạng thái thanh toán.
* **`SF-14` (Payment Risk Management):** Quản trị rủi ro, bao gồm cơ chế **Auto-Refund** (hoàn tiền nếu một bên không thanh toán sau 48h) và chế tài cấm lách luật.

### 🔄 Cập nhật/Sửa đổi

* **`SF-04` (Negotiation Communication):**
  * Áp dụng `SF-13` vào khung chat.
  * Bổ sung tính năng tạo Thỏa thuận nháp.
  * Thay đổi điểm kết thúc thương thảo $\rightarrow$ Chuyển sang `SF-12` (Paywall).
* **`SF-05` (Contract Management):**
  * Điều kiện kích hoạt hợp đồng $\rightarrow$ Phụ thuộc vào trạng thái thanh toán.
  * Tích hợp luồng xuất hóa đơn VAT điện tử.

---

## 📌 Tóm tắt luồng dữ liệu mới

`Thương thảo (SF-04)` $\rightarrow$ `Tạo Thỏa thuận nháp (UC-56)` $\rightarrow$ `Bức tường thu phí (SF-12/UC-50)` $\rightarrow$ `Mở khóa dữ liệu (SF-13)` $\rightarrow$ `Ký Hợp đồng điện tử (UC-22)` $\rightarrow$ `Xuất hóa đơn VAT (UC-52)`.

---

## 📱 3. Các Screen Specifications thay đổi

Để hỗ trợ các thay đổi nghiệp vụ trên, hệ thống đã được bổ sung và cập nhật các màn hình sau:

### ➕ Bổ sung mới (Screens)

| Mã Screen | Tên Screen | Vai trò | Mục đích |
| :--- | :--- | :--- | :--- |
| **SCR-027** | User_ServiceFeePaywall_Screen | User | Thanh toán phí dịch vụ, quét QR và theo dõi trạng thái. |
| **SCR-028** | Admin_BypassViolationList_Screen | Admin | Kiểm duyệt và xử lý các trường hợp vi phạm lách bộ lọc Data Masking. |
| **SCR-029** | Admin_PlatformRevenueReport_Screen | Admin | Xem báo cáo doanh thu phí dịch vụ toàn nền tảng. |
| **SCR-030** | Admin_PaymentReconciliation_Screen | Admin | Đối soát thủ công các giao dịch thanh toán không khớp hoặc lỗi webhook. |

### 🔄 Cập nhật/Sửa đổi (Screens)

* **`SCR-010` (User_DealList_Screen):** Thêm trạng thái `AWAITING_PAYMENT` và huy hiệu trạng thái thanh toán.
* **`SCR-011` (User_DealNegotiation_Screen):** Tích hợp luồng tạo thỏa thuận nháp (UC-56), xem trước phí (UC-51), và cơ chế Data Masking hiển thị tin nhắn.
* **`SCR-012` (User_ContractEdit_Screen):** Cập nhật nguồn điều hướng (chỉ được tạo sau khi 2/2 thanh toán thành công từ SCR-027).
* **`SCR-013` (User_ContractSign_Screen):** Bổ sung đếm ngược 72 giờ ký kết (Hard-lock) và trigger xử lý vi phạm (UC-49).
* **`SCR-014` (User_ContractView_Screen):** Loại bỏ chức năng yêu cầu hóa đơn VAT (UC-24 cũ).

### 🗺️ Bổ sung tài liệu Mapping

* **`BP03_screens.md`:** Tài liệu ánh xạ chi tiết các Use-case của BP03 vào hệ thống màn hình.
* **`BP01_screens.md`:** Cập nhật luồng điều hướng tổng thể của quy trình tài trợ để xen kẽ bước thanh toán BP03.
