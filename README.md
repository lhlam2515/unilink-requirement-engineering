# UniLink - Kho Yêu Cầu Phần Mềm

## Tổng quan

Repository này được sử dụng để quản lý quy trình Requirement Engineering cho dự án UniLink.

UniLink là nền tảng kết nối:

- Các câu lạc bộ, đội nhóm sinh viên tại trường đại học (cần tìm nhà tài trợ cho sự kiện/hoạt động)
- Các doanh nghiệp, thương hiệu (muốn tiếp cận nhóm người dùng trẻ và tăng nhận diện thương hiệu)

Mục tiêu là tạo ra quy trình hợp tác tài trợ hiệu quả, minh bạch và có khả năng mở rộng.

## Mục đích repository

Repository đóng vai trò workspace tập trung để:

- Thu thập, phân tích và hoàn thiện yêu cầu
- Đồng bộ các bên liên quan về tầm nhìn sản phẩm và phạm vi
- Lưu trữ tài liệu có cấu trúc cho thiết kế và phát triển hệ thống
- Hỗ trợ quy trình Requirement Engineering có trợ giúp từ AI Agent

## Cấu trúc thư mục

```
.
├── .agents/
│   └── skills/
├── docs/
│   ├── business/
│   │   └── business-processes/
│   └── requirements/
│       ├── system-features/
│       └── use-cases/
│           └── specifications/
└── README.md
```

## Hỗ trợ AI Agent

Thư mục `.agents/skills` chứa các skill có thể tái sử dụng, giúp AI Agent:

- Phân tích yêu cầu
- Tạo đặc tả
- Kiểm tra tính nhất quán
- Hỗ trợ quy trình soạn thảo tài liệu

## Hướng dẫn tài liệu

### 1. Tài liệu nghiệp vụ (`docs/business`)

Nên bao gồm:

- Tầm nhìn và sứ mệnh
- Giá trị đề xuất
- Nhóm người dùng mục tiêu
- Mô hình kinh doanh
- Phân tích các bên liên quan

### 2. Tài liệu yêu cầu (`docs/requirements`)

Nên bao gồm:

- Functional Requirements (FR)
- Non-functional Requirements (NFR)
- User Stories / Use Cases
- Ràng buộc hệ thống
- Tiêu chí chấp nhận

## Quy trình đề xuất

1. Xác định bối cảnh nghiệp vụ trong `docs/business`.
2. Thu thập và đặc tả yêu cầu trong `docs/requirements`.
3. Sử dụng các skill AI Agent để:
   - Tinh chỉnh yêu cầu
   - Phát hiện xung đột, thiếu sót
   - Nâng cao độ rõ ràng và tính kiểm chứng
4. Cập nhật tài liệu liên tục theo tiến độ dự án.

## Lưu ý

- Giữ tài liệu theo cấu trúc module, dễ version và dễ review
- Ưu tiên sự rõ ràng hơn sự phức tạp
- Đảm bảo traceability giữa mục tiêu nghiệp vụ và yêu cầu hệ thống

## Bối cảnh dự án

UniLink hướng đến việc thu hẹp khoảng cách giữa cộng đồng sinh viên và doanh nghiệp, từ đó:

- Mở rộng cơ hội tài trợ cho sinh viên
- Tạo kênh tiếp cận marketing mục tiêu và xác thực hơn cho doanh nghiệp
