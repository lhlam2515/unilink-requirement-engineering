# SF-03: Sponsorship Invitation Management

## Overview

```
Scope:            Quản lý quy trình gửi, nhận, chấp nhận và từ chối lời mời tài trợ hai chiều
                  giữa BTC và doanh nghiệp. Đây là cổng kết nối chính thức giữa hai bên
                  trước khi tiến vào giai đoạn thương thảo.
System Boundary:
  IN:             Gửi lời mời tài trợ (cả hai chiều); Chấp nhận/từ chối lời mời;
                  Theo dõi trạng thái lời mời; Thông báo tự động qua in-app và email.
  OUT:            Tìm kiếm đối tác (SF-02); Thương thảo hợp đồng (SF-04) — bắt đầu sau khi
                  lời mời được chấp nhận.
Assumptions:
  - [ASSUMED] Cả BTC và doanh nghiệp đều có thể là bên gửi hoặc bên nhận lời mời.
  - [ASSUMED] Mỗi cặp (hồ sơ tài trợ + doanh nghiệp) chỉ có thể có MỘT lời mời active tại một thời điểm.
  - [ASSUMED] Lý do từ chối là tùy chọn (optional) nhưng được khuyến khích.
Gaps Detected:
  - Quy trình gốc không nêu thời hạn hiệu lực của lời mời → cần bổ sung quy tắc hết hạn.
  - Không nêu rõ BTC gửi lời mời cho toàn doanh nghiệp hay cho một đại diện cụ thể.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Ban tổ chức (BTC) | `organizer` | Gửi lời mời tài trợ đến doanh nghiệp; Nhận và phản hồi lời mời từ doanh nghiệp |
| Doanh nghiệp | `sponsor` | Gửi lời mời tài trợ đến BTC (dựa trên hồ sơ sự kiện); Nhận và phản hồi lời mời từ BTC |
| Hệ thống | `system` | Xác thực tính hợp lệ lời mời, gửi thông báo, quản lý trạng thái, xử lý hết hạn tự động |

---

## Functional Requirements

### FR-0301: Gửi lời mời tài trợ

```
ID:            FR-0301
Name:          Gửi lời mời tài trợ cho đối tác
Description:   Hệ thống SHALL cho phép organizer gửi lời mời tài trợ đến một doanh nghiệp
               dựa trên một hồ sơ tài trợ cụ thể, hoặc cho phép sponsor gửi lời mời tài trợ
               đến BTC dựa trên một hồ sơ tài trợ cụ thể. Lời mời PHẢI bao gồm tin nhắn giới thiệu
               và liên kết đến hồ sơ tài trợ liên quan. Hệ thống SHALL xác thực tính hợp lệ
               theo BR-0301 trước khi tạo lời mời.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer hoặc Sponsor (khởi tạo)
Trigger:       Actor nhấn "Gửi lời mời tài trợ" từ trang chi tiết hồ sơ đối tác hoặc hồ sơ sự kiện
Inputs:        sender_id (UUID), sender_role (enum: organizer | sponsor),
               recipient_id (UUID), proposal_id (UUID),
               message (text, required), preferred_package_id (UUID, optional)
Outputs:       invitation_id (UUID), status = PENDING, sent_at (timestamp)
Business Rules: BR-0301, BR-0302, BR-0303
Acceptance Criteria:
  Given   organizer đang xem hồ sơ doanh nghiệp "ABC Corp"
  And     hồ sơ tài trợ "UniHack 2026" đang ở trạng thái PUBLISHED
  When    organizer nhấn "Gửi lời mời" với message = "Kính mời quý công ty hợp tác tài trợ..."
  Then    hệ thống SHALL tạo lời mời với trạng thái PENDING
  And     hệ thống SHALL gửi thông báo in-app và email cho "ABC Corp"

  Given   đã tồn tại lời mời PENDING giữa organizer và "ABC Corp" cho hồ sơ "UniHack 2026"
  When    organizer gửi lời mời mới cho cùng doanh nghiệp và hồ sơ
  Then    hệ thống SHALL từ chối với thông báo "Đã tồn tại lời mời đang chờ xử lý cho đối tác này"
Priority:      MUST
```

### FR-0302: Thông báo lời mời tài trợ mới

```
ID:            FR-0302
Name:          Thông báo cho bên nhận về lời mời tài trợ mới
Description:   Hệ thống SHALL tự động gửi thông báo qua kênh in-app và email đến bên nhận
               khi có lời mời tài trợ mới. Thông báo PHẢI bao gồm: tên bên gửi, tên sự kiện
               liên quan, tin nhắn giới thiệu, và liên kết đến trang chi tiết lời mời.
Classification: FULLY AUTOMATED
Actor:         System (khởi tạo), Recipient (nhận thông báo)
Trigger:       Lời mời tài trợ mới được tạo thành công (FR-0301)
Inputs:        invitation_id, recipient_id, sender_name, proposal_name
Outputs:       notification_id, delivery_status (SENT | DELIVERED | FAILED),
               sent_via (enum[]: IN_APP, EMAIL)
Business Rules: BR-0304
Acceptance Criteria:
  Given   lời mời tài trợ vừa được tạo cho doanh nghiệp "ABC Corp"
  When    hệ thống xử lý sự kiện tạo lời mời
  Then    hệ thống SHALL gửi thông báo in-app đến "ABC Corp" trong vòng 30 giây
  And     hệ thống SHALL gửi email thông báo đến email đăng ký của "ABC Corp"
  And     thông báo SHALL chứa tên BTC gửi, tên sự kiện, và nội dung tin nhắn giới thiệu
Priority:      MUST
```

### FR-0303: Chấp nhận lời mời tài trợ

```
ID:            FR-0303
Name:          Chấp nhận lời mời tài trợ
Description:   Hệ thống SHALL cho phép bên nhận chấp nhận lời mời tài trợ. Khi chấp nhận,
               hệ thống SHALL chuyển trạng thái lời mời sang ACCEPTED, tự động tạo một
               deal/negotiation context (liên kết đến SF-04), và thông báo cho bên gửi.
Classification: SYSTEM-SUPPORTED
Actor:         Recipient (Organizer hoặc Sponsor tùy chiều lời mời)
Trigger:       Bên nhận nhấn "Chấp nhận lời mời"
Inputs:        invitation_id, response_message (text, optional)
Outputs:       status = ACCEPTED, accepted_at (timestamp), deal_id (UUID — tạo tự động cho SF-04)
Business Rules: BR-0305
Acceptance Criteria:
  Given   doanh nghiệp "ABC Corp" nhận được lời mời tài trợ từ BTC cho "UniHack 2026"
  And     lời mời đang ở trạng thái PENDING
  When    "ABC Corp" nhấn "Chấp nhận"
  Then    hệ thống SHALL chuyển trạng thái sang ACCEPTED
  And     hệ thống SHALL tạo deal_id mới cho giai đoạn thương thảo (SF-04)
  And     hệ thống SHALL gửi thông báo in-app và email cho BTC

  Given   lời mời đã ở trạng thái EXPIRED
  When    bên nhận cố chấp nhận
  Then    hệ thống SHALL từ chối với thông báo "Lời mời đã hết hạn"
Priority:      MUST
```

### FR-0304: Từ chối lời mời tài trợ

```
ID:            FR-0304
Name:          Từ chối lời mời tài trợ
Description:   Hệ thống SHALL cho phép bên nhận từ chối lời mời tài trợ. Khi từ chối,
               hệ thống SHALL chuyển trạng thái lời mời sang DECLINED và thông báo cho bên gửi.
               Bên nhận có thể nhập lý do từ chối (tùy chọn).
Classification: SYSTEM-SUPPORTED
Actor:         Recipient (Organizer hoặc Sponsor)
Trigger:       Bên nhận nhấn "Từ chối lời mời"
Inputs:        invitation_id, decline_reason (text, optional)
Outputs:       status = DECLINED, declined_at (timestamp)
Business Rules: BR-0305
Acceptance Criteria:
  Given   doanh nghiệp nhận được lời mời tài trợ đang ở trạng thái PENDING
  When    doanh nghiệp nhấn "Từ chối" với lý do = "Không phù hợp ngân sách hiện tại"
  Then    hệ thống SHALL chuyển trạng thái sang DECLINED
  And     hệ thống SHALL gửi thông báo cho BTC bao gồm lý do từ chối (nếu có)

  Given   doanh nghiệp nhấn "Từ chối" mà không nhập lý do
  When    hệ thống xử lý
  Then    hệ thống SHALL chấp nhận từ chối không có lý do (lý do là optional)
Priority:      MUST
```

### FR-0305: Xử lý hết hạn lời mời tự động

```
ID:            FR-0305
Name:          Tự động hết hạn lời mời không được phản hồi
Description:   Hệ thống SHALL tự động chuyển trạng thái lời mời từ PENDING sang EXPIRED
               nếu bên nhận không phản hồi (chấp nhận hoặc từ chối) trong thời hạn quy định
               tại BR-0306. Hệ thống SHALL thông báo cho cả hai bên khi lời mời hết hạn.
Classification: FULLY AUTOMATED
Actor:         System (khởi tạo)
Trigger:       Scheduled job kiểm tra lời mời quá hạn (chạy mỗi giờ) [ASSUMED]
Inputs:        Danh sách lời mời có trạng thái PENDING và sent_at + expiry_duration < now
Outputs:       status = EXPIRED, expired_at (timestamp), notification cho cả hai bên
Business Rules: BR-0306
Acceptance Criteria:
  Given   lời mời tài trợ gửi ngày 01/05/2026 với thời hạn 14 ngày
  And     hôm nay là 16/05/2026
  And     lời mời vẫn ở trạng thái PENDING
  When    hệ thống chạy scheduled job kiểm tra hết hạn
  Then    hệ thống SHALL chuyển trạng thái sang EXPIRED
  And     hệ thống SHALL thông báo cho bên gửi "Lời mời tài trợ cho [tên sự kiện] đã hết hạn"
  And     hệ thống SHALL thông báo cho bên nhận "Lời mời tài trợ từ [tên BTC] đã hết hạn"
Priority:      SHOULD
```

### FR-0306: Theo dõi danh sách lời mời tài trợ

```
ID:            FR-0306
Name:          Xem và quản lý danh sách lời mời tài trợ
Description:   Hệ thống SHALL cho phép actor xem danh sách tất cả lời mời tài trợ đã gửi và
               đã nhận, lọc theo trạng thái (PENDING, ACCEPTED, DECLINED, EXPIRED).
               Danh sách hiển thị thông tin tóm tắt: đối tác, sự kiện, trạng thái, ngày gửi.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Actor truy cập trang "Lời mời tài trợ" trong dashboard
Inputs:        actor_id, filter_status (enum: ALL | PENDING | ACCEPTED | DECLINED | EXPIRED),
               filter_direction (enum: SENT | RECEIVED | ALL), page, page_size
Outputs:       invitations[]: { invitation_id, partner_name, proposal_name, status,
               sent_at, responded_at }, total_count
Business Rules: Không có BR riêng
Acceptance Criteria:
  Given   organizer đã gửi 5 lời mời và nhận 3 lời mời
  When    organizer truy cập trang "Lời mời tài trợ" với filter_direction = SENT
  Then    hệ thống SHALL hiển thị 5 lời mời đã gửi
  And     mỗi lời mời hiển thị trạng thái hiện tại (PENDING/ACCEPTED/DECLINED/EXPIRED)
Priority:      MUST
```

---

## Business Rules

```
ID:          BR-0301
Rule:        Lời mời tài trợ chỉ có thể được gửi đến hồ sơ tài trợ ở trạng thái PUBLISHED.
             Lời mời đến hồ sơ DRAFT SHALL bị từ chối.
Source:      Quy trình gốc — Bước 2.3 (Gửi lời mời tài trợ)
Type:        Validation

ID:          BR-0302
Rule:        Mỗi cặp (proposal_id + target_business_id) chỉ có thể có MỘT lời mời
             ở trạng thái PENDING tại một thời điểm. Lời mời mới chỉ được gửi khi
             lời mời trước đó đã ở trạng thái ACCEPTED, DECLINED, hoặc EXPIRED.
Source:      [INFERRED — ngăn spam lời mời]
Type:        Validation

ID:          BR-0303
Rule:        Tin nhắn giới thiệu (message) trong lời mời là BẮT BUỘC và PHẢI có
             ít nhất 20 ký tự. [ASSUMED — đảm bảo chất lượng lời mời]
Source:      [INFERRED — chất lượng giao tiếp]
Type:        Validation

ID:          BR-0304
Rule:        Thông báo lời mời mới PHẢI được gửi qua cả hai kênh: in-app và email.
             Thông báo in-app SHALL được gửi trong vòng 30 giây.
             Email SHALL được gửi trong vòng 5 phút.
Source:      Quy trình gốc — Bước 2.3 + quyết định thiết kế
Type:        Time-based

ID:          BR-0305
Rule:        Chỉ lời mời ở trạng thái PENDING mới có thể được chấp nhận hoặc từ chối.
             Lời mời đã ACCEPTED, DECLINED, hoặc EXPIRED không thể thay đổi trạng thái.
Source:      Quy trình gốc — Bước 2.3 (Sau khi nhận được lời mời, đối phương có thể từ chối hoặc đồng ý)
Type:        Routing

ID:          BR-0306
Rule:        Lời mời tài trợ PENDING tự động hết hạn sau 14 ngày kể từ ngày gửi. [ASSUMED]
Source:      [INFERRED — ngăn lời mời treo vô thời hạn]
Type:        Time-based
```

---

## Data Model

```
Entity:        SponsorshipInvitation
Attributes:
  - invitation_id: UUID (PK)
  - proposal_id: UUID (FK → SponsorshipProposal)
  - sender_id: UUID (FK → User)
  - sender_role: Enum [ORGANIZER, SPONSOR]
  - recipient_id: UUID (FK → User)
  - message: Text (required, min 20 chars)
  - preferred_package_id: UUID (FK → SponsorshipPackage, nullable)
  - status: Enum [PENDING, ACCEPTED, DECLINED, EXPIRED] (default: PENDING)
  - decline_reason: Text (nullable)
  - response_message: Text (nullable)
  - deal_id: UUID (FK → Deal, nullable — populated on ACCEPTED)
  - sent_at: DateTime
  - responded_at: DateTime (nullable)
  - expired_at: DateTime (nullable)
Relationships:
  - SponsorshipInvitation —(N:1)→ SponsorshipProposal
  - SponsorshipInvitation —(N:1)→ User (sender)
  - SponsorshipInvitation —(N:1)→ User (recipient)
  - SponsorshipInvitation —(1:1)→ Deal (when ACCEPTED, links to SF-04)
Constraints:
  - UNIQUE(proposal_id, sender_id, recipient_id) WHERE status = PENDING
    — chỉ 1 lời mời pending cho mỗi cặp quan hệ trên mỗi hồ sơ
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 2.3 BTC/DN gửi lời mời tài trợ | SYSTEM-SUPPORTED | FR-0301 | BR-0301, BR-0302, BR-0303 | SponsorshipInvitation |
| 2.3 Thông báo lời mời mới | FULLY AUTOMATED | FR-0302 | BR-0304 | Notification |
| 2.3 Đồng ý lời mời tài trợ | SYSTEM-SUPPORTED | FR-0303 | BR-0305 | SponsorshipInvitation, Deal |
| 2.3 Từ chối lời mời tài trợ | SYSTEM-SUPPORTED | FR-0304 | BR-0305 | SponsorshipInvitation |
| [INFERRED] Hết hạn tự động | FULLY AUTOMATED | FR-0305 | BR-0306 | SponsorshipInvitation |
| [INFERRED] Theo dõi lời mời | SYSTEM-SUPPORTED | FR-0306 | — | SponsorshipInvitation |
