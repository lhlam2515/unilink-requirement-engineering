# Use-Case Specification: UC-26 — Nộp bằng chứng hoàn thành nghĩa vụ

---

### Actors

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Authenticated User (Bên chịu trách nhiệm — Organizer hoặc Sponsor) | Người nộp bằng chứng hoàn thành |
| Secondary | System | Lưu bằng chứng, chuyển trạng thái, gửi thông báo |

---

### 1. Brief Description

> Authenticated User (bên chịu trách nhiệm nghĩa vụ) ghi nhận việc hoàn thành nghĩa vụ bằng cách nộp bằng chứng bao gồm mô tả văn bản và/hoặc file đính kèm. Hệ thống chuyển trạng thái nghĩa vụ sang SUBMITTED và thông báo cho đối tác xác nhận.

---

### 2. Flow of Events

**Trigger**
> Actor nhấn "Báo cáo hoàn thành" trên một nghĩa vụ cụ thể.

#### 2.1 Basic Flow

| Step | Actor / System | Action |
|------|----------------|--------|
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

#### 2.2 Alternate Flows

##### AF-26.a: Nộp lại bằng chứng sau khi bị từ chối
>
> *Triggered at Step 1 of the Basic Flow when nghĩa vụ ở trạng thái DISPUTED.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện nghĩa vụ ở trạng thái DISPUTED (đã bị từ chối) |
| 1b | System | Hiển thị lý do từ chối trước đó |
| 1c | Authenticated User | Xem lý do từ chối và chuẩn bị bằng chứng mới |
| 1d | System | Tiếp tục tại Step 2 — cho phép nộp bằng chứng mới |

#### 2.3 Exception Flows

##### EF-26.1: Nghĩa vụ đã CONFIRMED
>
> *Triggered at Step 1 of the Basic Flow when nghĩa vụ đã được xác nhận.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 1a | System | Phát hiện nghĩa vụ ở trạng thái CONFIRMED |
| 1b | System | Từ chối "Nghĩa vụ đã được xác nhận hoàn thành" |

##### EF-26.2: Mô tả quá ngắn
>
> *Triggered at Step 6 of the Basic Flow when mô tả dưới 20 ký tự.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 6a | System | Phát hiện mô tả dưới 20 ký tự |
| 6b | System | Từ chối "Mô tả bằng chứng phải có ít nhất 20 ký tự" |
| 6c | Authenticated User | Nhập mô tả chi tiết hơn |

---

### 3. Subflows

None.

---

### 4. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-26-01 | Nộp bằng chứng thành công | Actor nộp mô tả + file; nghĩa vụ chuyển SUBMITTED |
| SC-26-02 | Nộp lại sau từ chối | Nghĩa vụ DISPUTED; actor xem lý do từ chối và nộp bằng chứng mới (AF-26.a) |

---

### 5. Preconditions

#### 5.1 Actor đã xác thực

- Actor đã đăng nhập vào hệ thống

#### 5.2 Nghĩa vụ đang hoạt động

- Nghĩa vụ đang ở trạng thái PENDING, IN_PROGRESS, hoặc DISPUTED (đã bị từ chối, nộp lại)

#### 5.3 Actor là bên chịu trách nhiệm

- Actor là bên chịu trách nhiệm cho nghĩa vụ này (responsible_party)

---

### 6. Postconditions

#### 6.1 Success

- Nghĩa vụ chuyển sang trạng thái SUBMITTED
- Bằng chứng (mô tả + file) được lưu trữ
- Đối tác được thông báo để xác nhận (UC-27)

#### 6.2 Failure

- Nghĩa vụ không thay đổi trạng thái
- Actor được thông báo lỗi

---

### 7. Extension Points

None identified.

---

### 8. Special Requirements

None identified.

---

### 9. Business Rules

- BR-0602: Bằng chứng PHẢI bao gồm mô tả ≥ 20 ký tự. File đính kèm tùy chọn nhưng khuyến khích. Nghĩa vụ CONFIRMED không thể nộp lại

---

### 10. Additional Information

**Assumptions:**

- Nếu đối tác từ chối (DISPUTED), actor có thể nộp bằng chứng mới (flow quay lại)
- Đối tác xác nhận hoàn thành thông qua UC-27

**Related Use Cases:**

- UC-25: Theo dõi trạng thái nghĩa vụ (`<<extend>>` base — UC-26 mở rộng UC-25)
- UC-27: Xác nhận hoàn thành nghĩa vụ (sequential — đối tác xác nhận)
