---
title: "Dịch vụ Backend"
date: "2025-12-09"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Triển khai Dịch vụ Backend

Phần này bao gồm triển khai các Lambda functions backend cốt lõi cho quản lý nhân viên, theo dõi hiệu suất, chấm công và AI chatbot.

### Tổng quan

Chúng ta sẽ triển khai bốn dịch vụ chính:

1. **Employee Service** - Các thao tác CRUD cho hồ sơ nhân viên
2. **Performance Scores Service** - Theo dõi hiệu suất theo quý
3. **Attendance Service** - Quản lý check-in/check-out
4. **Chatbot Service** - Truy vấn ngôn ngữ tự nhiên được hỗ trợ bởi AI

### Employee Service

Dịch vụ nhân viên xử lý tất cả các thao tác liên quan đến nhân viên với lọc theo phòng ban và vị trí.

**Tính năng Chính**:
- Các thao tác CRUD (Create, Read, Update, Delete)
- Lọc theo phòng ban (DEV, QA, DAT, SEC, AI)
- Lọc theo vị trí (Junior, Mid, Senior, Lead, Manager)
- Lọc theo trạng thái (active, inactive)
- Tìm kiếm theo tên hoặc mã nhân viên
- Import hàng loạt từ CSV

**API Endpoints**:
```
GET    /employees              - Liệt kê tất cả nhân viên
GET    /employees/{id}         - Lấy nhân viên theo ID
POST   /employees              - Tạo nhân viên mới
PUT    /employees/{id}         - Cập nhật nhân viên
DELETE /employees/{id}         - Xóa nhân viên
POST   /employees/bulk         - Import hàng loạt từ CSV
```

### Performance Scores Service

Quản lý điểm hiệu suất theo quý với tính toán tự động.

**Tính năng Chính**:
- Tính toán điểm tự động (trung bình của KPI, completed_task, feedback_360)
- Lọc theo phòng ban và kỳ
- Truy cập dữ liệu dựa trên vai trò (Admin: tất cả, Manager: phòng ban, Employee: của riêng mình)
- Dữ liệu nhân viên được denormalize để tối ưu hiệu suất
- Import hàng loạt từ CSV

**API Endpoints**:
```
GET    /performance-scores                    - Liệt kê tất cả điểm
GET    /performance-scores/{id}/{period}      - Lấy điểm cụ thể
POST   /performance-scores                    - Tạo điểm
PUT    /performance-scores/{id}/{period}      - Cập nhật điểm
DELETE /performance-scores/{id}/{period}      - Xóa điểm
POST   /performance-scores/bulk               - Import hàng loạt
```

### Attendance Service

Theo dõi chấm công nhân viên với chức năng check-in/check-out.

**Tính năng Chính**:
- Check-in/check-out thời gian thực
- Lọc theo khoảng thời gian
- Lọc theo phòng ban
- Theo dõi trạng thái (present, absent, late, half-day)
- Thao tác hàng loạt

**API Endpoints**:
```
GET    /attendance                    - Liệt kê bản ghi chấm công
GET    /attendance/{id}/{date}        - Lấy bản ghi cụ thể
POST   /attendance/check-in           - Check in
POST   /attendance/check-out          - Check out
PUT    /attendance/{id}/{date}        - Cập nhật bản ghi
DELETE /attendance/{id}/{date}        - Xóa bản ghi
```

### Chatbot Service

Chatbot được hỗ trợ bởi AI sử dụng AWS Bedrock (Claude 3 Haiku).

**Tính năng Chính**:
- Tích hợp AWS Bedrock (Claude 3 Haiku)
- Xử lý truy vấn ngôn ngữ tự nhiên
- Xây dựng ngữ cảnh từ DynamoDB
- Lọc dữ liệu dựa trên vai trò
- Tiết kiệm chi phí ($0.0004 mỗi truy vấn)

**Truy vấn Được Hỗ trợ**:
- Thông tin nhân viên ("DEV-001 là ai?")
- Truy vấn hiệu suất ("Điểm trung bình cho Q1 2025 là bao nhiêu?")
- Thống kê phòng ban ("So sánh hiệu suất DEV và QA")
- Phân tích xu hướng ("Xu hướng hiệu suất qua các quý")

**API Endpoint**:
```
POST /chatbot/query
```

### Mẫu Triển khai

Tất cả các dịch vụ tuân theo mẫu này:

1. **Request Validation** - Xác thực tham số đầu vào
2. **Authorization** - Kiểm tra quyền người dùng dựa trên vai trò
3. **DynamoDB Operations** - Truy vấn hoặc cập nhật dữ liệu
4. **Response Formatting** - Trả về phản hồi JSON chuẩn hóa
5. **Error Handling** - Bắt và trả về lỗi phù hợp

### Triển khai

Triển khai tất cả các dịch vụ backend sử dụng các script triển khai Lambda được cung cấp trong tài liệu workshop. Mỗi dịch vụ được đóng gói với các dependencies của nó và triển khai lên AWS Lambda với các biến môi trường phù hợp.

### Kiểm tra

Kiểm tra mỗi dịch vụ sử dụng các script kiểm tra được cung cấp hoặc lệnh curl. Xác minh:
- Các thao tác CRUD hoạt động đúng
- Chức năng lọc và tìm kiếm
- Kiểm soát truy cập dựa trên vai trò
- Xử lý lỗi
- Hiệu suất và thời gian phản hồi

### Bước Tiếp theo

Với các dịch vụ backend đã được triển khai, tiến hành [Phát triển Frontend](../5.8-frontend/) để xây dựng ứng dụng React.
