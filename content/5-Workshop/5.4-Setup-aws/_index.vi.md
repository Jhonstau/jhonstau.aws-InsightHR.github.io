---
title: "Thiết lập Môi trường AWS"
date: "2025-12-09"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Thiết lập Môi trường AWS

Phần này hướng dẫn bạn cấu hình môi trường AWS cho nền tảng InsightHR, bao gồm IAM roles, policies và cấu hình dịch vụ ban đầu.

### Tổng quan

Chúng ta sẽ thiết lập:

1. IAM roles cho Lambda functions
2. IAM policies cho quyền truy cập dịch vụ
3. Cấu hình AWS region
4. Xác minh service quotas
5. Quyền truy cập mô hình Bedrock

### Bước 1: Cấu hình AWS Region

Đặt region mặc định là Singapore (ap-southeast-1):

```bash
# Cấu hình AWS CLI
aws configure set region ap-southeast-1

# Xác minh cấu hình
aws configure get region
```

**Tại sao Singapore?**
- Tất cả các dịch vụ cần thiết đều có sẵn
- Độ trễ tốt cho Đông Nam Á
- Bedrock Claude 3 Haiku có sẵn

### Bước 2: Tạo IAM Role cho Lambda

Lambda functions cần execution role để truy cập các dịch vụ AWS.

#### Tạo Trust Policy

**lambda-trust-policy.json**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### Tạo Role

```bash
# Tạo IAM role
aws iam create-role \
    --role-name insighthr-lambda-role \
    --assume-role-policy-document file://lambda-trust-policy.json \
    --description "Execution role for InsightHR Lambda functions"
```

### Bước 3: Tạo IAM Policies

#### DynamoDB Access Policy

**dynamodb-policy.json**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:BatchGetItem",
        "dynamodb:BatchWriteItem"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:*:table/insighthr-*"
      ]
    }
  ]
}
```

Tạo policy:
```bash
aws iam create-policy \
    --policy-name insighthr-dynamodb-policy \
    --policy-document file://dynamodb-policy.json
```

#### Cognito Access Policy

**cognito-policy.json**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cognito-idp:AdminInitiateAuth",
        "cognito-idp:AdminCreateUser",
        "cognito-idp:AdminSetUserPassword",
        "cognito-idp:AdminGetUser",
        "cognito-idp:AdminUpdateUserAttributes",
        "cognito-idp:ListUsers"
      ],
      "Resource": "arn:aws:cognito-idp:ap-southeast-1:*:userpool/*"
    }
  ]
}
```

Tạo policy:
```bash
aws iam create-policy \
    --policy-name insighthr-cognito-policy \
    --policy-document file://cognito-policy.json
```

#### Bedrock Access Policy

**bedrock-policy.json**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel"
      ],
      "Resource": "arn:aws:bedrock:ap-southeast-1::foundation-model/anthropic.claude-3-haiku-20240307-v1:0"
    }
  ]
}
```

Tạo policy:
```bash
aws iam create-policy \
    --policy-name insighthr-bedrock-policy \
    --policy-document file://bedrock-policy.json
```

### Bước 4: Gắn Policies vào Role

```bash
# Lấy AWS account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Gắn AWS managed policy cho Lambda basic execution
aws iam attach-role-policy \
    --role-name insighthr-lambda-role \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Gắn custom policies
aws iam attach-role-policy \
    --role-name insighthr-lambda-role \
    --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/insighthr-dynamodb-policy

aws iam attach-role-policy \
    --role-name insighthr-lambda-role \
    --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/insighthr-cognito-policy

aws iam attach-role-policy \
    --role-name insighthr-lambda-role \
    --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/insighthr-bedrock-policy
```

### Bước 5: Tạo IAM Role cho API Gateway

API Gateway cần role để ghi logs vào CloudWatch.

**api-gateway-trust-policy.json**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "apigateway.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Tạo role:
```bash
aws iam create-role \
    --role-name insighthr-apigateway-role \
    --assume-role-policy-document file://api-gateway-trust-policy.json

# Gắn CloudWatch logs policy
aws iam attach-role-policy \
    --role-name insighthr-apigateway-role \
    --policy-arn arn:aws:iam::aws:policy/service-role/AmazonAPIGatewayPushToCloudWatchLogs
```

### Bước 6: Bật Quyền truy cập Mô hình Bedrock

Yêu cầu quyền truy cập mô hình Claude 3 Haiku:

**Sử dụng AWS Console**:

1. Điều hướng đến Amazon Bedrock
2. Click "Model access" ở thanh bên trái
3. Click "Manage model access"
4. Tìm "Claude 3 Haiku" của Anthropic
5. Đánh dấu vào ô
6. Click "Request model access"
7. Đợi phê duyệt (thường là ngay lập tức)

**Xác minh Quyền truy cập**:
```bash
aws bedrock list-foundation-models \
    --region ap-southeast-1 \
    --query 'modelSummaries[?contains(modelId, `claude-3-haiku`)].modelId'
```

### Bước 7: Xác minh Service Quotas

Kiểm tra giới hạn dịch vụ cho tài khoản của bạn:

```bash
# Lambda concurrent executions
aws service-quotas get-service-quota \
    --service-code lambda \
    --quota-code L-B99A9384 \
    --region ap-southeast-1

# DynamoDB tables
aws service-quotas get-service-quota \
    --service-code dynamodb \
    --quota-code L-F98FE922 \
    --region ap-southeast-1

# API Gateway requests per second
aws service-quotas get-service-quota \
    --service-code apigateway \
    --quota-code L-8A5B8E43 \
    --region ap-southeast-1
```

### Bước 8: Tạo S3 Bucket cho Deployment Artifacts

```bash
# Tạo bucket cho Lambda deployment packages
aws s3 mb s3://insighthr-deployment-artifacts-${ACCOUNT_ID} \
    --region ap-southeast-1

# Bật versioning
aws s3api put-bucket-versioning \
    --bucket insighthr-deployment-artifacts-${ACCOUNT_ID} \
    --versioning-configuration Status=Enabled
```

### Bước 9: Thiết lập CloudWatch Log Groups

Tạo trước log groups cho Lambda functions:

```bash
# Tạo log groups
LOG_GROUPS=(
    "/aws/lambda/insighthr-auth-login-handler"
    "/aws/lambda/insighthr-auth-register-handler"
    "/aws/lambda/insighthr-auth-google-handler"
    "/aws/lambda/insighthr-employees-handler"
    "/aws/lambda/insighthr-performance-scores-handler"
    "/aws/lambda/insighthr-chatbot-handler"
    "/aws/lambda/insighthr-attendance-handler"
)

for log_group in "${LOG_GROUPS[@]}"; do
    aws logs create-log-group \
        --log-group-name $log_group \
        --region ap-southeast-1
    
    # Đặt retention là 7 ngày
    aws logs put-retention-policy \
        --log-group-name $log_group \
        --retention-in-days 7 \
        --region ap-southeast-1
done
```

### Bước 10: Cấu hình Biến Môi trường

Tạo file cấu hình cho biến môi trường:

**config.env**:
```bash
# AWS Configuration
export AWS_REGION=ap-southeast-1
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# DynamoDB Tables
export USERS_TABLE=insighthr-users-dev
export EMPLOYEES_TABLE=insighthr-employees-dev
export PERFORMANCE_SCORES_TABLE=insighthr-performance-scores-dev
export ATTENDANCE_HISTORY_TABLE=insighthr-attendance-history-dev

# Cognito (sẽ được đặt sau khi thiết lập Cognito)
export USER_POOL_ID=
export CLIENT_ID=

# Bedrock
export BEDROCK_MODEL_ID=anthropic.claude-3-haiku-20240307-v1:0
export BEDROCK_REGION=ap-southeast-1

# Lambda Role ARN
export LAMBDA_ROLE_ARN=arn:aws:iam::${AWS_ACCOUNT_ID}:role/insighthr-lambda-role
```

Load cấu hình:
```bash
source config.env
```

### Checklist Xác minh

Xác minh môi trường AWS của bạn đã sẵn sàng:

```bash
# Kiểm tra IAM role tồn tại
aws iam get-role --role-name insighthr-lambda-role

# Kiểm tra policies đã được gắn
aws iam list-attached-role-policies --role-name insighthr-lambda-role

# Kiểm tra quyền truy cập Bedrock
aws bedrock list-foundation-models \
    --region ap-southeast-1 \
    --query 'modelSummaries[?contains(modelId, `claude`)].modelId'

# Kiểm tra S3 bucket tồn tại
aws s3 ls s3://insighthr-deployment-artifacts-${ACCOUNT_ID}

# Kiểm tra CloudWatch log groups
aws logs describe-log-groups \
    --log-group-name-prefix "/aws/lambda/insighthr" \
    --region ap-southeast-1
```

### Tóm tắt Thiết lập IAM

| Tài nguyên | Mục đích | Policies Đã gắn |
|----------|---------|-------------------|
| insighthr-lambda-role | Lambda execution | AWSLambdaBasicExecutionRole, DynamoDB, Cognito, Bedrock |
| insighthr-apigateway-role | API Gateway logging | AmazonAPIGatewayPushToCloudWatchLogs |
| insighthr-dynamodb-policy | Truy cập DynamoDB | Custom policy cho thao tác bảng |
| insighthr-cognito-policy | Truy cập Cognito | Custom policy cho quản lý người dùng |
| insighthr-bedrock-policy | Truy cập Bedrock | Custom policy cho gọi mô hình |

### Best Practices Bảo mật

✅ **Least Privilege**: Policies chỉ cấp quyền cần thiết
✅ **Resource Restrictions**: Policies giới hạn cho tài nguyên insighthr-*
✅ **Separate Roles**: Các role khác nhau cho các dịch vụ khác nhau
✅ **CloudWatch Logging**: Tất cả Lambda functions ghi log vào CloudWatch
✅ **Versioning**: Bật versioning S3 bucket cho artifacts

### Xử lý Sự cố

**Tạo IAM Role Thất bại**:
- Kiểm tra quyền IAM
- Xác minh cú pháp trust policy
- Đảm bảo tên role là duy nhất

**Gắn Policy Thất bại**:
- Xác minh policy ARN đúng
- Kiểm tra role tồn tại
- Đảm bảo bạn có quyền iam:AttachRolePolicy

**Bedrock Access Denied**:
- Yêu cầu quyền truy cập mô hình trong Bedrock console
- Đợi phê duyệt (thường ngay lập tức cho Haiku)
- Xác minh region hỗ trợ Bedrock

**Vấn đề Service Quota**:
- Yêu cầu tăng quota trong Service Quotas console
- Sử dụng region khác nếu cần
- Liên hệ AWS support cho yêu cầu khẩn cấp

### Bước Tiếp theo

Với môi trường AWS đã được cấu hình, tiến hành [Thiết lập Database](../5.5-database-setup/) để tạo các bảng DynamoDB.
