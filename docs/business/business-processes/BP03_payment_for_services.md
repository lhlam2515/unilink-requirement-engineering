# QUY TRÌNH THANH TOÁN PHÍ DỊCH VỤ VÀ KẾT NỐI TÀI TRỢ SỰ KIỆN

Nền tảng hoạt động dưới mô hình Công cụ kết nối công nghệ (Tech-Matchmaker). Nền tảng **không** can thiệp vào dòng tiền tài trợ chính, **không** định giá hiện vật và **không** chịu trách nhiệm pháp lý về quá trình thực thi sự kiện hay vận chuyển (Logistics).

Nền tảng cung cấp môi trường thương thảo bảo mật, công cụ Hợp đồng điện tử (E-contract) và thu **Phí dịch vụ kết nối**. Quy trình này giúp Doanh nghiệp và Câu lạc bộ (CLB) tiếp cận nhau minh bạch, đồng thời giải phóng Nền tảng khỏi các rủi ro về kế toán thuế và quản lý quỹ trung gian.

## 1. Định nghĩa thuật ngữ và Cơ cấu thu phí

Nền tảng áp dụng cơ chế **Phí chia sẻ (Split Fee)** để tạo bộ lọc cam kết (Commitment Filter) từ cả hai phía.

* **Bắt tay kép (Double Handshake):** Cơ chế thanh toán yêu cầu cả Doanh nghiệp và CLB phải hoàn tất việc nộp phí dịch vụ trong vòng **48 giờ** để mở khóa thông tin liên hệ và kích hoạt hợp đồng.
* **Che giấu dữ liệu (Data Masking):** Thuật toán tự động ẩn các thông tin liên hệ định danh (Số điện thoại, Email, Link Mạng xã hội) trong khung chat cho đến khi việc thanh toán phí hoàn tất.
* **Hóa đơn Giá trị gia tăng (VAT Invoice):** Nền tảng **chỉ** xuất hóa đơn VAT điện tử cho khoản *Phí quản lý chiến dịch* thu từ Doanh nghiệp, tuyệt đối không xuất hóa đơn cho giá trị gói tài trợ.

**Bảng Cơ cấu Thu phí:**

| Đối tượng | Loại phí | Tài trợ Tiền mặt | Tài trợ Hiện vật |
| :--- | :--- | :--- | :--- |
| **CLB Sinh viên** | Phí duy trì nền tảng | **1%** giá trị hợp đồng (Tối đa: **100.000 VNĐ**) | Cố định **50.000 VNĐ** |
| **Doanh nghiệp** | Phí quản lý chiến dịch | Tính theo **%** giá trị hợp đồng (Có mức trần tối đa) | Cố định **500.000 VNĐ** |

---

## 2. Quy trình Thực hiện Giao dịch

### Bước 1: Thương thảo và Khóa thỏa thuận (Lock-in)

* **Chủ thể tham gia:** Doanh nghiệp, CLB, Nền tảng (Hệ thống tự động).
* **Hành động trên Nền tảng:**
  * Hai bên trao đổi chi tiết về quyền lợi, ngân sách, hoặc số lượng hiện vật qua hệ thống Chat (đã được áp dụng *Data Masking*).
  * Đối với hiện vật, hai bên phải tích chọn xác nhận miễn trừ trách nhiệm chất lượng và vận chuyển cho Nền tảng.
  * Khi đạt thỏa thuận, một bên tạo "Thỏa thuận nháp", bên còn lại bấm "Xác nhận". Trạng thái thương vụ chuyển sang `Chờ thanh toán phí dịch vụ`.

### Bước 2: Bức tường thu phí (The Paywall)

* **Chủ thể tham gia:** Doanh nghiệp, CLB.
* **Hành động trên Nền tảng:**
  * Hệ thống hiển thị Bức tường thu phí (Paywall) độc lập cho từng bên dựa trên công thức hoặc mức phí cố định đã thiết lập.
  * Doanh nghiệp được yêu cầu nhập thông tin Mã số thuế để hệ thống chuẩn bị xuất hóa đơn VAT.
* **Hành động ngoài Nền tảng:** Hai bên có **48 giờ** để quét mã QR (VietQR/MoMo/ZaloPay) nhằm thanh toán khoản phí dịch vụ này cho Nền tảng một cách độc lập. Hệ thống sẽ hiển thị trạng thái thanh toán theo thời gian thực (Ví dụ: `1/2 Đã thanh toán`).

### Bước 3: Mở khóa thông tin và Ký Hợp đồng (Unlocking)

* **Chủ thể tham gia:** Doanh nghiệp, CLB, Nền tảng.
* **Hành động trên Nền tảng:**
  * Ngay khi hệ thống ghi nhận trạng thái **2/2 hoàn tất thanh toán**, thuật toán *Data Masking* được gỡ bỏ, hiển thị công khai thông tin liên hệ thật của hai bên.
  * Hệ thống tự động sinh ra **Hợp đồng điện tử (E-contract)**. Hai bên tiến hành ký chữ ký số.
  * Hệ thống tự động kết nối API (hoặc Kế toán thực hiện thao tác) xuất **Hóa đơn điện tử (VAT)** cho phần *Phí quản lý chiến dịch* và gửi về email cho Doanh nghiệp.

### Bước 4: Thực hiện tài trợ (Ngoài nền tảng)

* **Chủ thể tham gia:** Doanh nghiệp, CLB.
* **Hành động ngoài Nền tảng:** Doanh nghiệp tự thực hiện chuyển khoản số tiền tài trợ hoặc tự vận chuyển bàn giao hiện vật **trực tiếp** cho CLB theo đúng các điều khoản trong E-contract. Hai bên tự ký Biên bản bàn giao (nếu là hiện vật). Nền tảng hoàn toàn không can thiệp vào luồng thực thi này.

### Bước 5: Nghiệm thu và Đóng dự án

* **Chủ thể tham gia:** Doanh nghiệp, CLB.
* **Hành động trên Nền tảng:** Sau sự kiện, CLB tải lên các báo cáo minh chứng (Proof of Performance). Doanh nghiệp kiểm tra và bấm "Xác nhận hoàn thành" để đánh giá mức độ uy tín (Rating) và đóng dự án trên hệ thống.

---

## 3. Các Quy tắc Xử lý Rủi ro Thanh toán

Để đảm bảo tính công bằng và bảo vệ luồng doanh thu của nền tảng, hệ thống áp dụng 3 quy tắc "cứng" sau:

1. **Hoàn tiền tự động (Auto-Refund):** Nếu một bên đã nộp phí dịch vụ, nhưng bên còn lại không thực hiện thanh toán khi hết thời hạn **48 giờ**. Hệ thống sẽ tự động hủy thương vụ và hoàn tiền 100% về tài khoản gốc của bên đã thanh toán.
2. **Không hoàn phí (Non-refundable):** Ngay khi hệ thống mở khóa thông tin liên hệ và cấp Hợp đồng điện tử (trạng thái 2/2 thanh toán), khoản phí dịch vụ của Nền tảng sẽ **không được hoàn lại** dưới bất kỳ hình thức nào, kể cả khi Doanh nghiệp và CLB phát sinh tranh chấp hoặc hủy bỏ tài trợ sau đó.
3. **Cấm lách luật (Zero Tolerance for Bypassing):** Bất kỳ tài khoản nào cố tình dùng từ lóng, ký tự đặc biệt để lách bộ lọc nội dung nhằm trao đổi số điện thoại/Zalo trước khi qua Bức tường thu phí sẽ bị khóa vĩnh viễn.
