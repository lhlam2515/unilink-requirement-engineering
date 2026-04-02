# SCR-003: Sponsor_EventSearch_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-003 |
| **Screen Name** | Sponsor_EventSearch_Screen |
| **Mục đích** | Sponsor tìm kiếm các hồ sơ tài trợ sự kiện đã phát hành, và xem danh mục gợi ý tự động phù hợp với nhu cầu |
| **Actor chính** | Sponsor (Doanh nghiệp) |
| **Quy trình nghiệp vụ** | BP-01 / Bước 2 — Tìm kiếm và tiếp cận đối tác phù hợp (2.1 — Doanh nghiệp chủ động tìm kiếm) |
| **Use case liên quan** | UC-06, UC-32 |

---

## Interaction Boundary Justification

| Tiêu chí | Đánh giá |
|---|---|
| Context/goal riêng biệt | ✅ Tìm kiếm sự kiện — mục tiêu discovery/browsing |
| Data scope riêng | ✅ Danh sách hồ sơ PUBLISHED + bộ lọc + kết quả gợi ý |
| Action set riêng | ✅ Tìm kiếm, lọc, sắp xếp, phân trang, xem gợi ý |
| Navigation boundary | ✅ Điều hướng sang SCR-005 khi chọn sự kiện |
| Independently testable | ✅ |

---

## Dữ liệu hiển thị (Read-only Data)

### Tab "Tìm kiếm" (UC-06)

Mỗi kết quả hiển thị:

- Tên sự kiện (event_name)
- Thumbnail (thumbnail_url)
- Loại hình sự kiện (event_type)
- Địa điểm (event_location)
- Thời gian tổ chức (event_date_start ~ event_date_end)
- Quy mô dự kiến (expected_scale)
- Ngân sách dự kiến (expected_budget)
- Hình thức tài trợ (sponsorship_type)
- Thống kê phân trang (total results, current page)

### Tab "Gợi ý cho bạn" (UC-32)

Mỗi đề xuất hiển thị:

- Tên sự kiện (event_name)
- Thumbnail (thumbnail_url)
- Lý do gợi ý (recommendation_reason)
- Điểm phù hợp (relevance_score)
- Thời điểm làm mới cuối (refreshed_at)

---

## Dữ liệu nhập (Input Fields)

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Từ khóa tìm kiếm | Text input | Full-text search |
| Địa điểm tổ chức | Text/Select | Bộ lọc |
| Quy mô (range) | Range slider / Min-Max input | Bộ lọc |
| Ngân sách (range) | Range slider / Min-Max input | Bộ lọc |
| Đối tượng khán giả | Tags / Keywords input | Bộ lọc |
| Hình thức tài trợ | Multi-select | CASH / IN_KIND / COMBINED |
| Tiêu chí sắp xếp | Select | Phù hợp / Ngày / Quy mô / Ngân sách |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| **Tìm kiếm / Áp dụng bộ lọc** | Cập nhật danh sách kết quả (UC-06 Main-4~8) |
| **Nhấn vào sự kiện** | Chuyển đến SCR-005 (Xem chi tiết hồ sơ tài trợ) |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Chuyển tab "Gợi ý cho bạn" | Hiển thị danh mục gợi ý (UC-32) |
| Thay đổi sắp xếp | Sắp xếp lại kết quả (UC-06 AF-06.a) |
| Chuyển trang | Tải trang kết quả tiếp theo (UC-06 AF-06.b) |
| Xóa bộ lọc | Reset toàn bộ tiêu chí tìm kiếm |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-0201: Kết quả chỉ bao gồm hồ sơ PUBLISHED
- BR-0202: Bộ lọc: địa điểm, quy mô (range), đối tượng (keywords), ngân sách (range), hình thức (multi-select). AND logic
- BR-0206: Tab "Gợi ý" tạo tự động dựa trên hồ sơ, lịch sử, bookmark, hành vi. Lọc theo vai trò: sponsor nhận gợi ý sự kiện

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Dashboard / Menu chính | Menu "Tìm kiếm sự kiện" |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn vào sự kiện trong kết quả | SCR-005 (Proposal Detail) |
| Nhấn vào đề xuất trong tab Gợi ý | SCR-005 (Proposal Detail) |

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải kết quả tìm kiếm / gợi ý |
| Empty Search State | Không tìm thấy sự kiện phù hợp (UC-06 EF-06.1) |
| Insufficient Data State | Chưa đủ dữ liệu để tạo gợi ý (UC-32 EF-32.1) — hiển thị danh sách mặc định |

## UI Components liên quan

- Tab strip — "Tìm kiếm" / "Gợi ý cho bạn"
- Search bar — từ khóa tìm kiếm
- Filter panel — bộ lọc nâng cao
- Card grid / List — kết quả sự kiện
- Sort dropdown — tiêu chí sắp xếp
- Pagination — phân trang
- Relevance badge — điểm phù hợp trên thẻ gợi ý
- Refresh timestamp — thời điểm làm mới gợi ý

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-06 | Main-1 | Truy cập trang tìm kiếm | Screen entry | Page load, Tab "Tìm kiếm" |
| UC-06 | Main-2 | Hiển thị form tìm kiếm | Screen content | Filter panel + Search bar |
| UC-06 | Main-3~4 | Nhập tiêu chí + nhấn tìm kiếm | Input + Action | Filter inputs + Search button |
| UC-06 | Main-5~7 | Truy vấn + áp dụng bộ lọc + sắp xếp | Action | System processing |
| UC-06 | Main-8 | Hiển thị kết quả phân trang | Read-only data | Card grid + Pagination |
| UC-06 | AF-06.a | Thay đổi sắp xếp | Component | Sort dropdown |
| UC-06 | AF-06.b | Điều hướng phân trang | Component | Pagination |
| UC-06 | EF-06.1 | Không có kết quả | UI State | Empty search state |
| UC-32 | Main-1 | Mở tab "Gợi ý" | Action | Tab switch |
| UC-32 | Main-2~4 | Tính toán + lọc theo vai trò | Action | System processing |
| UC-32 | Main-5 | Hiển thị danh sách đề xuất | Read-only data | Recommendation cards |
| UC-32 | Main-6 | Thời điểm làm mới | Read-only data | Refresh timestamp |
| UC-32 | AF-32.a | Xem chi tiết đề xuất | Navigation | Nhấn → SCR-005 |
| UC-32 | AF-32.b | Tự động làm mới | Action | Auto-refresh |
| UC-32 | EF-32.1 | Không đủ dữ liệu | UI State | Fallback: danh sách phổ biến/mới nhất |
