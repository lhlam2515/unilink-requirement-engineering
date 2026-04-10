# SF-13: Contact Data Masking & Unlocking

## Overview

```
Scope:            Quản lý thuật toán che giấu dữ liệu liên hệ (Data Masking) trong hệ thống
                  tin nhắn thương thảo, cơ chế gỡ bỏ masking sau thanh toán, phát hiện và
                  xử lý hành vi lách bộ lọc (anti-bypass), và xác nhận miễn trừ trách nhiệm
                  cho tài trợ hiện vật.
System Boundary:
  IN:             Áp dụng bộ lọc masking trên tin nhắn gửi đi; Gỡ bỏ masking khi thanh toán
                  hoàn tất; Phát hiện hành vi lách bộ lọc bằng từ lóng/ký tự đặc biệt;
                  Thu thập xác nhận miễn trừ trách nhiệm hiện vật.
  OUT:            Nhắn tin trao đổi trong deal (SF-04 — nơi masking được áp dụng);
                  Theo dõi thanh toán và trigger mở khóa (SF-12 — kích hoạt gỡ masking);
                  Khóa tài khoản vi phạm (quản trị hệ thống — ngoài phạm vi feature này).
Assumptions:
  - [ASSUMED] Data Masking hoạt động theo cơ chế filter trước hiển thị (pre-display filter),
    không sửa đổi nội dung gốc trong database. Nội dung gốc vẫn lưu nguyên bản.
  - [ASSUMED] Thuật toán masking sử dụng kết hợp regex pattern matching (cho số điện thoại,
    email, URL chuẩn) và từ điển phát hiện (cho từ lóng, ký tự đặc biệt biến thể).
  - [ASSUMED] Hệ thống phát hiện bypass dựa trên heuristic, không đảm bảo 100% chính xác.
    False positive (chặn nhầm) ưu tiên hơn false negative (bỏ lọt) để bảo vệ doanh thu.
  - [ASSUMED] Khóa tài khoản vĩnh viễn do vi phạm bypass cần admin review trước khi thực thi.
Gaps Detected:
  - BP03 không nêu rõ thuật toán masking cụ thể → đề xuất regex + dictionary approach.
  - Không nêu cách xử lý khi masking chặn nhầm nội dung hợp lệ → bổ sung appeal mechanism.
  - Không nêu masking cho nội dung ngoài tin nhắn (ví dụ: file đính kèm chứa contact info)
    → masking chỉ áp dụng cho text message, file attachment không quét nội dung.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Tài khoản đại diện Ban tổ chức (CLB) | `organizer` | Gửi tin nhắn (bị masking); Xem tin nhắn (đã masking); Xác nhận miễn trừ hiện vật |
| Tài khoản đại diện Doanh nghiệp | `sponsor` | Gửi tin nhắn (bị masking); Xem tin nhắn (đã masking); Xác nhận miễn trừ hiện vật |
| Hệ thống | `system` | Áp dụng filter masking; Phát hiện bypass; Gỡ masking; Ghi log vi phạm |
| Admin | `admin` | Review vi phạm bypass; Phê duyệt khóa tài khoản; Quản lý từ điển masking |

---

## Functional Requirements

### FR-1301: Áp dụng Data Masking trong tin nhắn thương thảo

```
ID:            FR-1301
Name:          Tự động che giấu thông tin liên hệ trong tin nhắn thương thảo
Description:   Hệ thống SHALL tự động áp dụng bộ lọc Data Masking trên TẤT CẢ tin nhắn
               văn bản trong deal context khi deal chưa hoàn tất thanh toán phí dịch vụ
               (deal.status ≠ AGREED hoặc deal.contact_unlocked = false).
               Bộ lọc SHALL phát hiện và thay thế bằng ký tự che giấu (ví dụ: "***")
               các loại thông tin liên hệ:
               - Số điện thoại (VN: 10 chữ số, các định dạng phổ biến)
               - Địa chỉ email
               - Link mạng xã hội (Facebook, Zalo, Instagram, LinkedIn, v.v.)
               - Số tài khoản ngân hàng
               Nội dung gốc vẫn được lưu trữ trong database; masking chỉ áp dụng khi
               hiển thị (pre-display filter).
Classification: FULLY AUTOMATED
Actor:         System (filter), tác động lên Organizer và Sponsor
Trigger:       Tin nhắn mới được gửi trong deal context (SF-04 FR-0401)
Inputs:        message_content (text), deal.contact_unlocked (boolean)
Outputs:       display_content (text — đã masking), masking_applied (boolean),
               masked_patterns_count (integer)
Business Rules: BR-1301
Acceptance Criteria:
  Given   deal chưa hoàn tất thanh toán (contact_unlocked = false)
  When    organizer gửi tin nhắn "Liên hệ tôi qua 0912345678 hoặc email@example.com"
  Then    hệ thống SHALL lưu nội dung gốc trong database
  And     hệ thống SHALL hiển thị "Liên hệ tôi qua ********** hoặc *****@*****"
  And     masking_applied = true, masked_patterns_count = 2

  Given   deal đã mở khóa (contact_unlocked = true)
  When    organizer gửi tin nhắn "Liên hệ tôi qua 0912345678"
  Then    hệ thống SHALL hiển thị nguyên bản "Liên hệ tôi qua 0912345678"
  And     masking_applied = false

  Given   deal chưa mở khóa
  When    sponsor gửi "Xem thêm tại facebook.com/doanhnghiep"
  Then    hệ thống SHALL hiển thị "Xem thêm tại **********************"
Priority:      MUST
```

### FR-1302: Gỡ bỏ Data Masking khi thanh toán hoàn tất

```
ID:            FR-1302
Name:          Tự động gỡ bỏ Data Masking và hiển thị nội dung gốc
Description:   Hệ thống SHALL tự động gỡ bỏ bộ lọc Data Masking cho toàn bộ deal context
               khi được kích hoạt bởi SF-12 FR-1206 (2/2 thanh toán hoàn tất).
               Sau khi gỡ masking:
               - Tất cả tin nhắn CŨ SHALL hiển thị nội dung gốc (re-render từ database).
               - Tất cả tin nhắn MỚI SHALL không bị masking.
               Hệ thống SHALL ghi nhận thời điểm gỡ masking cho audit trail.
Classification: FULLY AUTOMATED
Actor:         System (kích hoạt bởi SF-12)
Trigger:       SF-12 FR-1206 kích hoạt sự kiện CONTACT_UNLOCKED
Inputs:        deal_id, unlocked_at (timestamp)
Outputs:       deal.contact_unlocked = true,
               tất cả tin nhắn hiển thị nội dung gốc không masking
Business Rules: BR-1302
Acceptance Criteria:
  Given   deal đã có 15 tin nhắn với masking (một số chứa SĐT, email bị che)
  When    hệ thống nhận sự kiện CONTACT_UNLOCKED từ SF-12
  Then    hệ thống SHALL cập nhật deal.contact_unlocked = true
  And     tất cả 15 tin nhắn SHALL hiển thị lại nội dung gốc không masking
  And     tin nhắn mới sau đó SHALL không bị masking

  Given   deal đã unlocked
  When    gỡ masking hoàn tất
  Then    hệ thống SHALL KHÔNG BAO GIỜ áp dụng lại masking cho deal này
         (trạng thái unlocked là vĩnh viễn cho deal này)
Priority:      MUST
```

### FR-1303: Phát hiện và xử lý hành vi lách bộ lọc (Anti-Bypass)

```
ID:            FR-1303
Name:          Phát hiện hành vi cố tình lách bộ lọc Data Masking
Description:   Hệ thống SHALL triển khai cơ chế phát hiện hành vi cố tình lách bộ lọc
               Data Masking bằng cách sử dụng từ lóng, ký tự đặc biệt, khoảng trắng chen giữa,
               hoặc biến thể Unicode để truyền tải thông tin liên hệ. Ví dụ:
               - "không chín một hai bốn năm sáu bảy tám" (viết số bằng chữ)
               - "0-9-1-2-3-4-5-6-7-8" (chen ký tự đặc biệt)
               - "email tại abc chấm com" (mô tả thay vì viết trực tiếp)
               Khi phát hiện hành vi bypass:
               1. Tin nhắn SHALL bị chặn gửi và đánh dấu FLAGGED.
               2. Hệ thống SHALL cảnh báo người gửi.
               3. Hệ thống SHALL ghi log vi phạm (BypassViolationLog).
               4. Sau 3 lần vi phạm → hệ thống SHALL khóa tạm thời tính năng nhắn tin.
               5. Admin review để quyết định khóa vĩnh viễn theo chính sách Zero Tolerance.
Classification: FULLY AUTOMATED (phát hiện) + HUMAN (admin review khóa vĩnh viễn)
Actor:         System (phát hiện), Admin (review và quyết định)
Trigger:       Tin nhắn mới gửi trong deal context (chạy song song với FR-1301)
Inputs:        message_content (text), sender_id (UUID)
Outputs:       bypass_detected (boolean), violation_log_id (UUID, nếu phát hiện),
               message_status (SENT | FLAGGED | BLOCKED),
               sender_warning_count (integer)
Business Rules: BR-1303
Acceptance Criteria:
  Given   deal chưa mở khóa
  When    organizer gửi "liên hệ zalo tôi đi: không chín một hai 3 4 5 6 78"
  Then    hệ thống SHALL phát hiện hành vi bypass
  And     tin nhắn SHALL bị FLAGGED, không gửi đến sponsor
  And     hệ thống SHALL cảnh báo organizer "Tin nhắn bị chặn: phát hiện hành vi trao đổi
          thông tin liên hệ trước khi thanh toán. Vi phạm: 1/3"

  Given   organizer đã bị cảnh báo 3 lần
  When    organizer gửi tin nhắn vi phạm lần thứ 4
  Then    hệ thống SHALL khóa tạm thời tính năng nhắn tin của organizer trong deal này
  And     hệ thống SHALL thông báo admin để review

  Given   admin review vi phạm của tài khoản
  When    admin xác nhận hành vi cố tình
  Then    admin CÓ THỂ khóa tài khoản vĩnh viễn theo chính sách Zero Tolerance
Priority:      MUST
```

### FR-1304: Xác nhận miễn trừ trách nhiệm hiện vật

```
ID:            FR-1304
Name:          Thu thập xác nhận miễn trừ trách nhiệm chất lượng và vận chuyển hiện vật
Description:   Hệ thống SHALL yêu cầu CẢ HAI bên tích chọn xác nhận miễn trừ trách nhiệm
               cho nền tảng về chất lượng và vận chuyển hiện vật khi deal liên quan đến
               tài trợ hiện vật (IN_KIND hoặc COMBINED). Xác nhận này là BẮT BUỘC trước khi
               deal có thể chuyển sang AWAITING_PAYMENT. Nội dung miễn trừ nêu rõ:
               "Nền tảng không chịu trách nhiệm về chất lượng sản phẩm hiện vật
               và không can thiệp vào quá trình vận chuyển, bàn giao hiện vật."
Classification: SYSTEM-SUPPORTED
Actor:         Organizer, Sponsor
Trigger:       Hai bên xác nhận đồng thuận (SF-04 FR-0406) cho deal có hiện vật
Inputs:        deal_id, sponsorship_type (IN_KIND | COMBINED),
               organizer_disclaimer_accepted (boolean),
               sponsor_disclaimer_accepted (boolean)
Outputs:       deal.disclaimer_accepted = true (khi cả hai đã tích chọn),
               cho phép chuyển sang AWAITING_PAYMENT
Business Rules: BR-1304
Acceptance Criteria:
  Given   deal có sponsorship_type = IN_KIND
  And     hai bên đã đồng thuận (FR-0406)
  When    hệ thống kiểm tra trước khi chuyển sang AWAITING_PAYMENT
  Then    hệ thống SHALL hiển thị checkbox miễn trừ trách nhiệm cho cả hai bên
  And     deal SHALL KHÔNG chuyển sang AWAITING_PAYMENT cho đến khi cả hai tích chọn

  Given   deal có sponsorship_type = CASH (chỉ tiền mặt)
  When    hai bên đồng thuận
  Then    hệ thống SHALL KHÔNG yêu cầu xác nhận miễn trừ (bỏ qua bước này)

  Given   organizer đã tích chọn nhưng sponsor chưa
  When    hệ thống kiểm tra
  Then    hệ thống SHALL hiển thị "Chờ đối tác xác nhận miễn trừ trách nhiệm"
Priority:      MUST
```

---

## Business Rules

```
ID:          BR-1301
Rule:        Data Masking áp dụng cho TẤT CẢ tin nhắn văn bản trong deal context khi
             deal.contact_unlocked = false. Các loại thông tin bị masking:
             - Số điện thoại VN: 10 chữ số, bắt đầu bằng 0 (các pattern: 0xxx, +84xxx)
             - Email: pattern username@domain.tld
             - URL mạng xã hội: facebook.com/*, zalo.me/*, instagram.com/*, linkedin.com/*
             - URL tổng quát: http://* hoặc https://*
             - Số tài khoản ngân hàng: chuỗi 6-19 chữ số liên tiếp
             Masking KHÔNG áp dụng cho file đính kèm (chỉ text message).
             Nội dung gốc PHẢI được lưu nguyên bản trong database.
Source:      BP03 — Sec 1 (Data Masking: SĐT, Email, Link Mạng xã hội)
Type:        Validation + Data Protection

ID:          BR-1302
Rule:        Gỡ bỏ masking là VĨNH VIỄN cho deal đã unlocked.
             Sau khi contact_unlocked = true, hệ thống KHÔNG BAO GIỜ áp dụng lại masking
             cho deal đó, kể cả khi phát sinh tranh chấp hoặc hủy hợp đồng sau này.
Source:      BP03 — Bước 3 (ngay khi 2/2, masking được gỡ bỏ)
Type:        Routing

ID:          BR-1303
Rule:        Chính sách Zero Tolerance: bất kỳ tài khoản nào cố tình lách bộ lọc nội dung
             để trao đổi thông tin liên hệ trước khi thanh toán phí SHALL bị xử lý:
             - Vi phạm 1-2: Cảnh báo + tin nhắn bị FLAGGED
             - Vi phạm 3: Khóa tạm thời tính năng nhắn tin trong deal
             - Vi phạm 3+: Admin review → có thể khóa tài khoản VĨNH VIỄN
             Thuật toán phát hiện bypass ưu tiên false positive (chặn nhầm) hơn false negative
             (bỏ lọt) để bảo vệ doanh thu nền tảng. Người dùng có thể appeal false positive.
Source:      BP03 — Quy tắc 3 (Zero Tolerance for Bypassing)
Type:        Authorization + Validation

ID:          BR-1304
Rule:        Xác nhận miễn trừ trách nhiệm hiện vật là BẮT BUỘC cho deal có sponsorship_type
             = IN_KIND hoặc COMBINED. CẢ HAI bên PHẢI tích chọn trước khi deal chuyển sang
             AWAITING_PAYMENT. Deal chỉ có CASH được miễn bước này.
Source:      BP03 — Bước 1 (hai bên phải tích chọn xác nhận miễn trừ trách nhiệm)
Type:        Validation
```

---

## Data Model

```
Entity:        MaskingRule
Description:   Quy tắc regex/dictionary cho bộ lọc Data Masking, Admin có thể cập nhật.
Attributes:
  - rule_id: UUID (PK)
  - rule_name: String (required — ví dụ: "VN Phone Number")
  - pattern_type: Enum [REGEX, DICTIONARY, HEURISTIC]
  - pattern_value: Text (required — regex pattern hoặc từ điển)
  - replacement_text: String (default: "***")
  - is_active: Boolean (default: true)
  - priority: Integer (thứ tự áp dụng)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - MaskingRule —(standalone)→ áp dụng bởi System

Entity:        BypassViolationLog
Description:   Nhật ký vi phạm lách bộ lọc Data Masking.
Attributes:
  - violation_id: UUID (PK)
  - deal_id: UUID (FK → Deal)
  - violator_id: UUID (FK → User)
  - message_content: Text (nội dung tin nhắn vi phạm — lưu nguyên bản)
  - detection_method: Enum [REGEX, DICTIONARY, HEURISTIC, ADMIN_REPORT]
  - violation_count: Integer (lần vi phạm thứ mấy của user trong deal này)
  - action_taken: Enum [WARNING, MESSAGE_BLOCKED, CHAT_SUSPENDED, ACCOUNT_BANNED]
  - admin_reviewed: Boolean (default: false)
  - admin_reviewed_by: UUID (FK → Admin, nullable)
  - admin_reviewed_at: DateTime (nullable)
  - admin_decision: Enum [CONFIRMED_VIOLATION, FALSE_POSITIVE, ESCALATED] (nullable)
  - created_at: DateTime
Relationships:
  - BypassViolationLog —(N:1)→ Deal
  - BypassViolationLog —(N:1)→ User

Entity:        InKindDisclaimer
Description:   Bản ghi xác nhận miễn trừ trách nhiệm hiện vật cho mỗi deal.
Attributes:
  - disclaimer_id: UUID (PK)
  - deal_id: UUID (FK → Deal, UNIQUE)
  - organizer_accepted: Boolean (default: false)
  - organizer_accepted_at: DateTime (nullable)
  - sponsor_accepted: Boolean (default: false)
  - sponsor_accepted_at: DateTime (nullable)
  - disclaimer_text: Text (nội dung miễn trừ — versioned)
Relationships:
  - InKindDisclaimer —(1:1)→ Deal
```

---

## Traceability Matrix

| Process Step (BP03) | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| Sec 1. Data Masking trong chat | FULLY AUTOMATED | FR-1301 | BR-1301 | MaskingRule |
| Bước 3. Gỡ bỏ masking khi 2/2 thanh toán | FULLY AUTOMATED | FR-1302 | BR-1302 | Deal |
| Quy tắc 3. Zero Tolerance anti-bypass | FULLY AUTOMATED + HUMAN | FR-1303 | BR-1303 | BypassViolationLog |
| Bước 1. Miễn trừ trách nhiệm hiện vật | SYSTEM-SUPPORTED | FR-1304 | BR-1304 | InKindDisclaimer |
