# UC-32: Xem danh mục gợi ý tự động

**Brief Description**
> Authenticated User (Sponsor hoặc Organizer) truy cập danh mục "Gợi ý" để xem các sự kiện hoặc doanh nghiệp được hệ thống tự động đề xuất. Danh mục được tạo dựa trên hồ sơ hiện có, lịch sử tìm kiếm, bookmark, và hành vi tương tác gần đây của actor. Kết quả được lọc theo vai trò: sponsor nhận gợi ý sự kiện, organizer nhận gợi ý doanh nghiệp.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Sponsor hoặc Organizer) | Người xem danh mục gợi ý |
| Secondary | System | Tính toán, lọc và hiển thị đề xuất phù hợp |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Hệ thống đã thu thập đủ dữ liệu tương tác từ actor (hồ sơ, bookmark, lịch sử tìm kiếm)

---

**Trigger**
> Actor mở tab hoặc trang "Gợi ý" trong khu vực tìm kiếm/khám phá (SF-02).

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Mở tab "Gợi ý" trong trang tìm kiếm/khám phá |
| 2 | System | Xác định vai trò của actor (sponsor / organizer) |
| 3 | System | Tính toán danh sách đề xuất dựa trên: hồ sơ hiện có, lịch sử tìm kiếm, bookmark, hành vi tương tác gần đây (BR-0206) |
| 4 | System | Lọc kết quả theo vai trò: sponsor nhận gợi ý sự kiện/hồ sơ tài trợ, organizer nhận gợi ý doanh nghiệp |
| 5 | System | Hiển thị danh sách đề xuất, mỗi mục gồm: tên đối tượng, lý do gợi ý, điểm phù hợp (relevance_score) |
| 6 | System | Hiển thị thời điểm làm mới cuối (refreshed_at) |
| 7 | System | Use case kết thúc thành công — actor thấy danh mục gợi ý |

---

**Alternate Flows**

> AF-32.a: Actor xem chi tiết đề xuất (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Authenticated User | Nhấn vào một đề xuất trong danh mục |
| 5b | System | Chuyển đến trang chi tiết hồ sơ tương ứng: UC-08 (nếu sponsor xem sự kiện) hoặc UC-09 (nếu organizer xem doanh nghiệp) |

> AF-32.b: Danh mục tự động làm mới (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện dữ liệu đầu vào đã thay đổi kể từ lần làm mới cuối (bookmark mới, tìm kiếm mới) |
| 1b | System | Tự động cập nhật danh mục đề xuất với kết quả mới |
| 1c | System | Tiếp tục tại Step 5 |

---

**Exception Flows**

> EF-32.1: Không đủ dữ liệu để tạo gợi ý (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | System | Phát hiện actor chưa có đủ dữ liệu tương tác để tính toán đề xuất |
| 3b | System | Hiển thị thông báo "Hãy tìm kiếm và tương tác nhiều hơn để nhận gợi ý phù hợp" |
| 3c | System | Hiển thị danh sách mặc định: các hồ sơ phổ biến/mới nhất theo vai trò |

---

**Postconditions**

*Success:*
- Actor xem được danh sách đề xuất phù hợp với vai trò và hành vi
- Mỗi đề xuất bao gồm lý do và điểm phù hợp

*Failure:*
- Danh sách trống hoặc mặc định — actor được khuyến khích tương tác thêm

---

**Business Rules**

- BR-0206: Danh mục "Gợi ý" SHALL được tạo tự động dựa trên hồ sơ hiện có, lịch sử tìm kiếm, bookmark, và hành vi tương tác gần đây. Kết quả PHẢI được lọc theo vai trò: sponsor nhận gợi ý sự kiện, organizer nhận gợi ý doanh nghiệp

---

**Notes / Assumptions**

- Danh mục gợi ý là FULLY AUTOMATED — hệ thống tính toán hoàn toàn, actor chỉ xem kết quả
- Thuật toán gợi ý có thể được cải tiến theo thời gian (machine learning, collaborative filtering) nhưng phiên bản đầu dựa trên rule-based matching
- Liên kết: UC-06, UC-07, UC-08, UC-09, UC-10
