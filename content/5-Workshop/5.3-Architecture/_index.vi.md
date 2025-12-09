---
title: "Kiến trúc Dự án"
date: "2025-12-09"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Kiến trúc InsightHR Chi tiết

Phần này cung cấp cái nhìn chi tiết về kiến trúc nền tảng InsightHR, giải thích cách các dịch vụ AWS khác nhau hoạt động cùng nhau để tạo ra một ứng dụng serverless có khả năng mở rộng, bảo mật và tiết kiệm chi phí.

### Kiến trúc Tổng quan

```
┌─────────────────────────────────────────────────────────────────┐
│                         Trình duyệt                              │
│                    (React + TypeScript)                          │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CloudFront CDN                                │
│              - Các điểm biên toàn cầu                            │
│              - Kết thúc SSL/TLS                                  │
│              - Cache tài nguyên tĩnh                             │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    S3 Static Website                             │
│              - Lưu trữ React SPA                                 │
│              - Tài nguyên tĩnh (JS, CSS, hình ảnh)              │
└─────────────────────────────────────────────────────────────────┘
                         │ REST API calls
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    API Gateway (REST)                            │
│              - Định tuyến yêu cầu                                │
│              - Xác thực Cognito                                  │
│              - Chuyển đổi request/response                       │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┬───────────────┐
         ▼               ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Lambda     │  │   Lambda     │  │   Lambda     │  │   Lambda     │
│   Auth       │  │  Employees   │  │ Performance  │  │  Chatbot     │
│              │  │              │  │   Scores     │  │              │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        DynamoDB                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │  Users   │ │Employees │ │  Scores  │ │Attendance│          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AWS Bedrock                                   │
│              (Claude 3 Haiku Model)                              │
│              - Xử lý ngôn ngữ tự nhiên                           │
│              - Phản hồi nhận biết ngữ cảnh                       │
└─────────────────────────────────────────────────────────────────┘
```

### Phân tích Thành phần

#### 1. Tầng Frontend

**Amazon S3 + CloudFront**

- **S3 Bucket**: Lưu trữ React SPA dưới dạng file tĩnh
  - Cấu hình cho static website hosting
  - Lưu trữ HTML, JavaScript, CSS và tài nguyên hình ảnh
  - Bật versioning để có khả năng rollback

- **CloudFront Distribution**: CDN toàn cầu cho phân phối nội dung nhanh
  - Các điểm biên trên toàn thế giới cho độ trễ thấp
  - Chứng chỉ SSL/TLS từ ACM
  - Hỗ trợ tên miền tùy chỉnh (insight-hr.io.vn)
  - Chính sách cache để tối ưu hiệu suất
  - Origin Access Identity (OAI) cho bảo mật S3

**Ứng dụng React**
- Kiến trúc Single Page Application (SPA)
- Định tuyến phía client với React Router
- Quản lý state với Zustand
- TypeScript cho type safety
- Tailwind CSS cho styling

#### 2. Tầng API

**Amazon API Gateway (REST)**

- **Endpoints**: Thiết kế RESTful API
  - `/auth/*` - Endpoints xác thực
  - `/employees/*` - Quản lý nhân viên
  - `/performance-scores/*` - Theo dõi hiệu suất
  - `/attendance/*` - Quản lý chấm công
  - `/chatbot/*` - Truy vấn AI chatbot

- **Tính năng**:
  - Xác thực yêu cầu
  - Cấu hình CORS
  - Giới hạn tốc độ và throttling
  - Chuyển đổi request/response
  - API keys và usage plans

- **Xác thực**: Cognito User Pool Authorizer
  - Xác thực JWT token
  - Kiểm soát truy cập dựa trên vai trò
  - Tự động làm mới token

#### 3. Tầng Compute

**AWS Lambda Functions**

**Authentication Service** (`auth-*-handler`)
- Đăng nhập với Cognito
- Đăng ký người dùng
- Tích hợp Google OAuth
- Quy trình đặt lại mật khẩu
- Quản lý token

**Employee Service** (`employees-*-handler`)
- Các thao tác CRUD cho nhân viên
- Lọc theo phòng ban và vị trí
- Import hàng loạt từ CSV
- Chức năng tìm kiếm

**Performance Scores Service** (`performance-scores-handler`)
- Tính toán và quản lý điểm
- Theo dõi hiệu suất theo quý
- Tổng hợp KPI
- Truy cập dữ liệu dựa trên vai trò

**Chatbot Service** (`chatbot-handler`)
- Xử lý truy vấn ngôn ngữ tự nhiên
- Xây dựng ngữ cảnh từ DynamoDB
- Tích hợp Bedrock API
- Định dạng phản hồi

**Attendance Service** (`attendance-handler`)
- Thao tác check-in/check-out
- Theo dõi lịch sử chấm công
- Quản lý trạng thái
- Thao tác hàng loạt

**Cấu hình Lambda**:
- Runtime: Python 3.11
- Memory: 256-512 MB
- Timeout: 30-60 giây
- Biến môi trường cho cấu hình
- IAM roles với quyền tối thiểu

#### 4. Tầng Dữ liệu

**Amazon DynamoDB**

**Cấu trúc Bảng**:

1. **insighthr-users-dev**
   - Primary Key: `userId`
   - GSI: `email-index`
   - Mục đích: Xác thực người dùng và hồ sơ
   - Thuộc tính: email, name, role, employeeId, department

2. **insighthr-employees-dev**
   - Primary Key: `employeeId`
   - GSI: `department-index`
   - Mục đích: Dữ liệu chính của nhân viên
   - Thuộc tính: name, email, department, position, status

3. **insighthr-performance-scores-dev**
   - Primary Key: `employeeId`
   - Sort Key: `period`
   - GSI: `department-period-index`
   - Mục đích: Điểm hiệu suất theo quý
   - Thuộc tính: overallScore, kpiScores, calculatedAt

4. **insighthr-attendance-history-dev**
   - Primary Key: `employeeId`
   - Sort Key: `date`
   - GSI: `date-index`, `department-date-index`
   - Mục đích: Lịch sử check-in/check-out
   - Thuộc tính: checkInTime, checkOutTime, status

**Tính năng DynamoDB**:
- Chế độ thanh toán on-demand
- Point-in-time recovery
- Mã hóa khi lưu trữ
- Global secondary indexes cho truy vấn hiệu quả
- TTL cho tự động hết hạn dữ liệu (nếu cần)

#### 5. Tầng Xác thực

**Amazon Cognito User Pools**

- **Quản lý Người dùng**:
  - Xác thực email/password
  - Tích hợp Google OAuth
  - Thuộc tính người dùng (name, role, department)
  - Chính sách mật khẩu và hỗ trợ MFA

- **Quản lý Token**:
  - JWT tokens (ID, Access, Refresh)
  - Hết hạn và làm mới token
  - Custom claims cho roles

- **Tính năng Bảo mật**:
  - Yêu cầu độ phức tạp mật khẩu
  - Quy trình khôi phục tài khoản
  - Xác minh người dùng
  - Bảo vệ brute force

#### 6. Tầng AI/ML

**Amazon Bedrock (Claude 3 Haiku)**

- **Khả năng**:
  - Hiểu ngôn ngữ tự nhiên
  - Phản hồi nhận biết ngữ cảnh
  - Truy vấn và phân tích dữ liệu
  - Giao diện hội thoại

- **Tích hợp**:
  - Gọi từ Lambda function
  - Xây dựng ngữ cảnh từ dữ liệu DynamoDB
  - Lọc dữ liệu dựa trên vai trò
  - Định dạng và xác thực phản hồi

- **Tối ưu Chi phí**:
  - Mô hình Haiku tiết kiệm chi phí
  - Kỹ thuật prompt engineering hiệu quả
  - Cache phản hồi khi có thể

#### 7. Tầng Giám sát

**Amazon CloudWatch**

- **Logs**:
  - Logs của Lambda function
  - Logs truy cập API Gateway
  - Theo dõi lỗi và debug

- **Metrics**:
  - Số lượng yêu cầu API
  - Lượt gọi và thời gian Lambda
  - Dung lượng đọc/ghi DynamoDB
  - Tỷ lệ lỗi và độ trễ

- **Alarms**:
  - Cảnh báo tỷ lệ lỗi cao
  - Suy giảm hiệu suất
  - Cảnh báo ngưỡng chi phí

**CloudWatch Synthetics Canaries**:
- Kiểm tra luồng đăng nhập
- Tính khả dụng của dashboard
- Chức năng chatbot
- Tính toán điểm hiệu suất

### Ví dụ Luồng Dữ liệu

#### Luồng Đăng nhập

```
1. Người dùng nhập thông tin đăng nhập trong ứng dụng React
2. Ứng dụng React gọi API Gateway /auth/login
3. API Gateway định tuyến đến auth-login-handler Lambda
4. Lambda xác thực với Cognito
5. Cognito trả về JWT tokens
6. Lambda lưu phiên người dùng trong DynamoDB
7. Tokens được trả về ứng dụng React
8. Ứng dụng React lưu tokens trong localStorage
9. Các yêu cầu tiếp theo bao gồm JWT trong Authorization header
```

#### Luồng Truy vấn Nhân viên

```
1. Người dùng yêu cầu danh sách nhân viên trong ứng dụng React
2. Ứng dụng React gọi API Gateway /employees với filters
3. API Gateway xác thực JWT token với Cognito
4. Yêu cầu được định tuyến đến employees-handler Lambda
5. Lambda truy vấn bảng employees DynamoDB
6. Kết quả được lọc dựa trên vai trò người dùng
7. Dữ liệu được trả về ứng dụng React
8. Ứng dụng React hiển thị nhân viên trong bảng
```

#### Luồng Truy vấn Chatbot

```
1. Người dùng nhập câu hỏi trong giao diện chatbot
2. Ứng dụng React gọi API Gateway /chatbot/query
3. API Gateway xác thực JWT và định tuyến đến chatbot-handler
4. Lambda lấy dữ liệu liên quan từ DynamoDB
5. Lambda xây dựng ngữ cảnh và gọi Bedrock API
6. Bedrock xử lý truy vấn với Claude 3 Haiku
7. Phản hồi được định dạng và trả về ứng dụng React
8. Ứng dụng React hiển thị câu trả lời trong giao diện chat
```

### Kiến trúc Bảo mật

**Defense in Depth**:

1. **Bảo mật Mạng**:
   - Chỉ HTTPS (được thực thi bởi CloudFront)
   - API Gateway với AWS WAF (tùy chọn)
   - VPC endpoints cho kết nối riêng tư (tùy chọn)

2. **Xác thực & Phân quyền**:
   - Cognito cho xác thực người dùng
   - JWT tokens cho phân quyền API
   - Kiểm soát truy cập dựa trên vai trò (RBAC)
   - IAM roles với quyền tối thiểu

3. **Bảo mật Dữ liệu**:
   - Mã hóa khi lưu trữ (DynamoDB, S3)
   - Mã hóa khi truyền tải (TLS 1.2+)
   - Quản lý thông tin xác thực an toàn
   - Không có secrets được hardcode

4. **Bảo mật Ứng dụng**:
   - Xác thực đầu vào
   - Ngăn chặn SQL injection (NoSQL)
   - Bảo vệ XSS
   - Cấu hình CORS

### Khả năng Mở rộng & Hiệu suất

**Auto-Scaling**:
- Lambda: Tự động mở rộng concurrent execution
- DynamoDB: Chế độ dung lượng on-demand
- CloudFront: Mạng lưới biên toàn cầu
- API Gateway: Xử lý yêu cầu tự động

**Tối ưu Hiệu suất**:
- CloudFront caching cho tài nguyên tĩnh
- DynamoDB GSIs cho truy vấn hiệu quả
- Tối ưu Lambda function
- Cache phản hồi API

**Tính Sẵn sàng Cao**:
- Triển khai Multi-AZ (tự động)
- Phân phối toàn cầu CloudFront
- Sao chép DynamoDB
- Khả năng chịu lỗi Lambda

### Tối ưu Chi phí

**Chiến lược**:
- Giá on-demand cho workload biến đổi
- Sử dụng free tier Lambda
- CloudFront caching để giảm yêu cầu origin
- Tối ưu truy vấn DynamoDB
- Mô hình Bedrock Haiku tiết kiệm chi phí

**Chi phí Ước tính Hàng tháng** (Development):
- DynamoDB: $0.50
- Lambda: Free tier
- S3 + CloudFront: $1-2
- API Gateway: $0.10
- Bedrock: $0.0004 mỗi truy vấn
- **Tổng**: $2-5/tháng

### Bước Tiếp theo

Bây giờ bạn đã hiểu kiến trúc, hãy tiến hành [Thiết lập Môi trường AWS](../5.4-setup-aws/) để bắt đầu xây dựng nền tảng.
