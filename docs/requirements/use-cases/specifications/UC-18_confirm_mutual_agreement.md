# UC-18: Xác nhận đồng thuận ký kết

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) xác nhận sẵn sàng tiến đến ký kết hợp đồng. [UPDATED — BP03] Trước khi đồng thuận, PHẢI có Thỏa thuận nháp đã xác nhận (UC-56). Nếu deal có hiện vật (IN_KIND/COMBINED), bên nhận hiện vật phải tick miễn trừ trách nhiệm. Khi CẢ HAI bên xác nhận, hệ thống chuyển deal sang AWAITING_PAYMENT (thay vì AGREED), tự động tính phí dịch vụ và tạo Paywall.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Mỗi bên xác nhận đồng thuận |
| Secondary | System | Kiểm tra song phương, tính phí, tạo Paywall, chuyển trạng thái |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Deal đang ở trạng thái IN_PROGRESS
- Actor là một trong hai bên liên quan trong deal
- Thỏa thuận nháp (DraftAgreement) đã ở trạng thái CONFIRMED (UC-56)

---

**Trigger**
> Actor nhấn "Xác nhận đồng thuận" trong trang thương thảo.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Nhấn "Xác nhận đồng thuận" trong trang thương thảo |
| 2 | System | Kiểm tra DraftAgreement.status = CONFIRMED. Nếu không: xem EF-18.1 |
| 3 | System | Kiểm tra deal có hình thức hiện vật (IN_KIND hoặc COMBINED). Nếu có: hiển thị checkbox miễn trừ trách nhiệm (xem AF-18.c) |
| 4 | System | Ghi nhận xác nhận cho bên hiện tại (organizer_confirmed hoặc sponsor_confirmed = true) |
| 5 | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã xác nhận sẵn sàng ký kết" |
| 6 | System | Kiểm tra cả hai bên đã xác nhận chưa |
| 7 | System | Nếu cả hai đã xác nhận: tính phí dịch vụ tự động từ DraftAgreement (FR-1201) |
| 8 | System | Tạo PaywallSession và hai ServiceFeeTransaction (Organizer + Sponsor) |
| 9 | System | Chuyển deal sang trạng thái AWAITING_PAYMENT |
| 10 | System | Gửi thông báo cho cả hai bên "Đồng thuận hoàn tất. Vui lòng thanh toán phí dịch vụ để tiến hành ký kết." |
| 11 | System | Use case kết thúc thành công — deal chuyển sang AWAITING_PAYMENT, Paywall sẵn sàng (→ UC-50) |

---

**Alternate Flows**

> AF-18.a: Chỉ một bên xác nhận (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện chỉ có một bên đã xác nhận |
| 6b | System | Giữ deal ở trạng thái IN_PROGRESS, chờ bên còn lại xác nhận |
| 6c | System | Use case kết thúc (chưa hoàn thành — chờ đối tác) |

> AF-18.b: Rút lại xác nhận (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | Authenticated User | Nhấn "Rút lại xác nhận" (nếu đã xác nhận trước đó và bên còn lại CHƯA xác nhận) |
| 1b | System | Ghi nhận organizer_confirmed hoặc sponsor_confirmed = false |
| 1c | System | Gửi thông báo cho đối tác "[Bên hiện tại] đã rút lại xác nhận" |
| 1d | System | Use case kết thúc |

> AF-18.c: Miễn trừ trách nhiệm hiện vật — In-kind Disclaimer (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện DraftAgreement.sponsorship_type = IN_KIND hoặc COMBINED |
| 3b | System | Hiển thị checkbox bắt buộc: "Tôi hiểu rằng UniLink KHÔNG chịu trách nhiệm về chất lượng, số lượng, và thời gian giao nhận hiện vật. Mọi tranh chấp liên quan đến hiện vật là trách nhiệm giữa hai bên." |
| 3c | Authenticated User | Tick checkbox xác nhận |
| 3d | System | Ghi nhận InKindDisclaimer (user_id, deal_id, accepted_at) |
| 3e | System | Tiếp tục Step 4 bình thường |

---

**Exception Flows**

> EF-18.1: Chưa có Thỏa thuận nháp CONFIRMED (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện DraftAgreement chưa tồn tại hoặc không ở trạng thái CONFIRMED |
| 2b | System | Từ chối: "Cần tạo và xác nhận Thỏa thuận nháp trước khi đồng thuận (→ UC-56)" |

> EF-18.2: Chưa tick miễn trừ trách nhiệm hiện vật (triggered at AF-18.c Step 3c)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3c-a | System | Phát hiện checkbox chưa được tick |
| 3c-b | System | Từ chối xác nhận: "Vui lòng xác nhận miễn trừ trách nhiệm hiện vật để tiếp tục" |

---

**Postconditions**

*Success (cả hai xác nhận):*
- Deal chuyển sang trạng thái AWAITING_PAYMENT
- PaywallSession được tạo với expires_at = now + 48h
- Hai ServiceFeeTransaction được tạo (PENDING cho mỗi bên)
- Phí dịch vụ đã tính xong từ DraftAgreement
- Paywall sẵn sàng (→ UC-50)

*Success (một bên xác nhận):*
- Deal vẫn ở IN_PROGRESS, chờ bên còn lại

*Success (rút lại):*
- Xác nhận được reset, deal vẫn ở IN_PROGRESS

---

**Business Rules**

- BR-0405: Deal chuyển từ IN_PROGRESS sang AWAITING_PAYMENT khi CẢ HAI bên xác nhận. Xác nhận có thể rút lại trước khi bên còn lại xác nhận
- BR-0407: Thỏa thuận nháp CONFIRMED là BẮT BUỘC trước khi đồng thuận
- BR-1201: Phí dịch vụ tính tự động từ sponsorship_type và sponsorship_value trong DraftAgreement
- BR-1304: Miễn trừ trách nhiệm hiện vật BẮT BUỘC cho deal có IN_KIND/COMBINED

---

**Notes / Assumptions**

- Sau khi deal AWAITING_PAYMENT, không thể hủy bỏ thương thảo qua giao diện (chỉ timeout 48h)
- PaywallSession và ServiceFeeTransaction được tạo tự động — System response trong postconditions
- Liên kết: UC-56 (thỏa thuận nháp — prerequisite), UC-50 (thanh toán phí — next step), UC-14 (nhắn tin), UC-19 (hủy thương thảo — chỉ trước khi AWAITING_PAYMENT)
