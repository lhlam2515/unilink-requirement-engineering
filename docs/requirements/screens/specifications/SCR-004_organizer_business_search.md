# SCR-004: Organizer_BusinessSearch_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-004 |
| **Screen Name** | Organizer_BusinessSearch_Screen |
| **Mục đích** | Organizer tìm kiếm doanh nghiệp phù hợp để mời tài trợ cho sự kiện, và xem danh mục gợi ý tự động |
| **Actor chính** | Organizer (BTC) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 2 — Tìm kiếm và tiếp cận đối tác phù hợp (2.2 — BTC chủ động tìm kiếm) |
| **Use case liên quan** | UC-07, UC-32 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Tìm kiếm doanh nghiệp — discovery context cho organizer |
| Data scope riêng | ✅ Danh sách doanh nghiệp ACTIVE + bộ lọc + gợi ý |
| Action set riêng | ✅ Tìm kiếm, lọc, sắp xếp, xem gợi ý |
| Navigation boundary | ✅ Điều hướng sang SCR-006 khi chọn doanh nghiệp |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

### Tab "Tìm kiếm" (UC-07)

Mỗi kết quả hiển thị:

- Tên doanh nghiệp (business_name)
- Logo (logo_url)
- Lĩnh vực hoạt động (industry)
- Khu vực (region)
- Mục tiêu tài trợ (sponsorship_goal): MARKETING / CSR
- Ngân sách tài trợ dự kiến (sponsorship_budget)

### Tab "Gợi ý cho bạn" (UC-32)

Mỗi đề xuất hiển thị:

- Tên doanh nghiệp (business_name)
- Logo (logo_url)
- Lý do gợi ý (recommendation_reason)
- Điểm phù hợp (relevance_score)
- Thời điểm làm mới cuối (refreshed_at)

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Từ khóa tìm kiếm | Text input | Full-text search: tên, mô tả, lĩnh vực |
| Khu vực hoạt động | Text/Select | Bộ lọc |
| Đối tượng khách hàng | Tags / Keywords | Bộ lọc |
| Lĩnh vực | Select / Multi-select | Bộ lọc |
| Ngân sách tài trợ (range) | Range slider / Min-Max | Bộ lọc |
| Mục tiêu tài trợ | Select | MARKETING / CSR / ALL |
| Tiêu chí sắp xếp | Select | Phù hợp / Ngân sách / Lĩnh vực |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Tìm kiếm / Áp dụng bộ lọc** | Cập nhật danh sách kết quả (UC-07 Main-4~8) |
| **Nhấn vào doanh nghiệp** | Chuyển đến SCR-006 (Xem chi tiết doanh nghiệp) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Chuyển tab "Gợi ý cho bạn" | Hiển thị danh mục gợi ý doanh nghiệp (UC-32) |
| Thay đổi sắp xếp | Sắp xếp lại kết quả (UC-07 AF-07.a) |
| Xóa bộ lọc | Reset tiêu chí |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0201: Kết quả chỉ bao gồm doanh nghiệp ACTIVE
- BR-0203: Bộ lọc: khu vực, đối tượng KH, lĩnh vực, ngân sách (range), mục tiêu tài trợ. AND logic
- BR-0206: Tab "Gợi ý" tạo tự động. Organizer nhận gợi ý doanh nghiệp

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Dashboard / Menu chính | Menu "Tìm kiếm doanh nghiệp" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn vào doanh nghiệp trong kết quả | SCR-006 (Business Detail) |
| Nhấn vào đề xuất trong tab Gợi ý | SCR-006 (Business Detail) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải kết quả |
| Empty Search State | Không tìm thấy doanh nghiệp (UC-07 EF-07.1) |
| Insufficient Data State | Chưa đủ dữ liệu gợi ý (UC-32 EF-32.1) |

## UI Components liên quan

- Tab strip — "Tìm kiếm" / "Gợi ý cho bạn"
- Search bar — từ khóa
- Filter panel — bộ lọc nâng cao
- Card grid / List — kết quả doanh nghiệp
- Sort dropdown — tiêu chí sắp xếp
- Pagination — phân trang
- Relevance badge — điểm phù hợp

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-07 | Main-1 | Truy cập trang tìm kiếm DN | Screen entry | Page load |
| UC-07 | Main-2 | Hiển thị form tìm kiếm | Screen content | Filter panel |
| UC-07 | Main-3~4 | Nhập tiêu chí + tìm kiếm | Input + Action | Filters + Search button |
| UC-07 | Main-5~7 | Truy vấn + áp dụng lọc + full-text | Action | System processing |
| UC-07 | Main-8 | Hiển thị kết quả phân trang | Read-only data | Card grid + Pagination |
| UC-07 | AF-07.a | Thay đổi sắp xếp | Component | Sort dropdown |
| UC-07 | EF-07.1 | Không có kết quả | UI State | Empty search state |
| UC-32 | Main-1 | Mở tab "Gợi ý" | Action | Tab switch |
| UC-32 | Main-2~4 | Tính toán + lọc theo vai trò | Action | System processing |
| UC-32 | Main-5 | Hiển thị đề xuất | Read-only data | Recommendation cards |
| UC-32 | Main-6 | Thời điểm làm mới | Read-only data | Refresh timestamp |
| UC-32 | AF-32.a | Xem chi tiết đề xuất | Navigation | Nhấn → SCR-006 |
| UC-32 | EF-32.1 | Không đủ dữ liệu | UI State | Fallback list |
