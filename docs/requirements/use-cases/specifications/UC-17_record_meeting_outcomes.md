# UC-17: Ghi nhận kết quả cuộc họp

**Brief Description**
> Authenticated User (Organizer hoặc Sponsor) ghi nhận kết quả sau cuộc họp thương thảo vào notebook chung của deal, bao gồm tóm tắt nội dung, các quyết định đã thống nhất, và action items tiếp theo. Notebook được lưu trong deal context để tham khảo khi soạn hợp đồng; hệ thống không tạo nội dung tự động từ video meeting.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Organizer hoặc Sponsor) | Người ghi nhận kết quả |
| Secondary | System | Lưu trữ, hiển thị cho cả hai bên |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Cuộc họp đã ở trạng thái CONFIRMED (đã diễn ra hoặc đang diễn ra)
- Actor là một trong hai bên liên quan trong deal

---

**Trigger**
> Actor nhấn "Ghi nhận kết quả" sau cuộc họp hoặc trong trang chi tiết meeting.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Authenticated User | Mở trang chi tiết cuộc họp đã CONFIRMED |
| 2 | Authenticated User | Nhấn "Ghi nhận kết quả" |
| 3 | System | Hiển thị form ghi nhận: tóm tắt nội dung, quyết định đã thống nhất, action items |
| 4 | Authenticated User | Nhập tóm tắt nội dung cuộc họp |
| 5 | Authenticated User | Nhập danh sách quyết định đã thống nhất |
| 6 | Authenticated User | Nhập danh sách action items tiếp theo |
| 7 | Authenticated User | Nhấn "Lưu" |
| 8 | System | Lưu ghi chú cuộc họp, ghi nhận noted_by và noted_at |
| 9 | System | Hiển thị ghi chú cho cả hai bên trong deal context |
| 10 | System | Use case kết thúc thành công |

---

**Alternate Flows**

> AF-17.a: Chỉnh sửa ghi chú đã lưu (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Phát hiện cuộc họp đã có ghi chú trước đó |
| 2b | System | Hiển thị ghi chú hiện tại với khả năng chỉnh sửa |
| 2c | Authenticated User | Cập nhật nội dung ghi chú |
| 2d | System | Lưu thay đổi. Use case kết thúc |

---

**Exception Flows**

Không có exception flow đặc biệt cho use case này.

---

**Postconditions**

*Success:*
- Ghi chú cuộc họp được lưu và hiển thị cho cả hai bên
- Thông tin này có thể tham khảo khi soạn hợp đồng (SF-05)

*Failure:*
- Ghi chú không được lưu

---

**Business Rules**

- Không có business rule riêng cho use case này

---

**Notes / Assumptions**

- Cả hai bên đều có thể ghi nhận kết quả, nhưng mỗi meeting chỉ có MỘT notebook chung (bên ghi sau sẽ chỉnh sửa bản đã có)
- Notebook cuộc họp là nguồn tham khảo quan trọng cho giai đoạn soạn thảo hợp đồng
- Hệ thống không tạo nội dung tự động từ video meeting; actor tự nhập thông tin
- Liên kết: UC-15, UC-16, UC-20
