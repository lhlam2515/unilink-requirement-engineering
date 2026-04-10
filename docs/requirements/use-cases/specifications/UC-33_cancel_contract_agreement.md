# UC-33: ~~Hủy đồng thuận ký kết hợp đồng~~ [DEPRECATED — BP03]

> **⚠️ USE CASE NÀY ĐÃ BỊ LOẠI BỎ**
>
> Kể từ BP03, cơ chế "hủy đồng thuận ký kết" đã được thay thế bằng cơ chế **hard-lock** (SF-05 FR-0507, BR-0509).
>
> Sau khi 2/2 thanh toán phí dịch vụ hoàn tất (PaywallSession.status = COMPLETED):
> - Hợp đồng vào giai đoạn hard-lock
> - Mọi hành động hủy hợp đồng/hủy đồng thuận bị vô hiệu hóa
> - Hai bên có 72 giờ để hoàn tất ký chữ ký điện tử
> - Nếu quá hạn: UC-49 (Xử lý vi phạm ký kết) được kích hoạt
>
> **Thay thế bởi:**
> - SF-05 FR-0507: Khóa cứng giai đoạn ký kết hợp đồng
> - UC-49: Xử lý vi phạm ký kết hợp đồng
>
> **Lý do loại bỏ:**
> Sau khi cả hai bên đã thanh toán phí dịch vụ (non-refundable), việc cho phép hủy đồng thuận sẽ gây thiệt hại tài chính. Hard-lock đảm bảo cam kết nghiêm túc sau khi đã đầu tư tài chính.
