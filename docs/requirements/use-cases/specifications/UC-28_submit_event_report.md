# UC-28: Nộp báo cáo kết quả sự kiện

**Brief Description**
> Organizer nộp báo cáo kết quả sự kiện cho sponsor sau khi sự kiện kết thúc, bao gồm tóm tắt, số liệu thực tế, hình ảnh/video, đánh giá mức độ hoàn thành quyền lợi nhà tài trợ, và tài liệu đính kèm.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Organizer (BTC) | Người nộp báo cáo |
| Secondary | System | Lưu trữ, gửi thông báo cho sponsor |

---

**Preconditions**

- Organizer đã đăng nhập vào hệ thống
- Hợp đồng đang ở trạng thái SIGNED
- Chưa có báo cáo kết quả cho hợp đồng này

---

**Trigger**
> Organizer nhấn "Nộp báo cáo kết quả" trong trang hợp đồng.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Organizer | Nhấn "Nộp báo cáo kết quả" trên trang hợp đồng |
| 2 | System | Hiển thị form báo cáo: tóm tắt, lượng khán giả thực tế, reach truyền thông, ghi chú hoàn thành quyền lợi, ảnh/video sự kiện, file đính kèm |
| 3 | Organizer | Nhập tóm tắt kết quả sự kiện |
| 4 | Organizer | Nhập số liệu thực tế: lượng khán giả, reach truyền thông |
| 5 | Organizer | Tải lên hình ảnh/video sự kiện |
| 6 | Organizer | Nhập đánh giá mức độ hoàn thành quyền lợi nhà tài trợ |
| 7 | Organizer | Đính kèm file báo cáo chi tiết (nếu có) |
| 8 | Organizer | Nhấn "Nộp báo cáo" |
| 9 | System | Lưu báo cáo kết quả sự kiện, gắn vào hợp đồng |
| 10 | System | Ghi nhận submitted_at và submitted_by |
| 11 | System | Gửi thông báo cho sponsor "BTC đã nộp báo cáo kết quả sự kiện" |
| 12 | System | Use case kết thúc thành công |

---

**Alternate Flows**

Không có alternate flow cho use case này.

---

**Exception Flows**

> EF-28.1: Hợp đồng chưa ký (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện hợp đồng chưa ở trạng thái SIGNED |
| 1b | System | Từ chối "Chỉ có thể nộp báo cáo cho hợp đồng đã ký" |

> EF-28.2: Đã có báo cáo cho hợp đồng (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện đã tồn tại báo cáo kết quả cho hợp đồng |
| 1b | System | Từ chối "Hợp đồng đã có báo cáo kết quả. Bạn có thể chỉnh sửa báo cáo hiện tại." |

---

**Postconditions**

*Success:*
- Báo cáo kết quả sự kiện được lưu và gắn vào hợp đồng
- Sponsor được thông báo và có thể xem báo cáo
- Sponsor có thể sử dụng thông tin này khi đánh giá (UC-29)

*Failure:*
- Báo cáo không được lưu
- Organizer được thông báo lỗi

---

**Business Rules**

- BR-0604: Báo cáo kết quả chỉ nộp cho hợp đồng SIGNED. Mỗi hợp đồng chỉ có MỘT báo cáo

---

**Notes / Assumptions**

- Báo cáo là nghĩa vụ của BTC (organizer) theo hợp đồng
- Hệ thống gửi nhắc nhở nộp báo cáo tự động (FR-0606)
- Sponsor xem báo cáo để đánh giá chất lượng hợp tác (UC-29)
- Liên kết: UC-25, UC-29
