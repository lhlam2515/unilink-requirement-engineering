# SF-11: Public Organization Profile & Sponsorship History

## Overview

```
Scope:            Cung cấp hồ sơ tổ chức công khai cho đối tác tra cứu, bao gồm thông tin nhận
                  diện tổ chức, trạng thái xác thực, tóm tắt uy tín, lịch sử hồ sơ tài trợ công
                  khai của CLB/BTC, và lịch sử giao dịch tài trợ công khai của doanh nghiệp.
System Boundary:
  IN:             Xem hồ sơ tổ chức công khai; xem tóm tắt uy tín (read-only trên public profile);
                  xem lịch sử hồ sơ tài trợ công khai; xem lịch sử giao dịch tài trợ công khai;
                  lọc/phân trang danh sách lịch sử.
  OUT:            Chỉnh sửa hồ sơ tổ chức (SF-09); kiểm duyệt/xác thực hồ sơ (SF-10);
                  tạo hồ sơ tài trợ (SF-01); tìm kiếm/khám phá đối tác (SF-02);
                  gửi lời mời, thương thảo, hợp đồng, nghĩa vụ, đánh giá chi tiết;
                  xem chi tiết uy tín (SF-07 — chỉ Authenticated User, ngoài phạm vi SF-11).
Assumptions:
  - [CONFIRMED] Chỉ tổ chức có verification_status = VERIFIED mới có hồ sơ công khai.
    Tổ chức UNVERIFIED, PENDING_REVIEW, REJECTED, INFO_REQUIRED KHÔNG có public profile.
  - [CONFIRMED] Mỗi tổ chức chỉ có đúng MỘT vai trò duy nhất (ORGANIZER hoặc SPONSOR),
    không thể có cả hai vai trò.
  - [CONFIRMED] Hồ sơ công khai chỉ hiển thị dữ liệu đã được biên tập để công khai, không lộ
    draft, nội dung thương thảo, điều khoản hợp đồng, hoặc thông tin nhạy cảm.
  - [CONFIRMED] Chỉ các hồ sơ/giao dịch có trạng thái đủ điều kiện công khai mới xuất hiện
    trong danh sách lịch sử.
  - [CONFIRMED] Màn public profile là read-only; không có hành động chỉnh sửa từ phía visitor.
  - [UPDATED] Hồ sơ công khai chỉ dành cho Authenticated User — Guest KHÔNG xem được.
    Authenticated User có thể xem tóm tắt uy tín và điều hướng sang SCR-018 (UC-30).
  - [CONFIRMED] Danh sách lịch sử hiển thị tối đa 5 mục gần nhất; không cho phép cấu hình
    page size.
  - [CONFIRMED] Lịch sử public chỉ hiển thị dạng danh sách; mỗi mục có thể liên kết sang
    view chi tiết sự kiện hoặc hồ sơ doanh nghiệp tài trợ tương ứng (nếu có).
Gaps Detected:
  - [RESOLVED] Quy tắc public visibility → chỉ VERIFIED (BR-1101).
  - [RESOLVED] Mức độ ẩn/hiển thị → BR-1106 quy định rõ.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Tài khoản đã đăng nhập (Organizer hoặc Sponsor) | `authenticated user` | Xem hồ sơ tổ chức công khai, xem lịch sử công khai, xem tóm tắt uy tín, điều hướng sang chi tiết uy tín (UC-30/SCR-018) |
| Hệ thống | `system` | Truy xuất, lọc, phân trang, và biên tập dữ liệu công khai |

---

## Functional Requirements

### FR-1101: Xem hồ sơ tổ chức công khai

```
ID:            FR-1101
Name:          Xem hồ sơ tổ chức công khai
Description:   Hệ thống SHALL hiển thị hồ sơ công khai của một tổ chức, bao gồm tên tổ
               chức, logo hoặc ảnh đại diện, loại tổ chức, vai trò chính (Organizer hoặc
               Sponsor), trạng thái xác thực, khu vực hoạt động, và mô tả ngắn.
               Hệ thống SHALL chỉ hiển thị các tổ chức đủ điều kiện public theo BR-1101.
Classification: SYSTEM-SUPPORTED
Actor:         Authenticated User
Trigger:       Người dùng đã đăng nhập truy cập đường dẫn public profile hoặc chọn tổ chức từ một liên kết
               nội bộ trong hệ thống.
Inputs:        organization_id
Outputs:       PublicOrganizationProfileView
Business Rules: BR-1101, BR-1102
Acceptance Criteria:
  Given   tổ chức có hồ sơ public hợp lệ
  When    người dùng mở trang public profile
  Then    hệ thống SHALL hiển thị thông tin nhận diện và trạng thái xác thực

  Given   tổ chức không tồn tại hoặc chưa đủ điều kiện public
  When    người dùng truy cập đường dẫn
  Then    hệ thống SHALL trả về trang không tồn tại hoặc thông báo không khả dụng
Priority:      MUST
```

### FR-1102: Xem lịch sử hồ sơ tài trợ công khai của CLB/BTC

```
ID:            FR-1102
Name:          Xem lịch sử hồ sơ tài trợ công khai của tổ chức tổ chức sự kiện
Description:   Hệ thống SHALL hiển thị lịch sử công khai các hồ sơ tài trợ đã từng thực hiện
               của một tổ chức theo vai trò Organizer. Danh sách này chỉ bao gồm các hồ sơ
               đã đủ điều kiện công khai, ví dụ: PUBLISHED, đã kết thúc, hoặc được lưu trữ
               công khai theo chính sách. Mỗi mục lịch sử SHALL hiển thị dữ liệu tóm tắt về
               sự kiện, thời gian, trạng thái hợp tác, và số liệu tổng quan được phép công khai.
Classification: SYSTEM-SUPPORTED
Actor:         Authenticated User
Trigger:       Người dùng chọn tab hoặc section "Lịch sử tài trợ" trên public profile của tổ chức
               có vai trò Organizer.
Inputs:        organization_id, filter_year (optional), filter_status (optional), page
Outputs:       PublicSponsorshipHistoryItem[]
Business Rules: BR-1102, BR-1103, BR-1104
Acceptance Criteria:
  Given   tổ chức là Organizer và có ít nhất một hồ sơ tài trợ công khai
  When    người dùng mở tab "Lịch sử tài trợ"
  Then    hệ thống SHALL hiển thị danh sách lịch sử theo thứ tự mới nhất trước

  Given   tổ chức chưa có hồ sơ công khai nào
  When    người dùng mở tab "Lịch sử tài trợ"
  Then    hệ thống SHALL hiển thị empty state phù hợp
Priority:      MUST
```

### FR-1103: Xem lịch sử giao dịch tài trợ công khai của doanh nghiệp

```
ID:            FR-1103
Name:          Xem lịch sử giao dịch tài trợ công khai của tổ chức tài trợ
Description:   Hệ thống SHALL hiển thị lịch sử công khai các giao dịch tài trợ đã hoàn tất của
               một tổ chức theo vai trò Sponsor. Danh sách này chỉ bao gồm các giao dịch đủ
               điều kiện công khai và SHALL ẩn các thông tin thương thảo nội bộ, điều khoản hợp
               đồng, và dữ liệu nhạy cảm. Mỗi mục SHALL hiển thị thông tin tóm tắt về sự kiện,
               thời gian, hình thức tài trợ, và trạng thái hoàn tất.
Classification: SYSTEM-SUPPORTED
Actor:         Authenticated User
Trigger:       Người dùng chọn tab hoặc section "Lịch sử giao dịch" trên public profile của tổ chức
               có vai trò Sponsor.
Inputs:        organization_id, filter_year (optional), filter_type (optional), page
Outputs:       PublicTransactionHistoryItem[]
Business Rules: BR-1102, BR-1103, BR-1104
Acceptance Criteria:
  Given   tổ chức là Sponsor và có các giao dịch tài trợ công khai
  When    người dùng mở tab "Lịch sử giao dịch"
  Then    hệ thống SHALL hiển thị danh sách giao dịch theo thứ tự mới nhất trước

  Given   tổ chức chưa có giao dịch công khai nào
  When    người dùng mở tab "Lịch sử giao dịch"
  Then    hệ thống SHALL hiển thị empty state phù hợp
Priority:      MUST
```

### FR-1104: Xem tóm tắt uy tín và điều hướng sang chi tiết

```
ID:            FR-1104
Name:          Xem tóm tắt uy tín và điều hướng đến màn uy tín chi tiết
Description:   Hệ thống SHALL hiển thị một tóm tắt uy tín ngắn trên public profile để hỗ trợ
               người dùng đánh giá nhanh. Tóm tắt bao gồm: điểm uy tín trung bình, điểm chất
               lượng hợp tác trung bình, và tổng số đánh giá.
               Hệ thống SHALL cung cấp liên kết điều hướng sang màn chi tiết uy tín đánh giá
               công khai (UC-30 / SCR-018) để Authenticated User xem sâu hơn.
Classification: SYSTEM-SUPPORTED
Actor:         Authenticated User
Trigger:       Người dùng mở public profile
Inputs:        organization_id
Outputs:       ReputationSummaryView + liên kết đến SCR-018
Business Rules: BR-1105
Acceptance Criteria:
  Given   public profile đang hiển thị và tổ chức có đánh giá
  When    hệ thống nạp dữ liệu
  Then    hệ thống SHALL hiển thị tóm tắt uy tín (điểm trung bình, tổng đánh giá)
  And     hệ thống SHALL hiển thị liên kết "Xem chi tiết uy tín" đến SCR-018

  Given   tổ chức chưa có đánh giá nào
  When    hệ thống nạp dữ liệu
  Then    hệ thống SHALL hiển thị "Chưa có đánh giá"
Priority:      SHOULD
```

---

## Business Rules

```
ID:          BR-1101
Rule:        Chỉ tổ chức có verification_status = VERIFIED mới được hiển thị hồ sơ công khai.
             Tổ chức ở trạng thái UNVERIFIED, PENDING_REVIEW, REJECTED, hoặc INFO_REQUIRED
             KHÔNG có public profile. Truy cập đường dẫn public profile của tổ chức chưa
             VERIFIED SHALL trả về trang không tồn tại hoặc thông báo không khả dụng.
Source:      Nhu cầu bổ sung tính năng public profile — xác nhận chỉ VERIFIED
Type:        Visibility / Authorization

ID:          BR-1102
Rule:        Chỉ dữ liệu đã được biên tập cho public mới được hiển thị.
             Không hiển thị draft, nội dung thương thảo, điều khoản hợp đồng, hoặc thông tin
             cá nhân nhạy cảm. Dữ liệu tự động đủ điều kiện public khi trạng thái hồ sơ/giao
             dịch đạt trạng thái phù hợp (COMPLETED, ARCHIVED, PUBLISHED) — không yêu cầu
             biên tập thủ công.
Source:      Nhu cầu bảo vệ dữ liệu nhạy cảm
Type:        Data protection

ID:          BR-1103
Rule:        Danh sách lịch sử public hiển thị tối đa 5 mục gần nhất, sắp xếp theo thời gian
             giảm dần. Page size CỐ ĐỊNH, không cho phép người dùng cấu hình.
             Hỗ trợ phân trang (load more) khi có nhiều hơn 5 mục.
Source:      Nhu cầu khám phá lịch sử — xác nhận tối đa 5 mục
Type:        UI / List behavior

ID:          BR-1104
Rule:        Mỗi tổ chức chỉ có DUY NHẤT một vai trò (ORGANIZER hoặc SPONSOR). Loại lịch sử
             hiển thị tương ứng với vai trò:
             - Organizer: lịch sử hồ sơ tài trợ công khai.
             - Sponsor: lịch sử giao dịch tài trợ công khai.
             Không có trường hợp hiển thị cả hai loại cùng lúc.
Source:      Nhu cầu nghiệp vụ theo vai trò — xác nhận mỗi tổ chức một vai trò duy nhất
Type:        Content selection

ID:          BR-1105
Rule:        Public profile SHALL hiển thị tóm tắt uy tín gồm: điểm uy tín trung bình, điểm
             chất lượng hợp tác trung bình, và tổng số đánh giá. Vì chỉ Authenticated User
             mới có quyền xem public profile, hệ thống SHALL cung cấp liên kết điều hướng
             sang SCR-018 (UC-30) để xem chi tiết đánh giá công khai.
Source:      Nhu cầu liên kết giữa hồ sơ public và reputation
Type:        Navigation

ID:          BR-1106
Rule:        Lịch sử public KHÔNG hiển thị các thông tin nhạy cảm sau:
             - Giá trị tài trợ (số tiền, giá trị hiện vật cụ thể).
             - Điều khoản hợp đồng.
             - Số lượng deal nội bộ.
             Lịch sử public CÓ hiển thị:
             - Tên sự kiện liên quan.
             - Năm thực hiện.
             - Hình thức tài trợ (CASH / IN_KIND / MIXED) — chỉ cho Sponsor history.
             - Trạng thái tổng quan (COMPLETED / ARCHIVED).
             - Nhãn tóm tắt (mô tả ngắn do hệ thống tạo).
             Mỗi mục lịch sử CÓ THỂ liên kết sang view chi tiết sự kiện hoặc hồ sơ doanh
             nghiệp tài trợ tương ứng (nếu tồn tại và đủ điều kiện public).
Source:      Nhu cầu xác định rõ mức độ ẩn/hiện dữ liệu — gap đã giải quyết
Type:        Data protection / Display
```

---

## Data Model

```
Entity:        OrganizationPublicProfileView
Description:   View công khai của hồ sơ tổ chức. Chỉ tồn tại khi verification_status = VERIFIED.
Attributes:
  - organization_id: UUID (PK)
  - organization_name: string
  - role: enum (ORGANIZER | SPONSOR) — duy nhất một giá trị cho mỗi tổ chức
  - logo_url: string
  - verification_status: enum (luôn = VERIFIED trên public profile)
  - short_description: string
  - region: string
  - reputation_summary: ReputationSummaryView (embedded, dẫn xuất từ SF-07.ReputationScore)
Note:          Entity này là read-only view, không phải bảng gốc. Dữ liệu được dẫn xuất từ
               Organization (SF-09) và ReputationScore (SF-07).

Entity:        PublicSponsorshipHistoryItem
Description:   Mục lịch sử hồ sơ tài trợ công khai — hiển thị cho tổ chức vai trò Organizer.
Attributes:
  - history_item_id: UUID (PK)
  - organization_id: UUID (FK → Organization)
  - event_name: string
  - public_status: enum (PUBLISHED | COMPLETED | ARCHIVED)
  - year: integer
  - summary_label: string — nhãn tóm tắt ngắn do hệ thống tự tạo
  - published_at: datetime
  - archived_at: datetime (nullable)
  - event_link_id: UUID (nullable, FK → SponsorshipProposal — liên kết xem chi tiết nếu có)
Note:          Không hiển thị giá trị tài trợ, điều khoản hợp đồng (BR-1106).

Entity:        PublicTransactionHistoryItem
Description:   Mục lịch sử giao dịch tài trợ công khai — hiển thị cho tổ chức vai trò Sponsor.
Attributes:
  - history_item_id: UUID (PK)
  - organization_id: UUID (FK → Organization)
  - event_name: string
  - sponsorship_type: enum (CASH | IN_KIND | MIXED)
  - completion_status: enum (COMPLETED | ARCHIVED)
  - year: integer
  - summary_label: string — nhãn tóm tắt ngắn do hệ thống tự tạo
  - completed_at: datetime
  - organizer_link_id: UUID (nullable, FK → Organization — liên kết xem hồ sơ BTC nếu có)
Note:          Không hiển thị giá trị tiền, điều khoản hợp đồng (BR-1106).

Entity:        ReputationSummaryView
Description:   View read-only tóm tắt uy tín, dẫn xuất từ SF-07.ReputationScore.
               Không phải bảng riêng — là projection/cache từ dữ liệu gốc.
Attributes:
  - organization_id: UUID (FK → Organization, FK → SF-07.ReputationScore)
  - average_reputation_score: decimal (0.0–5.0)
  - average_quality_score: decimal (0.0–5.0)
  - total_reviews: integer
Relationships:
  - ReputationSummaryView ← dẫn xuất từ SF-07.ReputationScore (1:1)
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| Xem hồ sơ tổ chức công khai | SYSTEM-SUPPORTED | FR-1101 | BR-1101, BR-1102 | OrganizationPublicProfileView |
| Xem lịch sử hồ sơ tài trợ công khai (Organizer) | SYSTEM-SUPPORTED | FR-1102 | BR-1102, BR-1103, BR-1104, BR-1106 | PublicSponsorshipHistoryItem |
| Xem lịch sử giao dịch tài trợ công khai (Sponsor) | SYSTEM-SUPPORTED | FR-1103 | BR-1102, BR-1103, BR-1104, BR-1106 | PublicTransactionHistoryItem |
| Xem tóm tắt uy tín trên public profile | SYSTEM-SUPPORTED | FR-1104 | BR-1105 | ReputationSummaryView |
