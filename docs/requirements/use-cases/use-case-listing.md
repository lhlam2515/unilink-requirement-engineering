# Danh sách Use Case — UniLink Platform

## Tổng quan

Tài liệu này liệt kê toàn bộ use case của hệ thống UniLink, được phân tích từ 7 System Features
(SF-01 đến SF-07). Mỗi use case đại diện cho **một mục tiêu cụ thể** của một actor trong một phiên
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
| **Organizer (BTC)** | `organizer` | Authenticated User | Ban tổ chức — Câu lạc bộ, đội nhóm sinh viên tổ chức sự kiện |
| **Sponsor (Doanh nghiệp)** | `sponsor` | Authenticated User | Doanh nghiệp — Nhà tài trợ tiềm năng hoặc đã ký kết |
| **System** | `system` | — | Hệ thống UniLink — xử lý tự động, thông báo, nhắc nhở, tính toán |
| **Admin** | `admin` | Authenticated User | Quản trị viên hệ thống — kiểm duyệt nội dung |

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
| UC-14 | Trao đổi tin nhắn trong thương vụ | Authenticated User | FR-0401, FR-0402 | [UC-14](specifications/UC-14_exchange_messages_in_deal.md) |
| UC-15 | Đặt lịch họp thương thảo | Authenticated User | FR-0403 | [UC-15](specifications/UC-15_schedule_meeting.md) |
| UC-16 | Phản hồi đề xuất lịch họp | Authenticated User | FR-0404 | [UC-16](specifications/UC-16_respond_to_meeting_proposal.md) |
| UC-17 | Ghi nhận kết quả cuộc họp | Authenticated User | FR-0405 | [UC-17](specifications/UC-17_record_meeting_outcomes.md) |
| UC-18 | Xác nhận đồng thuận ký kết | Authenticated User | FR-0406 | [UC-18](specifications/UC-18_confirm_mutual_agreement.md) |
| UC-19 | Hủy bỏ thương thảo | Authenticated User | FR-0407 | [UC-19](specifications/UC-19_terminate_negotiation.md) |

---

### SF-05: Contract Management

| UC ID | Tên Use Case | Primary Actor | Source FRs | File |
|-------|-------------|---------------|------------|------|
| UC-20 | Chỉnh sửa điều khoản hợp đồng | Authenticated User | FR-0501, FR-0502 | [UC-20](specifications/UC-20_edit_contract_terms.md) |
| UC-21 | Xác nhận nội dung hợp đồng | Authenticated User | FR-0503 | [UC-21](specifications/UC-21_confirm_contract_content.md) |
| UC-22 | Ký hợp đồng điện tử | Authenticated User | FR-0504 | [UC-22](specifications/UC-22_sign_contract_electronically.md) |
| UC-23 | Xuất hợp đồng dạng PDF | Authenticated User | FR-0505 | [UC-23](specifications/UC-23_export_contract_as_pdf.md) |
| UC-24 | Yêu cầu hóa đơn VAT | Sponsor | FR-0506 | [UC-24](specifications/UC-24_request_vat_invoice.md) |
| UC-33 | Hủy đồng thuận ký kết hợp đồng | Authenticated User | FR-0507 | [UC-33](specifications/UC-33_cancel_contract_agreement.md) |

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

## Thống kê

| Nhóm Feature | Số lượng UC |
|---|---|
| SF-01: Sponsorship Proposal Management | 5 |
| SF-02: Event & Partner Discovery | 6 |
| SF-03: Sponsorship Invitation Management | 3 |
| SF-04: Negotiation & Communication | 6 |
| SF-05: Contract Management | 6 |
| SF-06: Obligation Fulfillment | 4 |
| SF-07: Review & Reputation | 3 |
| **Tổng cộng** | **33** |

---

## Ghi chú về Actor Model

- **Authenticated User** là actor trừu tượng, đóng vai trò generalization cho Organizer và Sponsor.
  Các use case có Primary Actor là "Authenticated User" có nghĩa cả Organizer và Sponsor đều có thể
  thực hiện use case đó (tùy ngữ cảnh cụ thể).
- Mỗi tổ chức chỉ có **một tài khoản đại diện** trên nền tảng; hệ thống không quản lý thành viên nội bộ hay phân quyền nhiều người dùng trong cùng tổ chức.
- **System** là secondary actor trong hầu hết các use case — chịu trách nhiệm xử lý tự động
  (gửi thông báo, tính toán, xác thực dữ liệu).
- Các hành vi hoàn toàn tự động (fully automated) như gửi thông báo, tự động hết hạn,
  tự động tạo nghĩa vụ được tích hợp vào các use case liên quan dưới dạng system response
  trong Main Flow hoặc Alternate Flow, thay vì tạo use case riêng biệt.
