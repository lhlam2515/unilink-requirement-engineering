# UC-26: Nộp bằng chứng hoàn thành nghĩa vụ

**Brief Description**
> Authenticated User (bên chịu trách nhiệm nghĩa vụ) ghi nhận việc hoàn thành nghĩa vụ bằng cách nộp bằng chứng bao gồm mô tả văn bản và/hoặc file đính kèm. Hệ thống chuyển trạng thái nghĩa vụ sang SUBMITTED và thông báo cho đối tác xác nhận.

---

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Bên chịu trách nhiệm — Organizer hoặc Sponsor) | Người nộp bằng chứng hoàn thành |
| Secondary | System | Lưu bằng chứng, chuyển trạng thái, gửi thông báo |

---

**Preconditions**

- Actor đã đăng nhập vào hệ thống
- Nghĩa vụ đang ở trạng thái PENDING, IN_PROGRESS, hoặc DISPUTED (đã bị từ chối, nộp lại)
- Actor là bên chịu trách nhiệm cho nghĩa vụ này (responsible_party)

---

**Trigger**
> Actor nhấn "Báo cáo hoàn thành" trên một nghĩa vụ cụ thể.

---

**Main Flow (Basic Path)**

| Step | Actor | Action / System Response |
|------|-------|--------------------------| 
| 1 | Authenticated User | Nhấn "Báo cáo hoàn thành" trên nghĩa vụ |
| 2 | System | Hiển thị form nộp bằng chứng: mô tả (bắt buộc), file đính kèm (tùy chọn) |
| 3 | Authenticated User | Nhập mô tả bằng chứng hoàn thành (tối thiểu 20 ký tự) |
| 4 | Authenticated User | Đính kèm file bằng chứng nếu có (ảnh biên lai, tài liệu, v.v.) |
| 5 | Authenticated User | Nhấn "Nộp" |
| 6 | System | Xác thực: mô tả ≥ 20 ký tự, file đính kèm hợp lệ (nếu có) |
| 7 | System | Lưu bằng chứng, chuyển nghĩa vụ sang trạng thái SUBMITTED |
| 8 | System | Ghi nhận submitted_at và submitted_by |
| 9 | System | Gửi thông báo cho đối tác "Bên [tên] đã báo cáo hoàn thành nghĩa vụ [tên nghĩa vụ]" |
| 10 | System | Use case kết thúc thành công — chờ đối tác xác nhận (UC-27) |

---

**Alternate Flows**

> AF-26.a: Nộp lại bằng chứng sau khi bị từ chối (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện nghĩa vụ ở trạng thái DISPUTED (đã bị từ chối) |
| 1b | System | Hiển thị lý do từ chối trước đó |
| 1c | Authenticated User | Xem lý do từ chối và chuẩn bị bằng chứng mới |
| 1d | System | Tiếp tục tại Step 2 — cho phép nộp bằng chứng mới |

---

**Exception Flows**

> EF-26.1: Nghĩa vụ đã CONFIRMED (triggered at Step 1)

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện nghĩa vụ ở trạng thái CONFIRMED |
| 1b | System | Từ chối "Nghĩa vụ đã được xác nhận hoàn thành" |

> EF-26.2: Mô tả quá ngắn (triggered at Step 6)

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện mô tả dưới 20 ký tự |
| 6b | System | Từ chối "Mô tả bằng chứng phải có ít nhất 20 ký tự" |
| 6c | Authenticated User | Nhập mô tả chi tiết hơn |

---

**Postconditions**

*Success:*
- Nghĩa vụ chuyển sang trạng thái SUBMITTED
- Bằng chứng (mô tả + file) được lưu trữ
- Đối tác được thông báo để xác nhận (UC-27)

*Failure:*
- Nghĩa vụ không thay đổi trạng thái
- Actor được thông báo lỗi

---

**Business Rules**

- BR-0602: Bằng chứng PHẢI bao gồm mô tả ≥ 20 ký tự. File đính kèm tùy chọn nhưng khuyến khích. Nghĩa vụ CONFIRMED không thể nộp lại

---

**Notes / Assumptions**

- Nếu đối tác từ chối (DISPUTED), actor có thể nộp bằng chứng mới (flow quay lại)
- Đối tác xác nhận hoàn thành thông qua UC-27
- Liên kết: UC-25, UC-27
