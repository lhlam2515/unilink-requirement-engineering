# SF-02: Event & Partner Discovery

## Overview

```
Scope:            Cung cấp khả năng tìm kiếm và khám phá hai chiều — doanh nghiệp tìm kiếm sự kiện
                  phù hợp để tài trợ, và BTC tìm kiếm doanh nghiệp phù hợp để mời tài trợ.
System Boundary:
  IN:             Tìm kiếm, lọc, sắp xếp hồ sơ sự kiện và hồ sơ doanh nghiệp;
                  Xem chi tiết hồ sơ; Lưu/bookmark hồ sơ quan tâm.
  OUT:            Tạo hồ sơ tài trợ sự kiện (SF-01); Tạo hồ sơ doanh nghiệp (feature riêng, đã tồn tại);
                  Gửi lời mời tài trợ (SF-03).
Assumptions:
  - [ASSUMED] Hồ sơ doanh nghiệp (business profile) đã được quản lý bởi một feature riêng ngoài
    phạm vi quy trình này. SF-02 chỉ đọc dữ liệu từ hồ sơ doanh nghiệp.
  - [ASSUMED] Chỉ hồ sơ tài trợ ở trạng thái PUBLISHED mới xuất hiện trong kết quả tìm kiếm.
  - [ASSUMED] Hệ thống lập chỉ mục tìm kiếm toàn văn (full-text search) cho các trường văn bản.
Gaps Detected:
  - Quy trình gốc không nêu rõ tiêu chí sắp xếp kết quả tìm kiếm → cần bổ sung logic xếp hạng.
  - Không đề cập tính năng gợi ý/đề xuất tự động → flag dưới dạng automation opportunity.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Doanh nghiệp — Nhà tài trợ tiềm năng | `sponsor` | Tìm kiếm sự kiện, xem chi tiết hồ sơ tài trợ đã phát hành, bookmark sự kiện |
| Ban tổ chức (BTC) | `organizer` | Tìm kiếm doanh nghiệp, xem chi tiết hồ sơ doanh nghiệp, bookmark doanh nghiệp |
| Hệ thống | `system` | Lập chỉ mục, xếp hạng kết quả, ghi log tìm kiếm |

---

## Functional Requirements

### FR-0201: Tìm kiếm sự kiện theo tiêu chí

```
ID:            FR-0201
Name:          Tìm kiếm sự kiện phù hợp để tài trợ
Description:   Hệ thống SHALL cho phép actor có vai trò `sponsor` tìm kiếm các hồ sơ tài trợ sự kiện
               đã phát hành (PUBLISHED) dựa trên các bộ lọc: địa điểm tổ chức, quy mô sự kiện,
               đối tượng khán giả, ngân sách dự kiến, và hình thức tài trợ. Hệ thống SHALL trả về
               danh sách kết quả phân trang và sắp xếp theo mức độ phù hợp.
Classification: SYSTEM-SUPPORTED
Actor:         Sponsor (khởi tạo)
Trigger:       Sponsor nhập tiêu chí tìm kiếm và nhấn "Tìm kiếm" hoặc áp dụng bộ lọc
Inputs:        search_query (text, optional), filters: {
                 venue_location (string), scale_min (integer), scale_max (integer),
                 audience_keywords (string[]), budget_min (decimal), budget_max (decimal),
                 sponsorship_types (enum[]: CASH | IN_KIND | COMBINED)
               }, sort_by (enum: RELEVANCE | DATE | SCALE | BUDGET), page (integer), page_size (integer)
Outputs:       results[]: { proposal_id, event_name, event_type, thumbnail_url, venue,
               event_date_start, expected_scale, estimated_budget, accepted_sponsorship_types },
               total_count, current_page
Business Rules: BR-0201, BR-0202
Acceptance Criteria:
  Given   sponsor đã đăng nhập
  When    sponsor tìm kiếm với venue_location = "TP.HCM" và sponsorship_types = [CASH]
  Then    hệ thống SHALL trả về danh sách sự kiện tại TP.HCM chấp nhận tài trợ tiền mặt
  And     kết quả chỉ bao gồm hồ sơ có trạng thái PUBLISHED
  And     kết quả được phân trang theo page_size mặc định

  Given   không có sự kiện nào phù hợp với tiêu chí
  When    sponsor tìm kiếm
  Then    hệ thống SHALL hiển thị thông báo "Không tìm thấy sự kiện phù hợp"
Priority:      MUST
```

### FR-0202: Tìm kiếm doanh nghiệp theo tiêu chí

```
ID:            FR-0202
Name:          Tìm kiếm doanh nghiệp phù hợp để mời tài trợ
Description:   Hệ thống SHALL cho phép actor có vai trò `organizer` tìm kiếm các doanh nghiệp
               dựa trên các bộ lọc: khu vực hoạt động kinh doanh, đối tượng khách hàng,
               lĩnh vực hoạt động, ngân sách tài trợ sẵn sàng, và mục tiêu tài trợ (Marketing/CSR).
               Hệ thống SHALL trả về danh sách kết quả phân trang.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer (khởi tạo)
Trigger:       Organizer nhập tiêu chí tìm kiếm doanh nghiệp
Inputs:        search_query (text, optional), filters: {
                 business_region (string), customer_segment (string[]),
                 industry (string[]), sponsorship_budget_min (decimal),
                 sponsorship_budget_max (decimal),
                 sponsorship_goals (enum[]: MARKETING | CSR)
               }, sort_by (enum: RELEVANCE | BUDGET | INDUSTRY), page, page_size
Outputs:       results[]: { business_id, business_name, logo_url, industry, region,
               sponsorship_goals, budget_range }, total_count, current_page
Business Rules: BR-0201, BR-0203
Acceptance Criteria:
  Given   organizer đã đăng nhập
  When    organizer tìm kiếm doanh nghiệp với industry = "Công nghệ" và region = "Hà Nội"
  Then    hệ thống SHALL trả về danh sách doanh nghiệp công nghệ hoạt động tại Hà Nội
  And     kết quả chỉ bao gồm doanh nghiệp có hồ sơ hoạt động (active)

  Given   organizer nhập từ khóa tìm kiếm = "Samsung"
  When    hệ thống thực hiện tìm kiếm
  Then    hệ thống SHALL tìm kiếm toàn văn trên tên, mô tả, và lĩnh vực doanh nghiệp
Priority:      MUST
```

### FR-0203: Xem chi tiết hồ sơ tài trợ sự kiện

```
ID:            FR-0203
Name:          Hiển thị chi tiết hồ sơ tài trợ sự kiện
Description:   Hệ thống SHALL hiển thị toàn bộ thông tin chi tiết của một hồ sơ tài trợ sự kiện
               đã phát hành cho sponsor xem, bao gồm: thông tin cơ bản, thông tin chi tiết,
               hình thức tài trợ, danh sách gói tài trợ với quyền lợi, và hình ảnh nhận diện.
Classification: SYSTEM-SUPPORTED
Actor:         Sponsor (xem)
Trigger:       Sponsor nhấn vào một sự kiện trong danh sách kết quả tìm kiếm
Inputs:        proposal_id
Outputs:       Toàn bộ thông tin hồ sơ tài trợ (SponsorshipProposal + SponsorshipPackage[] + PackageBenefit[])
Business Rules: BR-0204
Acceptance Criteria:
  Given   hồ sơ tài trợ proposal_id = "abc-123" đang ở trạng thái PUBLISHED
  When    sponsor nhấn xem chi tiết
  Then    hệ thống SHALL hiển thị đầy đủ thông tin sự kiện, gói tài trợ và quyền lợi

  Given   hồ sơ tài trợ ở trạng thái DRAFT
  When    sponsor cố truy cập bằng URL trực tiếp
  Then    hệ thống SHALL trả về lỗi 404 hoặc thông báo "Hồ sơ không tồn tại hoặc chưa được phát hành"
Priority:      MUST
```

### FR-0204: Xem chi tiết hồ sơ doanh nghiệp

```
ID:            FR-0204
Name:          Hiển thị chi tiết hồ sơ doanh nghiệp
Description:   Hệ thống SHALL hiển thị thông tin chi tiết doanh nghiệp cho organizer xem,
               bao gồm: tên doanh nghiệp, logo, lĩnh vực hoạt động, khu vực, đối tượng khách hàng,
               mục tiêu tài trợ, và ngân sách tài trợ dự kiến.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer (xem)
Trigger:       Organizer nhấn vào một doanh nghiệp trong danh sách kết quả tìm kiếm
Inputs:        business_id
Outputs:       Thông tin chi tiết doanh nghiệp (BusinessProfile)
Business Rules: BR-0204
Acceptance Criteria:
  Given   doanh nghiệp business_id = "biz-456" có hồ sơ active
  When    organizer nhấn xem chi tiết
  Then    hệ thống SHALL hiển thị đầy đủ thông tin doanh nghiệp
Priority:      MUST
```

### FR-0205: Bookmark hồ sơ quan tâm

```
ID:            FR-0205
Name:          Lưu hồ sơ vào danh sách quan tâm
Description:   Hệ thống SHALL cho phép sponsor bookmark hồ sơ tài trợ sự kiện, và organizer
               bookmark hồ sơ doanh nghiệp. Hệ thống SHALL cho phép xem danh sách đã bookmark
               và xóa bookmark.
Classification: SYSTEM-SUPPORTED
Actor:         Sponsor, Organizer
Trigger:       Actor nhấn nút "Bookmark" / "Lưu" trên trang chi tiết hoặc kết quả tìm kiếm
Inputs:        actor_id, target_type (enum: PROPOSAL | BUSINESS), target_id (UUID)
Outputs:       bookmark_id, bookmarked_at (timestamp)
Business Rules: BR-0205
Acceptance Criteria:
  Given   sponsor đang xem chi tiết hồ sơ tài trợ
  When    sponsor nhấn "Bookmark"
  Then    hệ thống SHALL lưu bookmark và hiển thị trạng thái "Đã lưu"

  Given   sponsor đã bookmark hồ sơ trước đó
  When    sponsor nhấn "Bỏ lưu"
  Then    hệ thống SHALL xóa bookmark

  Given   sponsor xem danh sách bookmark
  When    hồ sơ đã bookmark bị hủy phát hành
  Then    hệ thống SHALL hiển thị trạng thái "Hồ sơ không còn khả dụng"
Priority:      SHOULD
```

---

## Business Rules

```
ID:          BR-0201
Rule:        Kết quả tìm kiếm chỉ được bao gồm các hồ sơ đang ở trạng thái PUBLISHED
             (đối với sự kiện) hoặc ACTIVE (đối với doanh nghiệp).
Source:      Quy trình gốc — Bước 1 (Sau khi BTC tạo thành công hồ sơ... doanh nghiệp có thể tìm kiếm)
Type:        Validation

ID:          BR-0202
Rule:        Bộ lọc tìm kiếm sự kiện bao gồm: địa điểm tổ chức, quy mô sự kiện (range),
             đối tượng khán giả (keywords), ngân sách (range), hình thức tài trợ (multi-select).
             Các bộ lọc có thể kết hợp (AND logic).
Source:      Quy trình gốc — Bước 2.1 (Doanh nghiệp chủ động tìm kiếm)
Type:        Validation

ID:          BR-0203
Rule:        Bộ lọc tìm kiếm doanh nghiệp bao gồm: khu vực hoạt động kinh doanh,
             đối tượng khách hàng, lĩnh vực hoạt động, ngân sách tài trợ (range),
             mục tiêu tài trợ (MARKETING | CSR). Các bộ lọc có thể kết hợp (AND logic).
Source:      Quy trình gốc — Bước 2.2 (BTC chủ động tìm kiếm)
Type:        Validation

ID:          BR-0204
Rule:        Actor chỉ có thể xem chi tiết hồ sơ đã phát hành/active.
             Truy cập hồ sơ DRAFT hoặc INACTIVE bằng đường dẫn trực tiếp SHALL bị từ chối.
Source:      [INFERRED — bảo vệ dữ liệu chưa sẵn sàng]
Type:        Authorization

ID:          BR-0205
Rule:        Mỗi actor chỉ có thể bookmark một hồ sơ MỘT LẦN (không trùng lặp).
             Bookmark hồ sơ bị hủy phát hành SHALL được giữ lại nhưng đánh dấu "không khả dụng".
Source:      [INFERRED — trải nghiệm người dùng]
Type:        Validation
```

---

## Data Model

```
Entity:        Bookmark
Attributes:
  - bookmark_id: UUID (PK)
  - actor_id: UUID (FK → User)
  - target_type: Enum [PROPOSAL, BUSINESS]
  - target_id: UUID (FK → SponsorshipProposal | BusinessProfile)
  - bookmarked_at: DateTime
Relationships:
  - Bookmark —(N:1)→ User
  - Bookmark —(N:1)→ SponsorshipProposal (when target_type = PROPOSAL)
  - Bookmark —(N:1)→ BusinessProfile (when target_type = BUSINESS)
Constraints:
  - UNIQUE(actor_id, target_type, target_id) — ngăn bookmark trùng lặp
```

> **Note:** Entity `BusinessProfile` thuộc feature riêng ngoài phạm vi quy trình này.
> SF-02 đọc dữ liệu từ entity này nhưng không quản lý vòng đời của nó.

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 2.1 DN chủ động tìm kiếm sự kiện | SYSTEM-SUPPORTED | FR-0201 | BR-0201, BR-0202 | SponsorshipProposal |
| 2.1 DN xem chi tiết sự kiện | SYSTEM-SUPPORTED | FR-0203 | BR-0204 | SponsorshipProposal, SponsorshipPackage, PackageBenefit |
| 2.2 BTC chủ động tìm kiếm DN | SYSTEM-SUPPORTED | FR-0202 | BR-0201, BR-0203 | BusinessProfile |
| 2.2 BTC xem chi tiết hồ sơ DN | SYSTEM-SUPPORTED | FR-0204 | BR-0204 | BusinessProfile |
| [INFERRED] Lưu hồ sơ quan tâm | SYSTEM-SUPPORTED | FR-0205 | BR-0205 | Bookmark |

---

## Automation Opportunities

```
⚡ AUTOMATION OPPORTUNITY: Đề xuất tự động (Recommendation Engine)
   Thay vì sponsor/organizer phải tìm kiếm thủ công, hệ thống có thể tự động gợi ý
   sự kiện/doanh nghiệp phù hợp dựa trên hồ sơ, lịch sử tài trợ, và hành vi tìm kiếm.
   Classification: FULLY AUTOMATED
   Rationale: Tăng tỷ lệ kết nối thành công, giảm thời gian tìm kiếm.
   Priority: COULD (phase 2)
```
