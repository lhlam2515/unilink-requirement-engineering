# SF-01: Sponsorship Proposal Management

## Overview

```
Scope:            Quản lý toàn bộ vòng đời của hồ sơ tài trợ sự kiện — từ khởi tạo, soạn thảo nội dung,
                  định nghĩa gói tài trợ đến phát hành hồ sơ để doanh nghiệp có thể tìm kiếm và tiếp cận.
System Boundary:
  IN:             Tạo, chỉnh sửa, xác thực, phát hành và quản lý trạng thái hồ sơ tài trợ;
                  Quản lý media (banner/thumbnail); Định nghĩa gói tài trợ và quyền lợi nhà tài trợ.
  OUT:            Quản lý tài khoản tổ chức (tồn tại sẵn); Tìm kiếm và khám phá sự kiện (SF-02);
                  Quy trình gửi lời mời tài trợ (SF-03).
Assumptions:
  - [ASSUMED] BTC đã được xác thực và có tài khoản hợp lệ trên hệ thống trước khi tạo hồ sơ.
  - [ASSUMED] Mỗi tổ chức chỉ có một tài khoản đại diện trên nền tảng; hệ thống không quản lý
    thành viên nội bộ, tài khoản con, hoặc phân quyền theo nhiều người dùng trong cùng tổ chức.
  - [ASSUMED] Một tài khoản BTC có thể sở hữu nhiều hồ sơ tài trợ cho các sự kiện khác nhau đồng thời.
  - [ASSUMED] Hệ thống hỗ trợ upload ảnh với các định dạng phổ biến (JPEG, PNG, WebP) và giới hạn dung lượng.
  - [ASSUMED] Hồ sơ tài trợ cần đạt trạng thái "Đã phát hành" để hiển thị cho doanh nghiệp tìm kiếm.
Gaps Detected:
  - Cần làm rõ hồ sơ đã phát hành chỉ được hủy phát hành khi chưa bắt đầu thương thảo
    với bất kỳ lời mời tài trợ nào.
  - Cần làm rõ quyền sở hữu hồ sơ thuộc về tài khoản tổ chức đại diện, không phải tập hợp tài khoản thành viên.
```

---

## Actor Mapping

| Business Actor | System Role | Permissions / Access Level |
|---|---|---|
| Tài khoản đại diện của Ban tổ chức (BTC) | `organizer` | Tạo, chỉnh sửa, xóa bản nháp, phát hành, hủy phát hành nhiều hồ sơ tài trợ cho các sự kiện khác nhau |
| Tài khoản đại diện của doanh nghiệp | `sponsor` | Xem hồ sơ tài trợ đã phát hành (chỉ đọc) |
| Hệ thống | `system` | Xác thực dữ liệu, chuyển trạng thái, ghi log, lập chỉ mục tìm kiếm |

---

## Functional Requirements

### FR-0101: Tạo hồ sơ tài trợ sự kiện

```
ID:            FR-0101
Name:          Khởi tạo hồ sơ tài trợ sự kiện
Description:   Hệ thống SHALL cho phép actor có vai trò `organizer` khởi tạo một hồ sơ tài trợ
               sự kiện mới với trạng thái ban đầu là DRAFT. Hệ thống SHALL gán mã định danh duy nhất
               và ghi nhận thời gian tạo cùng danh tính người tạo.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer (khởi tạo)
Trigger:       Organizer chọn chức năng "Tạo hồ sơ tài trợ mới"
Inputs:        Không bắt buộc tại bước khởi tạo (cho phép tạo bản nháp trống)
Outputs:       proposal_id (UUID), status = DRAFT, created_at (timestamp), created_by (actor_id)
Business Rules: BR-0101, BR-0102
Acceptance Criteria:
  Given   organizer đã đăng nhập với vai trò `organizer`
  When    organizer chọn "Tạo hồ sơ tài trợ mới"
  Then    hệ thống SHALL tạo hồ sơ mới với trạng thái DRAFT
  And     hệ thống SHALL gán proposal_id duy nhất
  And     hệ thống SHALL ghi nhận created_at và created_by
  And     hệ thống SHALL chuyển organizer đến trang chỉnh sửa hồ sơ
Priority:      MUST
```

### FR-0102: Nhập thông tin cơ bản của sự kiện

```
ID:            FR-0102
Name:          Cập nhật thông tin cơ bản sự kiện
Description:   Hệ thống SHALL cho phép organizer nhập và chỉnh sửa thông tin cơ bản của sự kiện
               bao gồm: tên chương trình, loại hình sự kiện (cuộc thi, workshop, đêm nhạc, v.v.),
               thời gian tổ chức, và địa điểm tổ chức. Hệ thống SHALL xác thực từng trường
               theo BR-0103 trước khi lưu.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer
Trigger:       Organizer chỉnh sửa mục "Thông tin cơ bản" trong hồ sơ tài trợ
Inputs:        event_name (string), event_type (enum), event_date_start (datetime),
               event_date_end (datetime), venue (string)
Outputs:       Hồ sơ đã cập nhật với thông tin cơ bản, updated_at (timestamp)
Business Rules: BR-0103
Acceptance Criteria:
  Given   hồ sơ tài trợ đang ở trạng thái DRAFT hoặc PUBLISHED
  When    organizer nhập tên chương trình = "UniHack 2026" và chọn loại hình = "Cuộc thi"
  Then    hệ thống SHALL lưu thông tin và cập nhật updated_at
  And     nếu hồ sơ đang PUBLISHED, hệ thống SHALL cập nhật thông tin trên trang tìm kiếm

  Given   organizer nhập event_date_start nằm trong quá khứ
  When    organizer lưu thông tin
  Then    hệ thống SHALL từ chối với thông báo lỗi "Thời gian tổ chức phải trong tương lai"
Priority:      MUST
```

### FR-0103: Tải lên hình ảnh nhận diện sự kiện

```
ID:            FR-0103
Name:          Quản lý hình ảnh nhận diện sự kiện
Description:   Hệ thống SHALL cho phép organizer tải lên hình ảnh nhận diện sự kiện (banner,
               thumbnail). Hệ thống SHALL xác thực định dạng file (JPEG, PNG, WebP) và kích thước
               tối đa theo BR-0104. Hệ thống SHALL tạo các phiên bản thumbnail tự động.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer
Trigger:       Organizer tải lên file hình ảnh vào mục "Hình ảnh nhận diện"
Inputs:        image_file (binary), image_type (enum: BANNER | THUMBNAIL)
Outputs:       image_url, thumbnail_url, upload_status
Business Rules: BR-0104
Acceptance Criteria:
  Given   organizer đang chỉnh sửa hồ sơ tài trợ
  When    organizer tải lên file PNG kích thước 2MB với loại BANNER
  Then    hệ thống SHALL lưu trữ ảnh gốc và tạo thumbnail
  And     hệ thống SHALL trả về image_url và thumbnail_url

  Given   organizer tải lên file PDF
  When    hệ thống kiểm tra định dạng
  Then    hệ thống SHALL từ chối với thông báo "Chỉ chấp nhận JPEG, PNG, WebP"
Priority:      SHOULD
```

### FR-0104: Nhập thông tin chi tiết sự kiện

```
ID:            FR-0104
Name:          Cập nhật thông tin chi tiết sự kiện
Description:   Hệ thống SHALL cho phép organizer nhập thông tin chi tiết bao gồm: quy mô sự kiện
               (số lượng khán giả dự kiến), ngân sách dự kiến, đối tượng khán giả mục tiêu, và
               nội dung chương trình chi tiết. Hệ thống SHALL xác thực giá trị số (quy mô, ngân sách)
               là số dương.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer
Trigger:       Organizer chỉnh sửa mục "Thông tin chi tiết" trong hồ sơ tài trợ
Inputs:        expected_scale (integer), estimated_budget (decimal + currency),
               target_audience (text), program_content (rich text)
Outputs:       Hồ sơ đã cập nhật, updated_at
Business Rules: BR-0103
Acceptance Criteria:
  Given   hồ sơ tài trợ đang ở trạng thái DRAFT
  When    organizer nhập quy mô = 500, ngân sách = 50,000,000 VND, đối tượng = "Sinh viên IT năm 3-4"
  Then    hệ thống SHALL lưu thông tin chi tiết thành công

  Given   organizer nhập ngân sách = -100,000
  When    organizer lưu thông tin
  Then    hệ thống SHALL từ chối với thông báo "Ngân sách phải là số dương"
Priority:      MUST
```

### FR-0105: Định nghĩa hình thức tài trợ

```
ID:            FR-0105
Name:          Thiết lập hình thức tài trợ được chấp nhận
Description:   Hệ thống SHALL cho phép organizer chỉ định các hình thức tài trợ mà BTC chấp nhận:
               tiền mặt (CASH), hiện vật (IN_KIND), hoặc kết hợp (COMBINED). Organizer có thể
               chọn một hoặc nhiều hình thức.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer
Trigger:       Organizer chỉnh sửa mục "Hình thức tài trợ"
Inputs:        accepted_sponsorship_types (enum[]: CASH | IN_KIND | COMBINED)
Outputs:       Hồ sơ đã cập nhật danh sách hình thức tài trợ
Business Rules: BR-0105
Acceptance Criteria:
  Given   hồ sơ tài trợ đang ở trạng thái DRAFT
  When    organizer chọn hình thức "Tiền mặt" và "Hiện vật"
  Then    hệ thống SHALL lưu accepted_sponsorship_types = [CASH, IN_KIND]

  Given   organizer không chọn hình thức nào
  When    organizer cố gắng phát hành hồ sơ
  Then    hệ thống SHALL từ chối phát hành với thông báo "Cần chọn ít nhất một hình thức tài trợ"
Priority:      MUST
```

### FR-0106: Tạo và quản lý gói tài trợ

```
ID:            FR-0106
Name:          Tạo và quản lý các gói tài trợ
Description:   Hệ thống SHALL cho phép organizer tạo nhiều gói tài trợ (sponsorship packages)
               với các cấp độ khác nhau: Nhà tài trợ chính (Title Sponsor), Nhà tài trợ đồng hành
               (Co-Sponsor), Nhà tài trợ liên kết (Associate Sponsor), Nhà tài trợ kỹ thuật/hạ tầng
               (Technical Sponsor), Đối tác tài trợ (Sponsorship Partner). Mỗi gói bao gồm:
               tên gói, mô tả, giá trị tài trợ tối thiểu, số lượng slot khả dụng, và danh sách
               quyền lợi tương ứng.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer
Trigger:       Organizer chọn "Thêm gói tài trợ" hoặc chỉnh sửa gói hiện có
Inputs:        package_name (string), package_tier (enum), description (text),
               min_value (decimal), available_slots (integer), benefits[] (xem FR-0107)
Outputs:       package_id (UUID), hồ sơ đã cập nhật với gói tài trợ mới
Business Rules: BR-0106
Acceptance Criteria:
  Given   hồ sơ tài trợ đang ở trạng thái DRAFT
  When    organizer tạo gói "Nhà tài trợ chính" với giá trị tối thiểu 20,000,000 VND, 1 slot
  Then    hệ thống SHALL tạo gói tài trợ với package_id duy nhất
  And     hệ thống SHALL hiển thị gói trong danh sách gói tài trợ của hồ sơ

  Given   organizer tạo gói với available_slots = 0
  When    organizer lưu gói
  Then    hệ thống SHALL từ chối với thông báo "Số lượng slot phải ít nhất là 1"
Priority:      MUST
```

### FR-0107: Định nghĩa quyền lợi nhà tài trợ

```
ID:            FR-0107
Name:          Thiết lập quyền lợi cho từng gói tài trợ
Description:   Hệ thống SHALL cho phép organizer gắn danh sách quyền lợi (benefits) cho mỗi gói
               tài trợ. Các nhóm quyền lợi bao gồm: Xây dựng thương hiệu & tăng khả năng hiển thị
               (BRANDING), Sân khấu & thông báo (STAGE), Quảng bá kỹ thuật số & online (DIGITAL),
               Tương tác với khán giả (ENGAGEMENT). Mỗi quyền lợi có mô tả cụ thể về cam kết
               thực hiện của BTC.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer
Trigger:       Organizer chỉnh sửa quyền lợi của một gói tài trợ
Inputs:        package_id, benefits[]: { category (enum), title (string), description (text) }
Outputs:       Gói tài trợ đã cập nhật với danh sách quyền lợi
Business Rules: BR-0107
Acceptance Criteria:
  Given   gói tài trợ "Nhà tài trợ chính" đã tồn tại
  When    organizer thêm quyền lợi category = BRANDING, title = "Logo trên backdrop chính"
  Then    hệ thống SHALL lưu quyền lợi vào gói tài trợ

  Given   gói tài trợ chưa có quyền lợi nào
  When    organizer cố phát hành hồ sơ
  Then    hệ thống SHALL cảnh báo "Gói tài trợ [tên] chưa có quyền lợi nào"
Priority:      MUST
```

### FR-0108: Xác thực và phát hành hồ sơ tài trợ

```
ID:            FR-0108
Name:          Xác thực tính đầy đủ và phát hành hồ sơ tài trợ
Description:   Hệ thống SHALL xác thực toàn bộ hồ sơ tài trợ theo BR-0108 trước khi cho phép
               phát hành. Khi đạt yêu cầu, hệ thống SHALL chuyển trạng thái từ DRAFT sang PUBLISHED,
               ghi nhận thời gian phát hành, và lập chỉ mục hồ sơ cho tìm kiếm (SF-02).
Classification: SYSTEM-SUPPORTED
Actor:         Organizer (khởi tạo), System (xác thực và lập chỉ mục)
Trigger:       Organizer nhấn "Phát hành hồ sơ"
Inputs:        proposal_id
Outputs:       status = PUBLISHED, published_at (timestamp), validation_result (PASS | FAIL + errors[])
Business Rules: BR-0108
Acceptance Criteria:
  Given   hồ sơ tài trợ ở trạng thái DRAFT với đầy đủ thông tin theo BR-0108
  When    organizer nhấn "Phát hành hồ sơ"
  Then    hệ thống SHALL chuyển trạng thái sang PUBLISHED
  And     hệ thống SHALL lập chỉ mục để hồ sơ xuất hiện trong tìm kiếm

  Given   hồ sơ tài trợ thiếu trường bắt buộc (ví dụ: chưa có tên sự kiện)
  When    organizer nhấn "Phát hành hồ sơ"
  Then    hệ thống SHALL từ chối phát hành
  And     hệ thống SHALL hiển thị danh sách các trường còn thiếu
Priority:      MUST
```

### FR-0109: Hủy phát hành hồ sơ tài trợ

```
ID:            FR-0109
Name:          Hủy phát hành và ẩn hồ sơ tài trợ
Description:   Hệ thống SHALL cho phép organizer hủy phát hành một hồ sơ tài trợ đang ở trạng thái
               PUBLISHED theo điều kiện ở BR-0109. Khi hủy thành công, hệ thống SHALL chuyển hồ sơ
               về trạng thái DRAFT và xóa hồ sơ khỏi chỉ mục tìm kiếm. Các thông báo liên quan
               PHẢI được xử lý theo BR-0110.
Classification: SYSTEM-SUPPORTED
Actor:         Organizer
Trigger:       Organizer nhấn "Hủy phát hành"
Inputs:        proposal_id
Outputs:       status = DRAFT, unpublished_at (timestamp), notification_events[]
Business Rules: BR-0109, BR-0110
Acceptance Criteria:
  Given   hồ sơ tài trợ ở trạng thái PUBLISHED và chưa có Deal nào liên kết từ các lời mời tài trợ
  When    organizer nhấn "Hủy phát hành"
  Then    hệ thống SHALL chuyển trạng thái về DRAFT
  And     hệ thống SHALL xóa hồ sơ khỏi kết quả tìm kiếm

  Given   hồ sơ tài trợ có lời mời tài trợ đã gửi (SENT/PENDING) nhưng chưa có Deal liên kết
  When    organizer nhấn "Hủy phát hành"
  Then    hệ thống SHALL gửi thông báo phản hồi cho bên gửi lời mời
  And     nội dung thông báo SHALL nêu rõ hồ sơ đã hủy phát hành và lời mời không còn hiệu lực

  Given   hồ sơ tài trợ đã có ít nhất một Deal liên kết (đã bắt đầu thương thảo)
  When    organizer nhấn "Hủy phát hành"
  Then    hệ thống SHALL từ chối với thông báo "Không thể hủy phát hành hồ sơ đã bắt đầu thương thảo"
Priority:      SHOULD
```

---

## Business Rules

```
ID:          BR-0101
Rule:        Mỗi hồ sơ tài trợ khi khởi tạo PHẢI có trạng thái ban đầu là DRAFT.
Source:      Quy trình gốc — Bước 1 (Tạo hồ sơ tài trợ)
Type:        Validation

ID:          BR-0102
Rule:        Mỗi hồ sơ tài trợ PHẢI được gán cho đúng một tổ chức BTC (organizer).
             Một BTC có thể có nhiều hồ sơ tài trợ đồng thời.
Source:      Quy trình gốc — Bước 1
Type:        Validation

ID:          BR-0103
Rule:        Các trường số (quy mô, ngân sách) PHẢI là số dương.
             Thời gian tổ chức sự kiện PHẢI nằm trong tương lai so với thời điểm tạo/phát hành.
Source:      Quy trình gốc — Bước 1 (Thông tin cơ bản, Thông tin chi tiết)
Type:        Validation

ID:          BR-0104
Rule:        File hình ảnh upload PHẢI có định dạng JPEG, PNG, hoặc WebP.
             Kích thước tối đa mỗi file là 5MB. [ASSUMED]
Source:      Quy trình gốc — Bước 1 (Hình ảnh nhận diện)
Type:        Validation

ID:          BR-0105
Rule:        Hồ sơ tài trợ PHẢI có ít nhất một hình thức tài trợ được chọn trước khi phát hành.
             Giá trị hợp lệ: CASH, IN_KIND, COMBINED.
Source:      Quy trình gốc — Bước 1 (Hình thức tài trợ)
Type:        Validation

ID:          BR-0106
Rule:        Mỗi gói tài trợ PHẢI có tên duy nhất trong phạm vi hồ sơ, giá trị tối thiểu > 0,
             và số slot ≥ 1. Hồ sơ PHẢI có ít nhất một gói tài trợ trước khi phát hành.
Source:      Quy trình gốc — Bước 1 (Gói tài trợ)
Type:        Validation

ID:          BR-0107
Rule:        Mỗi quyền lợi nhà tài trợ PHẢI thuộc một trong các nhóm: BRANDING, STAGE, DIGITAL, ENGAGEMENT.
             Mỗi quyền lợi PHẢI có mô tả cam kết thực hiện cụ thể. [ASSUMED — quy trình gốc chỉ liệt kê nhóm]
Source:      Quy trình gốc — Bước 1 (Lợi ích mang lại cho nhà tài trợ)
Type:        Validation

ID:          BR-0108
Rule:        Hồ sơ tài trợ chỉ được phát hành khi ĐẦY ĐỦ các trường bắt buộc:
             tên sự kiện, loại hình, thời gian, địa điểm, quy mô, ngân sách, đối tượng khán giả,
             ít nhất một hình thức tài trợ, và ít nhất một gói tài trợ có quyền lợi.
Source:      Quy trình gốc — Bước 1 (toàn bộ)
Type:        Validation

ID:          BR-0109
Rule:        Hồ sơ tài trợ KHÔNG THỂ hủy phát hành nếu đã có ít nhất một Deal liên kết
             được tạo từ lời mời tài trợ đã chấp nhận (đã bắt đầu thương thảo).
Source:      [INFERRED — đảm bảo tính nhất quán với trạng thái thương thảo ở SF-04]
Type:        Routing

ID:          BR-0110
Rule:        Khi hủy phát hành hồ sơ thành công, hệ thống PHẢI gửi thông báo phản hồi:
             (1) cho các doanh nghiệp đã bookmark hồ sơ (nếu có),
             (2) cho bên gửi các lời mời tài trợ liên kết còn ở trạng thái SENT/PENDING và chưa tạo Deal.
             Nội dung thông báo PHẢI nêu rõ hồ sơ đã hủy phát hành và lời mời không còn hiệu lực.
Source:      [INFERRED — đảm bảo các bên liên quan nhận được phản hồi trạng thái hồ sơ]
Type:        Routing + Notification
```

---

## Data Model

```
Entity:        SponsorshipProposal
Attributes:
  - proposal_id: UUID (PK)
  - organizer_id: UUID (FK → Organization)
  - event_name: String (required)
  - event_type: Enum [COMPETITION, WORKSHOP, CONCERT, CONFERENCE, CHARITY, OTHER]
  - event_date_start: DateTime (required)
  - event_date_end: DateTime (required)
  - venue: String (required)
  - expected_scale: Integer (required, > 0)
  - estimated_budget: Decimal (required, > 0)
  - budget_currency: Enum [VND] (default: VND)
  - target_audience: Text (required)
  - program_content: RichText
  - accepted_sponsorship_types: Enum[] [CASH, IN_KIND, COMBINED]
  - banner_url: String (nullable)
  - thumbnail_url: String (nullable)
  - status: Enum [DRAFT, PUBLISHED] (default: DRAFT)
  - created_at: DateTime
  - created_by: UUID (FK → User)
  - updated_at: DateTime
  - published_at: DateTime (nullable)
Relationships:
  - SponsorshipProposal —(1:N)→ SponsorshipPackage
  - SponsorshipProposal —(N:1)→ Organization

Entity:        SponsorshipPackage
Attributes:
  - package_id: UUID (PK)
  - proposal_id: UUID (FK → SponsorshipProposal)
  - package_name: String (required, unique per proposal)
  - package_tier: Enum [TITLE, CO_SPONSOR, ASSOCIATE, TECHNICAL, PARTNER]
  - description: Text
  - min_value: Decimal (required, > 0)
  - available_slots: Integer (required, ≥ 1)
  - created_at: DateTime
  - updated_at: DateTime
Relationships:
  - SponsorshipPackage —(1:N)→ PackageBenefit
  - SponsorshipPackage —(N:1)→ SponsorshipProposal

Entity:        PackageBenefit
Attributes:
  - benefit_id: UUID (PK)
  - package_id: UUID (FK → SponsorshipPackage)
  - category: Enum [BRANDING, STAGE, DIGITAL, ENGAGEMENT]
  - title: String (required)
  - description: Text (required)
  - sort_order: Integer
Relationships:
  - PackageBenefit —(N:1)→ SponsorshipPackage
```

---

## Traceability Matrix

| Process Step | Classification | System Function (FR-ID) | Business Rules (BR-IDs) | Entity |
|---|---|---|---|---|
| 1. Tạo hồ sơ tài trợ — Khởi tạo | SYSTEM-SUPPORTED | FR-0101 | BR-0101, BR-0102 | SponsorshipProposal |
| 1. Thông tin cơ bản | SYSTEM-SUPPORTED | FR-0102 | BR-0103 | SponsorshipProposal |
| 1. Hình ảnh nhận diện | SYSTEM-SUPPORTED | FR-0103 | BR-0104 | SponsorshipProposal |
| 1. Thông tin chi tiết | SYSTEM-SUPPORTED | FR-0104 | BR-0103 | SponsorshipProposal |
| 1. Hình thức tài trợ | SYSTEM-SUPPORTED | FR-0105 | BR-0105 | SponsorshipProposal |
| 1. Gói tài trợ | SYSTEM-SUPPORTED | FR-0106 | BR-0106 | SponsorshipPackage |
| 1. Lợi ích cho nhà tài trợ | SYSTEM-SUPPORTED | FR-0107 | BR-0107 | PackageBenefit |
| 1. Phát hành hồ sơ | SYSTEM-SUPPORTED | FR-0108 | BR-0108 | SponsorshipProposal |
| [INFERRED] Hủy phát hành | SYSTEM-SUPPORTED | FR-0109 | BR-0109, BR-0110 | SponsorshipProposal |
