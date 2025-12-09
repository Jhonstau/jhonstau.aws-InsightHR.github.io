---
title: "Tổng quan Workshop"
date: "2025-12-09"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Tổng quan Nền tảng InsightHR

**InsightHR** là một nền tảng tự động hóa HR serverless toàn diện, minh họa phát triển ứng dụng cloud-native hiện đại trên AWS. Workshop này sẽ hướng dẫn bạn xây dựng một ứng dụng sẵn sàng production từ đầu.

### Phạm vi Dự án

Nền tảng InsightHR cung cấp:

- **Hệ thống Quản lý Nhân viên**: Các thao tác CRUD hoàn chỉnh cho hồ sơ nhân viên với bộ lọc nâng cao theo phòng ban, vị trí và trạng thái
- **Theo dõi Hiệu suất**: Điểm hiệu suất theo quý với tính toán tự động dựa trên KPI, nhiệm vụ hoàn thành và phản hồi 360 độ
- **Quản lý Chấm công**: Hệ thống check-in/check-out thời gian thực với theo dõi lịch sử và giám sát trạng thái
- **Chatbot hỗ trợ AI**: Giao diện truy vấn ngôn ngữ tự nhiên sử dụng AWS Bedrock (Claude 3 Haiku) cho thông tin dữ liệu thông minh
- **Dashboard Phân tích**: Trực quan hóa tương tác với biểu đồ, bảng và khả năng xuất dữ liệu
- **Kiểm soát Truy cập Dựa trên Vai trò**: Hệ thống truy cập ba cấp (Admin, Manager, Employee) với lọc dữ liệu phù hợp
- **Xác thực Bảo mật**: Tích hợp Email/password và Google OAuth qua AWS Cognito

### Tổng quan Kiến trúc

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Browser                             │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CloudFront CDN                                │
│              Custom Domain: insight-hr.io.vn                     │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    S3 Static Website                             │
│              React SPA (Vite + TypeScript)                       │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    API Gateway (REST)                            │
│                  Cognito Authorizer                              │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Lambda     │  │   Lambda     │  │   Lambda     │
│   Auth       │  │  Employees   │  │  Chatbot     │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       ▼                 ▼                 ▼
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
└─────────────────────────────────────────────────────────────────┘
```

### Stack Công nghệ

**Frontend:**
- React 18 với TypeScript
- Vite 7.2 (công cụ build)
- Tailwind CSS 3.4 (theme Frutiger Aero)
- Zustand 5.0 (quản lý state)
- React Hook Form + Zod (validation form)
- Recharts 3.4 (trực quan hóa dữ liệu)

**Backend:**
- Python 3.11 (Lambda runtime)
- AWS Lambda (serverless compute)
- AWS API Gateway (REST API)
- AWS DynamoDB (NoSQL database)
- AWS Cognito (authentication)
- AWS Bedrock (AI/ML)

**Infrastructure:**
- AWS S3 (static hosting)
- AWS CloudFront (CDN)
- AWS Route53 (DNS)
- AWS CloudWatch (monitoring)
- AWS IAM (security)

### Tính năng Chính

#### 1. Kiến trúc Serverless Hoàn toàn
- Không có EC2 instances cần quản lý
- Tự động scaling dựa trên nhu cầu
- Mô hình giá pay-per-use
- High availability tích hợp sẵn

#### 2. Stack Phát triển Hiện đại
- TypeScript cho type safety
- React cho UI responsive
- Python cho logic backend
- Nguyên tắc Infrastructure as Code

#### 3. Sẵn sàng Production
- Custom domain với SSL
- Giám sát CloudWatch
- Synthetic canaries cho testing
- Kiểm soát truy cập dựa trên vai trò

#### 4. Tiết kiệm Chi phí
- DynamoDB on-demand pricing
- Lambda free tier eligible
- Chi phí hàng tháng tối thiểu (~$2-5)
- Không có phí tài nguyên idle

### Cấu trúc Workshop

Workshop này được chia thành 11 modules:

1. **Tổng quan Workshop** (Hiện tại) - Hiểu phạm vi dự án
2. **Yêu cầu tiên quyết** - Thiết lập môi trường của bạn
3. **Kiến trúc Dự án** - Tìm hiểu sâu về thiết kế hệ thống
4. **Thiết lập Môi trường AWS** - Cấu hình tài khoản AWS và credentials
5. **Thiết lập Database** - Tạo và điền dữ liệu vào bảng DynamoDB
6. **Dịch vụ Xác thực** - Triển khai Cognito và auth Lambda functions
7. **Dịch vụ Backend** - Xây dựng employee, performance và chatbot APIs
8. **Phát triển Frontend** - Tạo ứng dụng React
9. **Triển khai** - Deploy lên S3 và CloudFront
10. **Kiểm thử & Giám sát** - Thiết lập CloudWatch và canaries
11. **Dọn dẹp** - Xóa tài nguyên để tránh phí

### Mục tiêu Học tập

Sau khi hoàn thành workshop này, bạn sẽ có thể:

- Thiết kế và triển khai kiến trúc serverless trên AWS
- Xây dựng RESTful APIs sử dụng Lambda và API Gateway
- Mô hình hóa dữ liệu hiệu quả trong DynamoDB
- Triển khai xác thực với AWS Cognito
- Tích hợp khả năng AI sử dụng AWS Bedrock
- Deploy static websites với S3 và CloudFront
- Giám sát ứng dụng với CloudWatch
- Áp dụng các phương pháp bảo mật tốt nhất với IAM
- Tối ưu hóa chi phí cho ứng dụng serverless

### Kiểm tra Yêu cầu tiên quyết

Trước khi tiếp tục, đảm bảo bạn có:

- ✅ Tài khoản AWS với quyền admin
- ✅ AWS CLI đã cài đặt và cấu hình
- ✅ Node.js 18+ và npm đã cài đặt
- ✅ Python 3.11+ đã cài đặt
- ✅ Hiểu biết cơ bản về React và TypeScript
- ✅ Quen thuộc với REST APIs
- ✅ Text editor hoặc IDE (khuyến nghị VS Code)

### Ước tính Chi phí

Chạy workshop này sẽ phát sinh chi phí tối thiểu:

| Dịch vụ | Ước tính Chi phí |
|---------|------------------|
| DynamoDB | $0.50/tháng |
| Lambda | Free tier |
| S3 + CloudFront | $1-2/tháng |
| API Gateway | $0.10/tháng |
| Bedrock | $0.0004/truy vấn |
| **Tổng cộng** | **$2-5/tháng** |

{{% notice warning %}}
Nhớ hoàn thành module dọn dẹp ở cuối để tránh phí tiếp tục.
{{% /notice %}}

### Bước tiếp theo

Sẵn sàng bắt đầu? Hãy chuyển đến phần [Yêu cầu tiên quyết](../5.2-prerequisite/) để thiết lập môi trường phát triển của bạn.
