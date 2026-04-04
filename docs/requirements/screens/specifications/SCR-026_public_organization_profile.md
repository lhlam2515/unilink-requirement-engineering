# SCR-026: User_PublicOrganizationProfile_Screen

## Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| **Screen ID** | SCR-026 |
| **Screen Name** | User_PublicOrganizationProfile_Screen |
| **Mục đích** | Người dùng xem hồ sơ tổ chức công khai, tóm tắt uy tín, và lịch sử công khai phù hợp theo vai trò duy nhất của tổ chức. Chỉ hiển thị cho tổ chức đã VERIFIED. |
| **Actor chính** | Guest hoặc Authenticated User |
| **Quy trình nghiệp vụ** | SF-11 / Hồ sơ tổ chức công khai và lịch sử tài trợ |
| **Use case liên quan** | UC-46, UC-47, UC-48 |

---

## Interaction Boundary Justification

| Tiêu chí | SCR-026 | So với SCR-018 |
|---|---|---|
| Context/goal riêng biệt | Public hub để xem hồ sơ tổ chức và lịch sử công khai | SCR-018 tập trung vào uy tín chi tiết và đánh giá |
| Data scope riêng | Identity, verification badge, reputation summary (read-only), history lists | SCR-018 chỉ có điểm uy tín, điểm chất lượng, review list |
| Action set riêng | Chọn tab lịch sử, lọc, phân trang, nhấn link chi tiết sự kiện/DN | SCR-018 cho xem thêm đánh giá và báo cáo đánh giá |
| Navigation boundary | Có thể vào từ URL public, từ link chia sẻ, hoặc từ màn chi tiết liên quan | SCR-018 là màn chuyên sâu, chỉ cho Authenticated User |
| Independently testable | Có | Có |

> **Kết luận quan hệ**: SCR-026 là màn public hub cấp cao (Guest + AU). SCR-018 là màn chuyên sâu về reputation/reviews (chỉ AU). Guest KHÔNG được điều hướng từ SCR-026 sang SCR-018.

---

## So sánh chức năng

| Khía cạnh | SCR-026 | SCR-018 | SCR-006 |
|---|---|---|---|
| Mục tiêu chính | Nhận diện tổ chức + lịch sử công khai | Đánh giá độ tin cậy qua reviews | Xem chi tiết DN để mời tài trợ |
| Người dùng mục tiêu | Guest, đối tác | AU đang cân nhắc hợp tác | Organizer đã đăng nhập |
| Dữ liệu chủ đạo | Tên, logo, xác thực, uy tín summary, lịch sử public | Điểm uy tín, review list | Tên, logo, ngân sách, sponsorship goal |
| Hành động chính | Chọn tab, lọc, xem thêm lịch sử | Xem thêm review, báo cáo vi phạm | Bookmark, gửi lời mời tài trợ |
| Mức độ chi tiết | Tổng quan công khai | Chuyên sâu reputation | Chuyên sâu business profile |
| Quan hệ | Superset công khai (read-only) | Drill-down reputation (AU only) | View riêng cho Organizer (authenticated) |

---

## Dữ liệu hiển thị (Read-only Data)

### Khu vực đầu trang

- Tên tổ chức
- Logo / avatar
- Vai trò duy nhất: Organizer hoặc Sponsor
- Trạng thái xác thực (luôn = VERIFIED trên màn này)
- Khu vực hoạt động
- Mô tả ngắn công khai

### Khu vực tóm tắt uy tín (read-only, không có link đi SCR-018)

- Điểm uy tín trung bình (X.X/5 ⭐)
- Điểm chất lượng hợp tác trung bình
- Tổng số đánh giá
- Nếu chưa có đánh giá: hiển thị "Chưa có đánh giá"

### Khu vực lịch sử công khai (chỉ một loại theo vai trò)

#### Với Organizer

- Danh sách lịch sử hồ sơ tài trợ công khai (tối đa 5 mục)
- Mỗi mục: tên sự kiện, năm, trạng thái công khai (PUBLISHED/COMPLETED/ARCHIVED), nhãn tóm tắt, thời điểm phát hành/lưu trữ
- Liên kết sang view chi tiết sự kiện (nếu có)

#### Với Sponsor

- Danh sách lịch sử giao dịch tài trợ công khai (tối đa 5 mục)
- Mỗi mục: tên sự kiện, năm, hình thức tài trợ (CASH/IN_KIND/MIXED), trạng thái hoàn tất (COMPLETED/ARCHIVED), nhãn tóm tắt, thời điểm hoàn tất
- Liên kết sang view hồ sơ BTC tương ứng (nếu có)

---

## Dữ liệu nhập (Input Fields)

Không có input trực tiếp trên screen chính.

### Bộ lọc lịch sử

| Trường | Loại | Ghi chú |
|--------|------|---------|
| Năm | Select | Tùy chọn |
| Trạng thái | Select | Tùy theo loại lịch sử (Organizer: PUBLISHED/COMPLETED/ARCHIVED; Sponsor: COMPLETED/ARCHIVED) |
| Hình thức tài trợ | Select | Chỉ áp dụng cho Sponsor history (CASH/IN_KIND/MIXED) |

---

## Hành động chính (Primary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Chọn tab lịch sử phù hợp | Hiển thị danh sách lịch sử theo vai trò duy nhất của tổ chức |
| Áp dụng bộ lọc | Làm mới danh sách lịch sử |

## Hành động phụ (Secondary Actions)

| Hành động | Đích đến / Kết quả |
|-----------|---------------------|
| Xem thêm (load more) | Phân trang — tải 5 mục tiếp theo |
| Nhấn vào mục lịch sử | Đi sang view chi tiết sự kiện / hồ sơ BTC tương ứng (nếu có và đủ điều kiện public) |
| Quay lại | Trở về màn trước hoặc danh sách nguồn |

---

## Quy tắc nghiệp vụ (Business Rules)

- BR-1101: Chỉ tổ chức VERIFIED mới hiển thị màn này. Truy cập tổ chức chưa VERIFIED → 404
- BR-1102: Không hiển thị draft, thương thảo nội bộ, điều khoản hợp đồng, hoặc dữ liệu nhạy cảm
- BR-1103: Danh sách history tối đa 5 mục gần nhất, page size cố định, hỗ trợ load more
- BR-1104: Chỉ hiển thị một loại history theo vai trò duy nhất của tổ chức
- BR-1105: Reputation summary là read-only, KHÔNG có link sang SCR-018 cho Guest
- BR-1106: Không hiển thị giá trị tài trợ, điều khoản hợp đồng; có hiển thị tên sự kiện, năm, hình thức, trạng thái, nhãn tóm tắt

---

## Quy tắc xác thực (Validation Rules)

- organization_id phải tồn tại và có verification_status = VERIFIED
- Bộ lọc lịch sử phải hợp lệ theo role duy nhất của tổ chức
- Không cho phép người dùng truy cập data private thông qua query parameter

---

## Điều hướng đến (Navigation In)

| Từ | Thông qua |
|----|-----------|
| Link chia sẻ / URL public | Truy cập trực tiếp |
| Màn tìm kiếm / danh sách nội bộ | Nhấn vào tên tổ chức |
| SCR-005, SCR-006 | Nhấn vào tên tổ chức trong view chi tiết |

## Điều hướng đi (Navigation Out)

| Khi | Đến |
|-----|-----|
| Nhấn vào mục lịch sử có link | View chi tiết sự kiện hoặc hồ sơ BTC tương ứng |
| Quay lại | Màn nguồn trước đó |

> **Ghi chú**: KHÔNG có navigation out từ SCR-026 sang SCR-018 cho Guest. Authenticated User truy cập SCR-018 từ các luồng khác (SCR-005, SCR-006) trong hệ thống (SF-07).

---

## UI States liên quan

| State | Điều kiện kích hoạt |
|-------|---------------------|
| Loading State | Đang tải hồ sơ public |
| Empty History State | Không có lịch sử công khai |
| No Reviews State | Tổ chức chưa có đánh giá (hiển thị "Chưa có đánh giá") |
| Organization Not Available | Không tồn tại hoặc chưa VERIFIED → 404 |
| Role-Based History State | Tự hiển thị loại lịch sử phù hợp với vai trò duy nhất |
| Filter Active State | Bộ lọc đang được áp dụng |
| Filter No Results State | Bộ lọc không trả về kết quả |
| Load More Available | Có nhiều hơn 5 mục, hiển thị nút "Xem thêm" |

## UI Components liên quan

- Profile header
- Verification badge (luôn hiển thị VERIFIED)
- Reputation summary card (read-only, no link to SCR-018)
- History section (single type based on role)
- History cards / list items
- Filter bar
- Load more button
- Empty states (history, reviews)

---

## UC-to-Screen Mapping

| UC ID | Flow Step | Mô tả | Classification | Phần tử trong screen |
|-------|-----------|--------|----------------|---------------------|
| UC-46 | Main-1 | Mở trang public profile | Screen entry | Page load |
| UC-46 | Main-2 | Kiểm tra VERIFIED | Action | System check / 404 |
| UC-46 | Main-3 | Hiển thị nhận diện tổ chức | Read-only data | Profile header |
| UC-46 | Main-4 | Hiển thị tóm tắt uy tín (read-only) | Read-only data | Reputation summary card |
| UC-46 | Main-5 | Hiển thị khu vực lịch sử theo vai trò | Read-only data | History section |
| UC-46 | AF-46.a | Không có lịch sử | UI State | Empty history state |
| UC-46 | AF-46.b | Chưa có đánh giá | UI State | No reviews state |
| UC-46 | EF-46.1 | Không tồn tại / chưa VERIFIED | UI State | Error / 404 |
| UC-47 | Main-1 | Chọn tab lịch sử tài trợ (Organizer) | Action | History section |
| UC-47 | Main-2~5 | Hiển thị danh sách public sponsorship history | Read-only data | History cards |
| UC-47 | Main-6 | Liên kết sang view sự kiện | Navigation out | Link in card |
| UC-47 | AF-47.a | Không có lịch sử | UI State | Empty state |
| UC-47 | AF-47.b | Xem thêm (load more) | Action | Load more button |
| UC-47 | AF-47.c | Áp dụng bộ lọc | Action | Filter bar |
| UC-48 | Main-1 | Chọn tab lịch sử giao dịch (Sponsor) | Action | History section |
| UC-48 | Main-2~5 | Hiển thị danh sách public transaction history | Read-only data | History cards |
| UC-48 | Main-6 | Liên kết sang view hồ sơ BTC | Navigation out | Link in card |
| UC-48 | AF-48.a | Không có lịch sử | UI State | Empty state |
| UC-48 | AF-48.b | Xem thêm (load more) | Action | Load more button |
| UC-48 | AF-48.c | Áp dụng bộ lọc | Action | Filter bar |
