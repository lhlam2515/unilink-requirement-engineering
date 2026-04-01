# SF-07: Review & Reputation

## Overview

```
Scope:            Quản lý quy trình đánh giá hai chiều sau khi hợp đồng tài trợ kết thúc.
                  Cả BTC và doanh nghiệp đánh giá mức độ uy tín và chất lượng hợp tác của
                  đối phương. Điểm đánh giá tổng hợp hiển thị trên hồ sơ công khai.
System Boundary:
  IN:             Gửi đánh giá/phản hồi sau hợp đồng; Tính điểm uy tín tổng hợp;
                  Hiển thị điểm uy tín trên hồ sơ; Ngăn chặn đánh giá trùng lặp.
  OUT:            Thực hiện nghĩa vụ (SF-06 — đã hoàn tất); Trang hồ sơ công khai
                  (hiển thị điểm uy tín — được tích hợp vào SF-02 và feature hồ sơ doanh nghiệp).
Assumptions:
  - [ASSUMED] Đánh giá chỉ mở sau khi hợp đồng kết thúc (validity_end đã qua hoặc tất cả
    nghĩa vụ đã CONFIRMED).
  - [ASSUMED] Mỗi bên được đánh giá đối phương MỘT LẦN cho mỗi hợp đồng.
  - [ASSUMED] Thang điểm đánh giá: 1-5 sao.
  - [ASSUMED] Đánh giá ẩn danh một phần (điểm số hiển thị, nội dung review hiển thị
    nhưng không tiết lộ danh tính người review cho bên đối tác — tùy chọn thiết kế).
Gaps Detected:
  - Quy trình gốc chỉ nêu "đánh giá, phản hồi về mức độ uy tín và chất lượng hợp tác"
    nhưng không chi tiết tiêu chí đánh giá → cần bổ sung.
  - Không nêu cơ chế xử lý đánh giá vi phạm (ngôn ngữ không phù hợp, spam).
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Ban tổ chức (BTC) | `organizer` | Gửi đánh giá cho sponsor; Xem đánh giá mà mình nhận được |
| Doanh nghiệp | `sponsor` | Gửi đánh giá cho organizer; Xem đánh giá mà mình nhận được |
| Hệ thống | `system` | Kiểm tra điều kiện đánh giá, tính điểm tổng hợp, kiểm duyệt nội dung |

---

## Functional Requirements

### FR-0701: Gửi đánh giá đối tác sau hợp đồng

```
ID:            FR-0701
Name:          Gửi đánh giá và phản hồi về đối tác
Description:   Hệ thống SHALL cho phép mỗi bên (organizer và sponsor) gửi đánh giá về đối tác
               sau khi hợp đồng kết thúc. Đánh giá bao gồm: điểm uy tín (1-5 sao),
               điểm chất lượng hợp tác (1-5 sao), và nhận xét văn bản (tùy chọn).
               Hệ thống SHALL chỉ cho phép đánh giá khi điều kiện tại BR-0701 được đáp ứng.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer hoặc Sponsor
Trigger:       Actor nhấn "Đánh giá đối tác" trên trang hợp đồng đã kết thúc,
               hoặc từ thông báo nhắc đánh giá
Inputs:        contract_id, reviewer_id (UUID), reviewer_role (enum: ORGANIZER | SPONSOR),
               credibility_rating (integer: 1-5), collaboration_quality_rating (integer: 1-5),
               review_comment (text, optional)
Outputs:       review_id (UUID), submitted_at (timestamp),
               notification cho bên được đánh giá
Business Rules: BR-0701, BR-0702, BR-0703
Acceptance Criteria:
  Given   hợp đồng "contract-001" đã kết thúc (validity_end < hôm nay)
  And     organizer chưa đánh giá sponsor cho hợp đồng này
  When    organizer gửi đánh giá với credibility = 4, quality = 5,
          comment = "Doanh nghiệp chuyển khoản đúng hạn, hợp tác rất chuyên nghiệp"
  Then    hệ thống SHALL lưu đánh giá thành công
  And     hệ thống SHALL thông báo cho sponsor "Bạn đã nhận được đánh giá mới"

  Given   organizer đã đánh giá sponsor trước đó cho cùng hợp đồng
  When    organizer cố gửi đánh giá lần hai
  Then    hệ thống SHALL từ chối "Bạn đã đánh giá đối tác cho hợp đồng này rồi"

  Given   hợp đồng vẫn đang hiệu lực (validity_end > hôm nay)
  When    actor cố gửi đánh giá
  Then    hệ thống SHALL từ chối "Chỉ có thể đánh giá sau khi hợp đồng kết thúc"
Priority:      MUST
```

### FR-0702: Thông báo nhắc đánh giá

```
ID:            FR-0702
Name:          Gửi thông báo nhắc nhở đánh giá đối tác
Description:   Hệ thống SHALL tự động gửi thông báo nhắc nhở đánh giá qua in-app và email
               cho cả hai bên khi hợp đồng kết thúc. Nhắc nhở gửi vào ngày kết thúc hợp đồng
               và nhắc lại sau 7 ngày nếu chưa đánh giá.
Classification: FULLY AUTOMATED
Actor:         System (khởi tạo)
Trigger:       Scheduled job kiểm tra hợp đồng kết thúc (chạy hàng ngày)
Inputs:        Danh sách hợp đồng kết thúc mà bên liên quan chưa gửi đánh giá
Outputs:       Notifications cho actors chưa đánh giá
Business Rules: BR-0704
Acceptance Criteria:
  Given   hợp đồng "contract-001" kết thúc ngày 30/06/2026
  And     hôm nay là 30/06/2026
  And     cả hai bên chưa gửi đánh giá
  When    scheduled job chạy
  Then    hệ thống SHALL gửi thông báo cho organizer và sponsor
          "Hợp đồng [tên] đã kết thúc. Vui lòng đánh giá đối tác."

  Given   hôm nay là 07/07/2026 và organizer vẫn chưa đánh giá
  When    scheduled job chạy
  Then    hệ thống SHALL gửi nhắc nhở lần 2 cho organizer
Priority:      SHOULD
```

### FR-0703: Tính điểm uy tín tổng hợp

```
ID:            FR-0703
Name:          Tính và cập nhật điểm uy tín tổng hợp
Description:   Hệ thống SHALL tự động tính điểm uy tín tổng hợp cho mỗi tổ chức (BTC hoặc
               doanh nghiệp) dựa trên tất cả đánh giá đã nhận. Điểm tổng hợp là trung bình
               cộng có trọng số theo công thức tại BR-0705. Điểm được cập nhật khi có đánh giá mới.
Classification: FULLY AUTOMATED
Actor:         System
Trigger:       Đánh giá mới được gửi thành công (FR-0701)
Inputs:        Tất cả reviews cho target_entity_id
Outputs:       aggregated_credibility_score (decimal: 0.0-5.0),
               aggregated_quality_score (decimal: 0.0-5.0),
               total_review_count (integer)
Business Rules: BR-0705
Acceptance Criteria:
  Given   doanh nghiệp "ABC Corp" đã nhận 3 đánh giá credibility: [4, 5, 3]
  When    hệ thống tính điểm tổng hợp
  Then    aggregated_credibility_score = (4 + 5 + 3) / 3 = 4.0
  And     total_review_count = 3

  Given   đánh giá mới credibility = 5 được gửi cho "ABC Corp"
  When    hệ thống cập nhật
  Then    aggregated_credibility_score = (4 + 5 + 3 + 5) / 4 = 4.25
  And     total_review_count = 4
Priority:      MUST
```

### FR-0704: Hiển thị điểm uy tín trên hồ sơ

```
ID:            FR-0704
Name:          Hiển thị điểm uy tín và đánh giá trên hồ sơ công khai
Description:   Hệ thống SHALL hiển thị điểm uy tín tổng hợp và danh sách đánh giá gần đây
               trên trang hồ sơ công khai của BTC và doanh nghiệp. Hiển thị bao gồm:
               điểm uy tín trung bình, điểm chất lượng trung bình, tổng số đánh giá,
               và danh sách đánh giá với nội dung nhận xét.
Classification: SYSTEM-SUPPORTED
Actor:         Bất kỳ actor đã xác thực (xem)
Trigger:       Actor truy cập trang hồ sơ công khai của BTC hoặc doanh nghiệp
Inputs:        entity_id, entity_type (ORGANIZATION | BUSINESS)
Outputs:       reputation_summary: { credibility_avg, quality_avg, total_reviews },
               recent_reviews[]: { rating, comment, reviewed_at, contract_event_name }
Business Rules: BR-0706
Acceptance Criteria:
  Given   doanh nghiệp "ABC Corp" có 10 đánh giá với credibility_avg = 4.2
  When    organizer xem hồ sơ "ABC Corp"
  Then    hệ thống SHALL hiển thị "4.2/5 ⭐ (10 đánh giá)"
  And     hệ thống SHALL hiển thị 5 đánh giá gần nhất với nội dung nhận xét

  Given   doanh nghiệp mới chưa có đánh giá nào
  When    organizer xem hồ sơ
  Then    hệ thống SHALL hiển thị "Chưa có đánh giá"
Priority:      MUST
```

### FR-0705: Ngăn chặn đánh giá vi phạm

```
ID:            FR-0705
Name:          Kiểm duyệt nội dung đánh giá
Description:   Hệ thống SHALL kiểm duyệt nội dung nhận xét trước khi hiển thị công khai.
               Nội dung vi phạm (ngôn ngữ không phù hợp, spam, thông tin sai lệch)
               SHALL bị đánh dấu để admin review. Actor có thể báo cáo đánh giá vi phạm.
Classification: SYSTEM-SUPPORTED (auto-filter) + HUMAN (admin review)
Actor:         System (auto-filter), Admin (review), Any actor (báo cáo)
Trigger:       Đánh giá mới được gửi (auto-filter) hoặc actor báo cáo đánh giá vi phạm
Inputs:        review_id, review_comment (for auto-filter),
               report_reason (text, for flagging)
Outputs:       moderation_status (enum: APPROVED | FLAGGED | REMOVED),
               moderation_log entry
Business Rules: BR-0707
Acceptance Criteria:
  Given   organizer gửi đánh giá với comment chứa từ ngữ không phù hợp
  When    hệ thống kiểm duyệt tự động
  Then    hệ thống SHALL đánh dấu đánh giá là FLAGGED
  And     đánh giá SHALL không hiển thị công khai cho đến khi admin xử lý

  Given   sponsor thấy đánh giá sai sự thật về mình
  When    sponsor nhấn "Báo cáo đánh giá"
  Then    hệ thống SHALL tạo report và gửi cho admin xem xét
Priority:      SHOULD
```

---

## Business Rules

```
ID:          BR-0701
Rule:        Đánh giá chỉ có thể gửi khi hợp đồng đã kết thúc:
             validity_end < ngày hiện tại HOẶC tất cả nghĩa vụ đều CONFIRMED.
Source:      Quy trình gốc — Bước 5 (Sau khi kết thúc hợp đồng, hai bên đánh giá)
Type:        Validation

ID:          BR-0702
Rule:        Mỗi bên chỉ được gửi MỘT đánh giá cho mỗi hợp đồng.
             UNIQUE(contract_id, reviewer_id).
Source:      [INFERRED — ngăn đánh giá trùng lặp]
Type:        Validation

ID:          BR-0703
Rule:        Điểm đánh giá PHẢI nằm trong khoảng 1-5 (số nguyên).
             Nhận xét văn bản là TÙY CHỌN, tối đa 1000 ký tự. [ASSUMED]
Source:      Quy trình gốc — Bước 5 (đánh giá mức độ uy tín và chất lượng hợp tác)
Type:        Validation

ID:          BR-0704
Rule:        Thông báo nhắc đánh giá gửi qua in-app và email:
             lần 1 vào ngày kết thúc hợp đồng, lần 2 sau 7 ngày nếu chưa đánh giá.
             Tối đa 2 lần nhắc nhở. [ASSUMED]
Source:      [INFERRED — khuyến khích đánh giá nhưng không spam]
Type:        Time-based

ID:          BR-0705
Rule:        Điểm uy tín tổng hợp = trung bình cộng đơn giản của tất cả điểm đã nhận.
             Chỉ tính đánh giá có moderation_status = APPROVED.
             Cập nhật real-time khi có đánh giá mới. [ASSUMED — simple average cho phiên bản đầu]
Source:      [INFERRED — công thức tính điểm]
Type:        Calculation

ID:          BR-0706
Rule:        Trang hồ sơ công khai hiển thị TỐI ĐA 5 đánh giá gần nhất.
             Có thể xem thêm qua phân trang. Chỉ hiển thị đánh giá APPROVED.
Source:      [INFERRED — trải nghiệm giao diện]
Type:        Validation

ID:          BR-0707
Rule:        Đánh giá mới SHALL được kiểm duyệt tự động bằng bộ lọc từ khóa.
             Đánh giá FLAGGED cần admin review trong vòng 48 giờ. [ASSUMED]
             Đánh giá bị REMOVED SHALL không hiển thị và không tính vào điểm tổng hợp.
Source:      [INFERRED — bảo vệ chất lượng nội dung]
Type:        Validation + Time-based
```

---

## Data Model

```
Entity:        Review
Attributes:
  - review_id: UUID (PK)
  - contract_id: UUID (FK → Contract)
  - reviewer_id: UUID (FK → User)
  - reviewer_role: Enum [ORGANIZER, SPONSOR]
  - reviewee_entity_id: UUID (FK → Organization hoặc Business)
  - reviewee_entity_type: Enum [ORGANIZATION, BUSINESS]
  - credibility_rating: Integer (1-5, required)
  - collaboration_quality_rating: Integer (1-5, required)
  - review_comment: Text (nullable, max 1000 chars)
  - moderation_status: Enum [PENDING_REVIEW, APPROVED, FLAGGED, REMOVED]
                       (default: PENDING_REVIEW)
  - submitted_at: DateTime
  - moderated_at: DateTime (nullable)
  - moderated_by: UUID (FK → User/Admin, nullable)
Relationships:
  - Review —(N:1)→ Contract
  - Review —(N:1)→ User (reviewer)
Constraints:
  - UNIQUE(contract_id, reviewer_id) — mỗi bên chỉ đánh giá 1 lần mỗi hợp đồng

Entity:        ReputationScore (materialized / cached)
Attributes:
  - entity_id: UUID (PK, FK → Organization hoặc Business)
  - entity_type: Enum [ORGANIZATION, BUSINESS]
  - avg_credibility_score: Decimal (0.0-5.0)
  - avg_quality_score: Decimal (0.0-5.0)
  - total_review_count: Integer
  - last_updated_at: DateTime
Relationships:
  - ReputationScore —(1:1)→ Organization hoặc Business

Entity:        ReviewReport
Attributes:
  - report_id: UUID (PK)
  - review_id: UUID (FK → Review)
  - reported_by: UUID (FK → User)
  - report_reason: Text (required)
  - report_status: Enum [PENDING, RESOLVED_KEPT, RESOLVED_REMOVED]
  - reported_at: DateTime
  - resolved_at: DateTime (nullable)
  - resolved_by: UUID (FK → User/Admin, nullable)
Relationships:
  - ReviewReport —(N:1)→ Review
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 5. Đánh giá, phản hồi về uy tín và chất lượng | SYSTEM-SUPPORTED | FR-0701 | BR-0701, BR-0702, BR-0703 | Review |
| [INFERRED] Nhắc nhở đánh giá | FULLY AUTOMATED | FR-0702 | BR-0704 | Notification |
| [INFERRED] Tính điểm uy tín | FULLY AUTOMATED | FR-0703 | BR-0705 | ReputationScore |
| [INFERRED] Hiển thị điểm uy tín | SYSTEM-SUPPORTED | FR-0704 | BR-0706 | ReputationScore, Review |
| [INFERRED] Kiểm duyệt đánh giá | SYSTEM-SUPPORTED + HUMAN | FR-0705 | BR-0707 | Review, ReviewReport |
