---
title: "Dọn dẹp"
date: "2025-12-09"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Dọn dẹp Tài nguyên

Để tránh các khoản phí liên tục, điều quan trọng là phải xóa tất cả các tài nguyên AWS được tạo trong workshop này. Phần này cung cấp hướng dẫn từng bước để dọn dẹp.

{{% notice warning %}}
**Quan trọng**: Xóa tài nguyên là không thể hoàn tác. Đảm bảo bạn đã sao lưu bất kỳ dữ liệu nào bạn muốn giữ trước khi tiếp tục.
{{% /notice %}}

### Tổng quan Dọn dẹp

Chúng ta sẽ xóa tài nguyên theo thứ tự sau:

1. CloudFront Distribution
2. S3 Buckets
3. API Gateway
4. Lambda Functions
5. DynamoDB Tables
6. Cognito User Pool
7. CloudWatch Logs
8. IAM Roles và Policies
9. Route53 (nếu đã cấu hình)
10. ACM Certificates (nếu đã tạo)

### Bước 1: Xóa CloudFront Distribution

CloudFront distributions phải được vô hiệu hóa trước khi xóa.

**Sử dụng AWS Console**:

1. Điều hướng đến CloudFront
2. Chọn InsightHR distribution
3. Click "Disable"
4. Đợi trạng thái chuyển sang "Disabled" (có thể mất 15-20 phút)
5. Chọn lại distribution
6. Click "Delete"

**Sử dụng AWS CLI**:

```bash
# Lấy distribution ID
DIST_ID=$(aws cloudfront list-distributions \
    --query "DistributionList.Items[?Comment=='InsightHR CloudFront Distribution'].Id" \
    --output text)

# Vô hiệu hóa distribution
# (Xem hướng dẫn chi tiết trong file gốc)

# Xóa distribution
aws cloudfront delete-distribution --id $DIST_ID --if-match $ETAG
```

### Bước 2: Xóa S3 Buckets

**Làm trống và xóa bucket ứng dụng web**:

```bash
# Làm trống bucket
aws s3 rm s3://insighthr-web-app-sg --recursive

# Xóa bucket
aws s3 rb s3://insighthr-web-app-sg --force
```

### Bước 3: Xóa API Gateway

```bash
# Lấy API ID
API_ID=$(aws apigateway get-rest-apis \
    --query "items[?name=='InsightHR API'].id" \
    --output text \
    --region ap-southeast-1)

# Xóa API
aws apigateway delete-rest-api \
    --rest-api-id $API_ID \
    --region ap-southeast-1
```

### Bước 4: Xóa Lambda Functions

**Xóa tất cả Lambda functions**:

```bash
# Danh sách functions cần xóa
FUNCTIONS=(
    "insighthr-auth-login-handler"
    "insighthr-auth-register-handler"
    "insighthr-auth-google-handler"
    "insighthr-employees-handler"
    "insighthr-employees-bulk-handler"
    "insighthr-performance-scores-handler"
    "insighthr-chatbot-handler"
    "insighthr-attendance-handler"
)

# Xóa từng function
for func in "${FUNCTIONS[@]}"; do
    echo "Đang xóa $func..."
    aws lambda delete-function \
        --function-name $func \
        --region ap-southeast-1
done
```

### Bước 5: Xóa DynamoDB Tables

**Sử dụng AWS CLI**:

```bash
# Danh sách bảng cần xóa
TABLES=(
    "insighthr-users-dev"
    "insighthr-employees-dev"
    "insighthr-performance-scores-dev"
    "insighthr-attendance-history-dev"
)

# Xóa từng bảng
for table in "${TABLES[@]}"; do
    echo "Đang xóa $table..."
    aws dynamodb delete-table \
        --table-name $table \
        --region ap-southeast-1
done
```

### Bước 6: Xóa Cognito User Pool

```bash
# Lấy user pool ID
USER_POOL_ID=$(aws cognito-idp list-user-pools \
    --max-results 10 \
    --query "UserPools[?Name=='insighthr-user-pool'].Id" \
    --output text \
    --region ap-southeast-1)

# Xóa user pool
aws cognito-idp delete-user-pool \
    --user-pool-id $USER_POOL_ID \
    --region ap-southeast-1
```

### Bước 7: Xóa CloudWatch Logs

**Xóa log groups**:

```bash
# Liệt kê log groups
aws logs describe-log-groups \
    --log-group-name-prefix "/aws/lambda/insighthr" \
    --region ap-southeast-1

# Xóa từng log group
LOG_GROUPS=$(aws logs describe-log-groups \
    --log-group-name-prefix "/aws/lambda/insighthr" \
    --query 'logGroups[*].logGroupName' \
    --output text \
    --region ap-southeast-1)

for log_group in $LOG_GROUPS; do
    echo "Đang xóa $log_group..."
    aws logs delete-log-group \
        --log-group-name $log_group \
        --region ap-southeast-1
done
```

### Bước 8: Xóa IAM Roles và Policies

**Sử dụng AWS CLI**:

```bash
# Liệt kê roles
aws iam list-roles \
    --query 'Roles[?starts_with(RoleName, `insighthr`)].RoleName' \
    --output text

# Với mỗi role, tách policies và xóa
ROLES=$(aws iam list-roles \
    --query 'Roles[?starts_with(RoleName, `insighthr`)].RoleName' \
    --output text)

for role in $ROLES; do
    echo "Đang xử lý role $role..."
    
    # Tách managed policies
    POLICIES=$(aws iam list-attached-role-policies \
        --role-name $role \
        --query 'AttachedPolicies[*].PolicyArn' \
        --output text)
    
    for policy in $POLICIES; do
        aws iam detach-role-policy \
            --role-name $role \
            --policy-arn $policy
    done
    
    # Xóa role
    aws iam delete-role --role-name $role
done
```

### Xác minh

Sau khi dọn dẹp, xác minh tất cả tài nguyên đã được xóa:

```bash
# Kiểm tra S3 buckets
aws s3 ls | grep insighthr

# Kiểm tra Lambda functions
aws lambda list-functions \
    --query 'Functions[?starts_with(FunctionName, `insighthr`)].FunctionName' \
    --region ap-southeast-1

# Kiểm tra DynamoDB tables
aws dynamodb list-tables \
    --query 'TableNames[?starts_with(@, `insighthr`)]' \
    --region ap-southeast-1
```

### Xác minh Chi phí

Sau khi dọn dẹp, giám sát AWS billing của bạn:

1. Đi đến AWS Billing Dashboard
2. Kiểm tra "Bills" cho tháng hiện tại
3. Xác minh không có khoản phí liên tục cho các dịch vụ đã xóa
4. Kiểm tra "Cost Explorer" cho xu hướng

### Checklist Dọn dẹp

- [ ] CloudFront distribution đã vô hiệu hóa và xóa
- [ ] S3 buckets đã làm trống và xóa
- [ ] API Gateway đã xóa
- [ ] Tất cả Lambda functions đã xóa
- [ ] Tất cả DynamoDB tables đã xóa
- [ ] Cognito User Pool đã xóa
- [ ] CloudWatch log groups đã xóa
- [ ] CloudWatch Synthetics canaries đã xóa
- [ ] IAM roles và policies đã xóa
- [ ] Route53 hosted zone đã xóa (nếu đã tạo)
- [ ] ACM certificates đã xóa (nếu đã tạo)
- [ ] Billing dashboard đã kiểm tra các khoản phí liên tục

### Xử lý Sự cố Dọn dẹp

**CloudFront không xóa được**:
- Đảm bảo distribution đã được vô hiệu hóa hoàn toàn
- Đợi 15-20 phút sau khi vô hiệu hóa
- Kiểm tra các tài nguyên liên quan

**S3 bucket không xóa được**:
- Đảm bảo bucket hoàn toàn trống
- Kiểm tra các versioned objects
- Vô hiệu hóa versioning trước khi xóa

**IAM role không xóa được**:
- Tách tất cả managed policies trước
- Xóa tất cả inline policies
- Kiểm tra service-linked roles

### Ghi chú Cuối cùng

{{% notice tip %}}
**Best Practice**: Luôn dọn dẹp tài nguyên sau khi hoàn thành workshop để tránh các khoản phí không mong muốn. Thiết lập cảnh báo billing để thông báo cho bạn về bất kỳ chi phí liên tục nào.
{{% /notice %}}

### Chúc mừng!

Bạn đã hoàn thành thành công workshop InsightHR và dọn dẹp tất cả tài nguyên. Bạn đã học được:

- ✅ Thiết kế kiến trúc serverless
- ✅ AWS Lambda và API Gateway
- ✅ Mô hình hóa dữ liệu DynamoDB
- ✅ Xác thực Cognito
- ✅ Tích hợp AI AWS Bedrock
- ✅ Triển khai CloudFront CDN
- ✅ Nguyên tắc Infrastructure as Code
- ✅ Chiến lược tối ưu chi phí

Cảm ơn bạn đã tham gia workshop này!
