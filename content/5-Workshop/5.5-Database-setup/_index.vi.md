---
title: "Thiết lập Database (DynamoDB)"
date: "2025-12-09"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Thiết lập Database DynamoDB

Trong phần này, bạn sẽ tạo và cấu hình các bảng DynamoDB cần thiết cho nền tảng InsightHR.

### Tổng quan

InsightHR sử dụng Amazon DynamoDB, một dịch vụ cơ sở dữ liệu NoSQL được quản lý hoàn toàn, cung cấp hiệu suất nhanh và có thể dự đoán với khả năng mở rộng liền mạch. Chúng ta sẽ tạo 4 bảng chính:

1. **Users Table** - Xác thực và hồ sơ người dùng
2. **Employees Table** - Dữ liệu chính của nhân viên
3. **Performance Scores Table** - Theo dõi hiệu suất theo quý
4. **Attendance History Table** - Bản ghi check-in/check-out

### Nguyên tắc Thiết kế Bảng

**Best Practices DynamoDB**:
- Sử dụng partition keys để phân phối dữ liệu đều
- Thêm sort keys cho truy vấn phạm vi
- Tạo GSIs cho các mẫu truy vấn thay thế
- Denormalize dữ liệu cho hiệu suất đọc
- Sử dụng thanh toán on-demand cho workload biến đổi

### Bước 1: Tạo Users Table

**Cấu hình Bảng**:
- **Tên Bảng**: `insighthr-users-dev`
- **Partition Key**: `userId` (String)
- **Billing Mode**: On-demand

**Sử dụng AWS Console**:

1. Điều hướng đến DynamoDB trong AWS Console
2. Click "Create table"
3. Nhập tên bảng: `insighthr-users-dev`
4. Partition key: `userId` (String)
5. Table settings: Sử dụng cài đặt mặc định
6. Billing mode: On-demand
7. Click "Create table"

**Thêm Global Secondary Index**:

1. Chọn bảng
2. Đi đến tab "Indexes"
3. Click "Create index"
4. Index name: `email-index`
5. Partition key: `email` (String)
6. Projected attributes: All
7. Click "Create index"

**Sử dụng AWS CLI**:

```bash
# Tạo bảng
aws dynamodb create-table \
    --table-name insighthr-users-dev \
    --attribute-definitions \
        AttributeName=userId,AttributeType=S \
        AttributeName=email,AttributeType=S \
    --key-schema \
        AttributeName=userId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1

# Tạo GSI
aws dynamodb update-table \
    --table-name insighthr-users-dev \
    --attribute-definitions \
        AttributeName=email,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"email-index\",\"KeySchema\":[{\"AttributeName\":\"email\",\"KeyType\":\"HASH\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1
```

**Cấu trúc Dữ liệu Mẫu**:
```json
{
  "userId": "user-123",
  "email": "admin@example.com",
  "name": "Admin User",
  "role": "Admin",
  "employeeId": "DEV-001",
  "department": "DEV",
  "isActive": true,
  "createdAt": "2025-12-09T00:00:00Z",
  "updatedAt": "2025-12-09T00:00:00Z"
}
```

### Bước 2: Tạo Employees Table

**Cấu hình Bảng**:
- **Tên Bảng**: `insighthr-employees-dev`
- **Partition Key**: `employeeId` (String)
- **Billing Mode**: On-demand

**Sử dụng AWS CLI**:

```bash
# Tạo bảng
aws dynamodb create-table \
    --table-name insighthr-employees-dev \
    --attribute-definitions \
        AttributeName=employeeId,AttributeType=S \
        AttributeName=department,AttributeType=S \
    --key-schema \
        AttributeName=employeeId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1

# Tạo GSI cho truy vấn department
aws dynamodb update-table \
    --table-name insighthr-employees-dev \
    --attribute-definitions \
        AttributeName=department,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"department-index\",\"KeySchema\":[{\"AttributeName\":\"department\",\"KeyType\":\"HASH\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1
```

**Cấu trúc Dữ liệu Mẫu**:
```json
{
  "employeeId": "DEV-001",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "department": "DEV",
  "position": "Senior",
  "status": "active",
  "hireDate": "2024-01-01",
  "managerId": "DEV-000",
  "createdAt": "2025-12-09T00:00:00Z",
  "updatedAt": "2025-12-09T00:00:00Z"
}
```

### Bước 3: Tạo Performance Scores Table

**Cấu hình Bảng**:
- **Tên Bảng**: `insighthr-performance-scores-dev`
- **Partition Key**: `employeeId` (String)
- **Sort Key**: `period` (String)
- **Billing Mode**: On-demand

**Sử dụng AWS CLI**:

```bash
# Tạo bảng
aws dynamodb create-table \
    --table-name insighthr-performance-scores-dev \
    --attribute-definitions \
        AttributeName=employeeId,AttributeType=S \
        AttributeName=period,AttributeType=S \
        AttributeName=department,AttributeType=S \
    --key-schema \
        AttributeName=employeeId,KeyType=HASH \
        AttributeName=period,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1

# Tạo GSI cho truy vấn department-period
aws dynamodb update-table \
    --table-name insighthr-performance-scores-dev \
    --attribute-definitions \
        AttributeName=department,AttributeType=S \
        AttributeName=period,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"department-period-index\",\"KeySchema\":[{\"AttributeName\":\"department\",\"KeyType\":\"HASH\"},{\"AttributeName\":\"period\",\"KeyType\":\"RANGE\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1
```

**Cấu trúc Dữ liệu Mẫu**:
```json
{
  "scoreId": "score-123",
  "employeeId": "DEV-001",
  "period": "2025-Q1",
  "employeeName": "John Doe",
  "department": "DEV",
  "position": "Senior",
  "overallScore": 85.5,
  "kpiScores": {
    "KPI": 85.0,
    "completed_task": 88.0,
    "feedback_360": 83.5
  },
  "calculatedAt": "2025-12-09T00:00:00Z",
  "createdAt": "2025-12-09T00:00:00Z",
  "updatedAt": "2025-12-09T00:00:00Z"
}
```

### Bước 4: Tạo Attendance History Table

**Cấu hình Bảng**:
- **Tên Bảng**: `insighthr-attendance-history-dev`
- **Partition Key**: `employeeId` (String)
- **Sort Key**: `date` (String)
- **Billing Mode**: On-demand

**Sử dụng AWS CLI**:

```bash
# Tạo bảng
aws dynamodb create-table \
    --table-name insighthr-attendance-history-dev \
    --attribute-definitions \
        AttributeName=employeeId,AttributeType=S \
        AttributeName=date,AttributeType=S \
        AttributeName=department,AttributeType=S \
    --key-schema \
        AttributeName=employeeId,KeyType=HASH \
        AttributeName=date,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1

# Tạo GSI cho truy vấn date
aws dynamodb update-table \
    --table-name insighthr-attendance-history-dev \
    --attribute-definitions \
        AttributeName=date,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"date-index\",\"KeySchema\":[{\"AttributeName\":\"date\",\"KeyType\":\"HASH\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1

# Tạo GSI cho truy vấn department-date
aws dynamodb update-table \
    --table-name insighthr-attendance-history-dev \
    --attribute-definitions \
        AttributeName=department,AttributeType=S \
        AttributeName=date,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"department-date-index\",\"KeySchema\":[{\"AttributeName\":\"department\",\"KeyType\":\"HASH\"},{\"AttributeName\":\"date\",\"KeyType\":\"RANGE\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1
```

**Cấu trúc Dữ liệu Mẫu**:
```json
{
  "employeeId": "DEV-001",
  "date": "2025-12-09",
  "employeeName": "John Doe",
  "department": "DEV",
  "checkInTime": "09:00:00",
  "checkOutTime": "18:00:00",
  "status": "present",
  "notes": "On time",
  "createdAt": "2025-12-09T09:00:00Z",
  "updatedAt": "2025-12-09T18:00:00Z"
}
```

### Bước 5: Xác minh Tạo Bảng

**Sử dụng AWS Console**:
1. Điều hướng đến DynamoDB
2. Kiểm tra tất cả 4 bảng được liệt kê
3. Xác minh trạng thái mỗi bảng là "Active"
4. Kiểm tra GSIs đã được tạo và active

**Sử dụng AWS CLI**:

```bash
# Liệt kê tất cả bảng
aws dynamodb list-tables --region ap-southeast-1

# Mô tả bảng cụ thể
aws dynamodb describe-table \
    --table-name insighthr-users-dev \
    --region ap-southeast-1
```

### Bước 6: Load Dữ liệu Mẫu (Tùy chọn)

Để kiểm tra, bạn có thể load dữ liệu mẫu vào các bảng.

**Tạo file dữ liệu mẫu** (`sample-users.json`):
```json
{
  "insighthr-users-dev": [
    {
      "PutRequest": {
        "Item": {
          "userId": {"S": "user-001"},
          "email": {"S": "admin@example.com"},
          "name": {"S": "Admin User"},
          "role": {"S": "Admin"},
          "employeeId": {"S": "DEV-001"},
          "department": {"S": "DEV"},
          "isActive": {"BOOL": true},
          "createdAt": {"S": "2025-12-09T00:00:00Z"},
          "updatedAt": {"S": "2025-12-09T00:00:00Z"}
        }
      }
    }
  ]
}
```

**Load dữ liệu**:
```bash
aws dynamodb batch-write-item \
    --request-items file://sample-users.json \
    --region ap-southeast-1
```

### Tóm tắt Cấu hình Database

| Bảng | Partition Key | Sort Key | GSIs | Mục đích |
|-------|--------------|----------|------|---------|
| insighthr-users-dev | userId | - | email-index | Xác thực người dùng |
| insighthr-employees-dev | employeeId | - | department-index | Dữ liệu nhân viên |
| insighthr-performance-scores-dev | employeeId | period | department-period-index | Theo dõi hiệu suất |
| insighthr-attendance-history-dev | employeeId | date | date-index, department-date-index | Bản ghi chấm công |

### Best Practices Đã Triển khai

✅ **Thiết kế Partition Key**: Phân phối dữ liệu đều
✅ **Sort Keys**: Cho phép truy vấn phạm vi cho dữ liệu time-series
✅ **GSIs**: Hỗ trợ các mẫu truy vấn thay thế
✅ **On-Demand Billing**: Tiết kiệm chi phí cho workload biến đổi
✅ **Naming Convention**: Đặt tên bảng nhất quán với hậu tố môi trường

### Xử lý Sự cố

**Tạo Bảng Thất bại**:
- Kiểm tra quyền IAM cho DynamoDB
- Xác minh region đúng
- Đảm bảo tên bảng chưa tồn tại

**Tạo GSI Thất bại**:
- Đợi bảng ở trạng thái ACTIVE trước khi tạo GSI
- Kiểm tra định nghĩa thuộc tính khớp
- Xác minh quyền IAM

**Chi phí Cao**:
- Sử dụng thanh toán on-demand cho development
- Giám sát read/write capacity units
- Tối ưu các mẫu truy vấn

### Bước Tiếp theo

Với các bảng database đã được tạo, tiến hành [Dịch vụ Xác thực](../5.6-authentication/) để thiết lập Cognito và các Lambda functions xác thực.
