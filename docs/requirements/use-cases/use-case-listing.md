# Danh sách Use Case — UniLink Platform

## Tổng quan

Tài liệu này liệt kê toàn bộ use case của hệ thống UniLink, được phân tích từ 10 System Features
(SF-01 đến SF-10). Mỗi use case đại diện cho **một mục tiêu cụ thể** của một actor trong một phiên
làm việc duy nhất.

> Mô hình use case (quan hệ, phụ thuộc, sơ đồ Mermaid, module decomposition) được tách riêng tại [use-case-model.md](use-case-model.md).

---

## Actor Model

### Abstract Actor

| Actor | Mô tả |
|-------|--------|
| **Authenticated User** | Actor trừu tượng — đại diện cho bất kỳ người dùng đã xác thực trên hệ thống. Là generalization của Organizer và Sponsor. |

### Concrete Actors

| Actor | System Role | Kế thừa từ | Mô tả |
|-------|-------------|------------|--------|
| **Guest** | `guest` | — | Người dùng chưa đăng nhập / chưa có tài khoản trên hệ thống |
| **Organizer (BTC)** | `organizer` | Authenticated User | Ban tổ chức — Câu lạc bộ, đội nhóm sinh viên tổ chức sự kiện |
| **Sponsor (Doanh nghiệp)** | `sponsor` | Authenticated User | Doanh nghiệp — Nhà tài trợ tiềm năng hoặc đã ký kết |
| **System** | `system` | — | Hệ thống UniLink — xử lý tự động, thông báo, nhắc nhở, tính toán |
| **Admin** | `admin` | Authenticated User | Quản trị viên hệ thống — kiểm duyệt nội dung, xác thực tổ chức |

---

## Danh sách Use Case

### SF-01: Sponsorship Proposal Management

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-01 | Tạo hồ sơ tài trợ sự kiện | Organizer | FR-0101 | [UC-01](specifications/UC-01_create_sponsorship_proposal.md) |
| UC-02 | Chỉnh sửa nội dung hồ sơ tài trợ | Organizer | FR-0102, FR-0103, FR-0104, FR-0105 | [UC-02](specifications/UC-02_edit_sponsorship_proposal_content.md) |
| UC-03 | Quản lý gói tài trợ | Organizer | FR-0106, FR-0107 | [UC-03](specifications/UC-03_manage_sponsorship_packages.md) |
| UC-04 | Phát hành hồ sơ tài trợ | Organizer | FR-0108 | [UC-04](specifications/UC-04_publish_sponsorship_proposal.md) |
| UC-05 | Hủy phát hành hồ sơ tài trợ | Organizer | FR-0109 | [UC-05](specifications/UC-05_unpublish_sponsorship_proposal.md) |

---

### SF-02: Event & Partner Discovery

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-06 | Tìm kiếm sự kiện để tài trợ | Sponsor | FR-0201 | [UC-06](specifications/UC-06_search_events_for_sponsorship.md) |
| UC-07 | Tìm kiếm doanh nghiệp để mời tài trợ | Organizer | FR-0202 | [UC-07](specifications/UC-07_search_businesses_for_partnership.md) |
| UC-08 | Xem chi tiết hồ sơ tài trợ sự kiện | Sponsor | FR-0203 | [UC-08](specifications/UC-08_view_sponsorship_proposal_details.md) |
| UC-09 | Xem chi tiết hồ sơ doanh nghiệp | Organizer | FR-0204 | [UC-09](specifications/UC-09_view_business_profile_details.md) |
| UC-10 | Lưu hồ sơ quan tâm | Authenticated User | FR-0205 | [UC-10](specifications/UC-10_bookmark_profile.md) |
| UC-32 | Xem danh mục gợi ý tự động | Authenticated User | FR-0206 | [UC-32](specifications/UC-32_view_automated_recommendations.md) |

---

### SF-03: Sponsorship Invitation Management

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-11 | Gửi lời mời tài trợ | Authenticated User | FR-0301, FR-0302 | [UC-11](specifications/UC-11_send_sponsorship_invitation.md) |
| UC-12 | Phản hồi lời mời tài trợ | Authenticated User | FR-0303, FR-0304, FR-0305 | [UC-12](specifications/UC-12_respond_to_sponsorship_invitation.md) |
| UC-13 | Theo dõi danh sách lời mời tài trợ | Authenticated User | FR-0306 | [UC-13](specifications/UC-13_track_sponsorship_invitations.md) |

---

### SF-04: Negotiation & Communication

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-14 | Trao đổi tin nhắn trong thương vụ | Authenticated User | FR-0401, FR-0402, FR-1301, FR-1303 | [UC-14](specifications/UC-14_exchange_messages_in_deal.md) |
| UC-15 | Đặt lịch họp thương thảo | Authenticated User | FR-0403 | [UC-15](specifications/UC-15_schedule_meeting.md) |
| UC-16 | Phản hồi đề xuất lịch họp | Authenticated User | FR-0404 | [UC-16](specifications/UC-16_respond_to_meeting_proposal.md) |
| UC-17 | Ghi nhận kết quả cuộc họp | Authenticated User | FR-0405 | [UC-17](specifications/UC-17_record_meeting_outcomes.md) |
| UC-18 | Xác nhận đồng thuận ký kết | Authenticated User | FR-0406, FR-1201, FR-1304 | [UC-18](specifications/UC-18_confirm_mutual_agreement.md) |
| UC-19 | Hủy bỏ thương thảo | Authenticated User | FR-0407 | [UC-19](specifications/UC-19_terminate_negotiation.md) |
| UC-56 | Tạo thỏa thuận nháp | Authenticated User | FR-0408 | [UC-56](specifications/UC-56_create_draft_agreement.md) |

> **Ghi chú [UPDATED — BP03]:**
>
> - UC-14: Bổ sung Data Masking (FR-1301) và anti-bypass detection (FR-1303) vào luồng nhắn tin.
> - UC-18: Chuyển đồng thuận sang AWAITING_PAYMENT thay vì AGREED. Yêu cầu Thỏa thuận nháp (UC-56) và miễn trừ trách nhiệm hiện vật (FR-1304).
> - UC-56: Mới — Tạo thỏa thuận nháp (FR-0408), prerequisite cho UC-18.

---

### SF-05: Contract Management

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-20 | Chỉnh sửa điều khoản hợp đồng | Authenticated User | FR-0501, FR-0502 | [UC-20](specifications/UC-20_edit_contract_terms.md) |
| UC-21 | Xác nhận nội dung hợp đồng | Authenticated User | FR-0503 | [UC-21](specifications/UC-21_confirm_contract_content.md) |
| UC-22 | Ký hợp đồng điện tử | Authenticated User | FR-0504 | [UC-22](specifications/UC-22_sign_contract_electronically.md) |
| UC-23 | Xuất hợp đồng dạng PDF | Authenticated User | FR-0505 | [UC-23](specifications/UC-23_export_contract_as_pdf.md) |
| ~~UC-24~~ | ~~Yêu cầu hóa đơn VAT~~ | ~~Sponsor~~ | ~~FR-0506~~ | ~~REMOVED — BP03~~ |
| ~~UC-33~~ | ~~Hủy đồng thuận ký kết hợp đồng~~ | ~~Authenticated User~~ | ~~FR-0507~~ | ~~REMOVED — hard-lock sau 2/2 thanh toán~~ |
| UC-49 | Xử lý vi phạm ký kết hợp đồng | Authenticated User | FR-1406 | [UC-49](specifications/UC-49_handle_contract_signing_breach.md) |

> **Ghi chú [UPDATED — BP03]:**
>
> - UC-24 (Yêu cầu hóa đơn VAT cho giá trị tài trợ) đã bị **LOẠI BỎ**. Nền tảng chỉ xuất hóa đơn VAT cho Phí quản lý chiến dịch (SF-12). FR-0506 và BR-0508 đã bị xóa.
> - FR-0501: Trigger thay đổi — hợp đồng chỉ được tạo sau khi thanh toán phí dịch vụ hoàn tất (SF-12, 2/2 payment).

---

### SF-06: Obligation Fulfillment

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-25 | Theo dõi trạng thái nghĩa vụ | Authenticated User | FR-0601, FR-0602 | [UC-25](specifications/UC-25_track_obligation_status.md) |
| UC-26 | Nộp bằng chứng hoàn thành nghĩa vụ | Authenticated User | FR-0603 | [UC-26](specifications/UC-26_submit_obligation_fulfillment_evidence.md) |
| UC-27 | Xác nhận hoàn thành nghĩa vụ | Authenticated User | FR-0604 | [UC-27](specifications/UC-27_confirm_obligation_completion.md) |
| UC-28 | Nộp báo cáo kết quả sự kiện | Organizer | FR-0605, FR-0606 | [UC-28](specifications/UC-28_submit_event_report.md) |

---

### SF-07: Review & Reputation

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-29 | Gửi đánh giá đối tác | Authenticated User | FR-0701, FR-0702, FR-0703 | [UC-29](specifications/UC-29_submit_partner_review.md) |
| UC-30 | Xem điểm uy tín đối tác | Authenticated User | FR-0704 | [UC-30](specifications/UC-30_view_reputation_score.md) |
| UC-31 | Báo cáo đánh giá vi phạm | Authenticated User | FR-0705 | [UC-31](specifications/UC-31_report_inappropriate_review.md) |

---

### SF-08: Account Registration & Authentication

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-34 | Đăng ký tài khoản bằng email | Guest | FR-0801, FR-0804, FR-0805 | [UC-34](specifications/UC-34_register_account_by_email.md) |
| UC-35 | Đăng ký tài khoản bằng Google | Guest | FR-0802, FR-0804, FR-0805 | [UC-35](specifications/UC-35_register_account_by_google.md) |
| UC-36 | Đăng nhập hệ thống | Guest | FR-0803, FR-0802 | [UC-36](specifications/UC-36_login.md) |
| UC-37 | Đặt lại mật khẩu | Guest | FR-0806 | [UC-37](specifications/UC-37_reset_password.md) |

> **Ghi chú FR mapping**: FR-0804 (Chọn vai trò) và FR-0805 (Nhập thông tin tổ chức) được gộp vào UC-34 và UC-35 vì chúng là bước bắt buộc trong cùng phiên đăng ký, không có mục tiêu độc lập. FR-0802 cover cả đăng ký (UC-35) và đăng nhập (UC-36 — alternate flow).

---

### SF-09: Organization Profile & Document Management

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-38 | Bổ sung thông tin và tài liệu minh chứng | Authenticated User | FR-0901 | [UC-38](specifications/UC-38_supplement_org_info_documents.md) |
| UC-39 | Chỉnh sửa hồ sơ tổ chức | Authenticated User | FR-0902 | [UC-39](specifications/UC-39_edit_organization_profile.md) |
| UC-40 | Gửi hồ sơ xác thực | Authenticated User | FR-0903 | [UC-40](specifications/UC-40_submit_verification_request.md) |

> **Ghi chú FR mapping**: FR-0904 (Phân quyền tự động) và FR-0905 (Xóa tài liệu tạm) là FULLY AUTOMATED — được nhúng vào system response của UC liên quan (UC-40, UC-43) thay vì tạo UC riêng, nhất quán với nguyên tắc thiết kế hiện tại.

---

### SF-10: Organization Verification & Moderation

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-41 | Xem danh sách hồ sơ chờ kiểm duyệt | Admin | FR-1001 | [UC-41](specifications/UC-41_view_pending_verifications.md) |
| UC-42 | Xem chi tiết hồ sơ xác thực | Admin | FR-1002 | [UC-42](specifications/UC-42_view_verification_details.md) |
| UC-43 | Phê duyệt hồ sơ tổ chức | Admin | FR-1003 | [UC-43](specifications/UC-43_approve_organization.md) |
| UC-44 | Từ chối hồ sơ tổ chức | Admin | FR-1004 | [UC-44](specifications/UC-44_reject_organization.md) |
| UC-45 | Yêu cầu bổ sung thông tin hồ sơ | Admin | FR-1005 | [UC-45](specifications/UC-45_request_additional_info.md) |

> **Ghi chú FR mapping**: FR-1006 (Thông báo sự kiện xác thực) là FULLY AUTOMATED — được nhúng vào postconditions của UC-43, UC-44, UC-45 thay vì tạo UC riêng.

---

### SF-11: Public Organization Profile & Sponsorship History

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-46 | Xem hồ sơ tổ chức công khai | Authenticated User | FR-1101, FR-1104 | [UC-46](specifications/UC-46_view_public_organization_profile.md) |
| UC-47 | Xem lịch sử hồ sơ tài trợ công khai | Authenticated User | FR-1102 | [UC-47](specifications/UC-47_view_public_sponsorship_history.md) |
| UC-48 | Xem lịch sử giao dịch tài trợ công khai | Authenticated User | FR-1103 | [UC-48](specifications/UC-48_view_public_sponsorship_transaction_history.md) |

> **Ghi chú**: Chỉ Authenticated User mới truy cập được public profile (không có Guest). FR-1104 hiển thị tóm tắt uy tín + liên kết sang UC-30/SCR-018. UC-30 là `<<extend>>` từ UC-46. Chỉ tổ chức VERIFIED mới có public profile (BR-1101).

---

### SF-12: Service Fee Calculation & Paywall `[NEW — BP03]`

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-50 | Thanh toán phí dịch vụ kết nối | Authenticated User | FR-1201→1207, FR-1401→1403 | [UC-50](specifications/UC-50_pay_service_fee.md) |
| UC-51 | Xem trước chi phí dịch vụ | Authenticated User | FR-1202 | [UC-51](specifications/UC-51_preview_service_fee.md) |
| UC-52 | Xuất hóa đơn VAT phí dịch vụ | System | FR-1208 | [UC-52](specifications/UC-52_issue_service_fee_vat_invoice.md) |

> **Ghi chú:** Nhiều FR tự động (FR-1201, FR-1204→1207, FR-1401→1403) được tích hợp vào UC-50 dưới dạng system response/postconditions theo convention của dự án.

---

### SF-13: Contact Data Masking & Unlocking `[NEW — BP03]`

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-53 | Xem xét vi phạm lách bộ lọc | Admin | FR-1303 | [UC-53](specifications/UC-53_review_bypass_violation.md) |

> **Ghi chú:** FR-1301 (masking) và FR-1302 (unmask) là FULLY AUTOMATED — được nhúng vào UC-14 và UC-50. FR-1304 (in-kind disclaimer) được nhúng vào UC-18.

---

### SF-14: Payment Risk Management & Compliance `[NEW — BP03]`

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-54 | Xem báo cáo doanh thu nền tảng | Admin | FR-1404 | [UC-54](specifications/UC-54_view_platform_revenue_report.md) |
| UC-55 | Đối soát thanh toán thủ công | Admin | FR-1405 | [UC-55](specifications/UC-55_manual_payment_reconciliation.md) |

> **Ghi chú:** FR-1401 (auto-refund), FR-1402 (non-refundable), FR-1403 (nhắc nhở) là FULLY AUTOMATED — được nhúng vào UC-50 postconditions. UC-49 (vi phạm ký kết) thuộc SF-05/SF-14 cross-cutting.

---

## Thống kê

| Nhóm Feature | Số lượng UC |
|---|---|
| SF-01: Sponsorship Proposal Management | 5 |
| SF-02: Event & Partner Discovery | 6 |
| SF-03: Sponsorship Invitation Management | 3 |
| SF-04: Negotiation & Communication | 7 (+1 UC-56 mới) |
| SF-05: Contract Management | 4 (-2 UC-24/UC-33 removed, +1 UC-49) |
| SF-06: Obligation Fulfillment | 4 |
| SF-07: Review & Reputation | 3 |
| SF-08: Account Registration & Authentication | 4 |
| SF-09: Organization Profile & Document Management | 3 |
| SF-10: Organization Verification & Moderation | 5 |
| SF-11: Public Organization Profile & Sponsorship History | 3 |
| SF-12: Service Fee Calculation & Paywall | 3 |
| SF-13: Contact Data Masking & Unlocking | 1 |
| SF-14: Payment Risk Management & Compliance | 2 |
| **Tổng cộng** | **53** |

---

## Ghi chú về Actor Model

- **Authenticated User** là actor trừu tượng, đóng vai trò generalization cho Organizer và Sponsor.
  Các use case có Primary Actor là "Authenticated User" có nghĩa cả Organizer và Sponsor đều có thể
  thực hiện use case đó (tùy ngữ cảnh cụ thể).
- **Guest** là actor đại diện cho người dùng chưa có tài khoản hoặc chưa đăng nhập.
  Sau khi hoàn tất đăng ký (UC-34/UC-35), Guest trở thành Authenticated User.
  Sau khi đăng nhập (UC-36), Guest cũng trở thành Authenticated User.
- Mỗi tổ chức chỉ có **một tài khoản đại diện** trên nền tảng; hệ thống không quản lý thành viên nội bộ hay phân quyền nhiều người dùng trong cùng tổ chức.
- **System** là secondary actor trong hầu hết các use case — chịu trách nhiệm xử lý tự động
  (gửi thông báo, tính toán, xác thực dữ liệu).
- Các hành vi hoàn toàn tự động (fully automated) như gửi thông báo, tự động hết hạn,
  tự động tạo nghĩa vụ, phân quyền tự động, xóa tài liệu tạm được tích hợp vào các use case
  liên quan dưới dạng system response trong Main Flow hoặc Postconditions, thay vì tạo use case
  riêng biệt.
