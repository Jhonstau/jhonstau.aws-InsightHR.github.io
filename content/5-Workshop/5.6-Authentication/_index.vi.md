---
title: "Dịch vụ Xác thực"
date: "2025-12-09"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Thiết lập Dịch vụ Xác thực

Phần này bao gồm thiết lập AWS Cognito cho xác thực người dùng và triển khai các Lambda functions xác thực.

### Tổng quan

Hệ thống xác thực bao gồm:

1. **AWS Cognito User Pool** - Quản lý và xác thực người dùng
2. **Login Handler** - Xác thực email/password
3. **Registration Handler** - Đăng ký người dùng mới
4. **Google OAuth Handler** - Tích hợp đăng nhập xã hội
5. **Password Reset** - Quy trình khôi phục mật khẩu

### Bước 1: Tạo Cognito User Pool

#### Sử dụng AWS Console

1. Điều hướng đến Amazon Cognito
2. Click "Create user pool"
3. Cấu hình trải nghiệm đăng nhập:
   - Sign-in options: Email
   - User name requirements: Cho phép địa chỉ email
   - Click "Next"

4. Cấu hình yêu cầu bảo mật:
   - Password policy: Cognito defaults
   - Multi-factor authentication: Optional
   - User account recovery: Chỉ Email
   - Click "Next"

5. Cấu hình trải nghiệm đăng ký:
   - Self-registration: Enabled
   - Required attributes: name, email
   - Click "Next"

6. Cấu hình phân phối tin nhắn:
   - Email provider: Gửi email với Cognito
   - FROM email address: no-reply@verificationemail.com
   - Click "Next"

7. Tích hợp ứng dụng của bạn:
   - User pool name: `insighthr-user-pool`
   - App client name: `insighthr-web-client`
   - Client secret: Không tạo
   - Authentication flows: ALLOW_USER_PASSWORD_AUTH, ALLOW_REFRESH_TOKEN_AUTH
   - Click "Next"

8. Xem lại và tạo

#### Sử dụng AWS CLI

```bash
# Tạo user pool
aws cognito-idp create-user-pool \
    --pool-name insighthr-user-pool \
    --policies '{
        "PasswordPolicy": {
            "MinimumLength": 8,
            "RequireUppercase": true,
            "RequireLowercase": true,
            "RequireNumbers": true,
            "RequireSymbols": false
        }
    }' \
    --auto-verified-attributes email \
    --username-attributes email \
    --schema '[
        {
            "Name": "email",
            "AttributeDataType": "String",
            "Required": true,
            "Mutable": true
        },
        {
            "Name": "name",
            "AttributeDataType": "String",
            "Required": true,
            "Mutable": true
        }
    ]' \
    --region ap-southeast-1

# Lấy user pool ID
USER_POOL_ID=$(aws cognito-idp list-user-pools \
    --max-results 10 \
    --query "UserPools[?Name=='insighthr-user-pool'].Id" \
    --output text \
    --region ap-southeast-1)

echo "User Pool ID: $USER_POOL_ID"
```

#### Tạo App Client

```bash
# Tạo app client
aws cognito-idp create-user-pool-client \
    --user-pool-id $USER_POOL_ID \
    --client-name insighthr-web-client \
    --no-generate-secret \
    --explicit-auth-flows ALLOW_USER_PASSWORD_AUTH ALLOW_REFRESH_TOKEN_AUTH ALLOW_USER_SRP_AUTH \
    --region ap-southeast-1

# Lấy client ID
CLIENT_ID=$(aws cognito-idp list-user-pool-clients \
    --user-pool-id $USER_POOL_ID \
    --query "UserPoolClients[?ClientName=='insighthr-web-client'].ClientId" \
    --output text \
    --region ap-southeast-1)

echo "Client ID: $CLIENT_ID"
```

### Bước 2: Cấu hình Cognito cho Development

Để phát triển, tắt yêu cầu xác minh email:

```bash
# Cập nhật user pool để tự động xác minh emails
aws cognito-idp update-user-pool \
    --user-pool-id $USER_POOL_ID \
    --auto-verified-attributes email \
    --region ap-southeast-1
```

### Bước 3: Tạo Login Lambda Function

#### Tạo Function Code

**auth_login_handler.py**: (Xem file gốc để biết code đầy đủ)

#### Tạo requirements.txt

```
boto3>=1.26.0
```

#### Đóng gói và Triển khai

```bash
# Tạo deployment package
mkdir -p lambda/auth/package
cd lambda/auth
pip install -r requirements.txt -t package/
cd package
zip -r ../auth-login-handler.zip .
cd ..
zip -g auth-login-handler.zip auth_login_handler.py

# Triển khai Lambda function
aws lambda create-function \
    --function-name insighthr-auth-login-handler \
    --runtime python3.11 \
    --role arn:aws:iam::${ACCOUNT_ID}:role/insighthr-lambda-role \
    --handler auth_login_handler.lambda_handler \
    --zip-file fileb://auth-login-handler.zip \
    --timeout 30 \
    --memory-size 256 \
    --environment Variables="{
        USER_POOL_ID=${USER_POOL_ID},
        CLIENT_ID=${CLIENT_ID},
        DYNAMODB_USERS_TABLE=insighthr-users-dev
    }" \
    --region ap-southeast-1
```

### Bước 4: Tạo Registration Lambda Function

Tương tự như login handler, tạo và triển khai registration handler với code tương ứng.

### Bước 5: Tạo API Gateway Endpoints

#### Tạo REST API

```bash
# Tạo API
API_ID=$(aws apigateway create-rest-api \
    --name "InsightHR API" \
    --description "InsightHR Backend API" \
    --region ap-southeast-1 \
    --query 'id' \
    --output text)

echo "API ID: $API_ID"
```

#### Tạo Cognito Authorizer

```bash
# Tạo authorizer
AUTHORIZER_ID=$(aws apigateway create-authorizer \
    --rest-api-id $API_ID \
    --name insighthr-cognito-authorizer \
    --type COGNITO_USER_POOLS \
    --provider-arns arn:aws:cognito-idp:ap-southeast-1:${ACCOUNT_ID}:userpool/${USER_POOL_ID} \
    --identity-source method.request.header.Authorization \
    --region ap-southeast-1 \
    --query 'id' \
    --output text)

echo "Authorizer ID: $AUTHORIZER_ID"
```

### Bước 6: Bật CORS

Cấu hình CORS cho các endpoints API Gateway để cho phép frontend gọi API.

### Bước 7: Triển khai API

```bash
# Tạo deployment
aws apigateway create-deployment \
    --rest-api-id $API_ID \
    --stage-name dev \
    --description "Initial deployment" \
    --region ap-southeast-1

# Lấy API endpoint
echo "API Endpoint: https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/dev"
```

### Bước 8: Kiểm tra Xác thực

#### Kiểm tra Đăng ký

```bash
curl -X POST \
    https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/dev/auth/register \
    -H "Content-Type: application/json" \
    -d '{
        "email": "test@example.com",
        "password": "Test123!",
        "name": "Test User",
        "role": "Employee",
        "department": "DEV"
    }'
```

#### Kiểm tra Đăng nhập

```bash
curl -X POST \
    https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/dev/auth/login \
    -H "Content-Type: application/json" \
    -d '{
        "email": "test@example.com",
        "password": "Test123!"
    }'
```

### Tóm tắt Luồng Xác thực

```
1. Người dùng gửi thông tin đăng nhập → Frontend
2. Frontend gọi /auth/login → API Gateway
3. API Gateway định tuyến đến Lambda → auth-login-handler
4. Lambda xác thực với Cognito → Cognito User Pool
5. Cognito trả về JWT tokens → Lambda
6. Lambda lấy dữ liệu người dùng → DynamoDB
7. Lambda trả về tokens + dữ liệu người dùng → Frontend
8. Frontend lưu tokens → localStorage
9. Các yêu cầu tiếp theo bao gồm JWT → Authorization header
```

### Best Practices Bảo mật

✅ **Password Policy**: Yêu cầu mật khẩu mạnh
✅ **JWT Tokens**: Access tokens có thời gian sống ngắn
✅ **Refresh Tokens**: Thời gian sống dài để làm mới token
✅ **Chỉ HTTPS**: Tất cả giao tiếp được mã hóa
✅ **CORS**: Cấu hình đúng cho domain frontend
✅ **No Secrets**: Client không sử dụng client secret

### Xử lý Sự cố

**Xác thực Cognito Thất bại**:
- Xác minh user pool ID và client ID
- Kiểm tra mật khẩu đáp ứng yêu cầu
- Đảm bảo người dùng đã được xác nhận

**Lambda Permission Denied**:
- Kiểm tra IAM role có quyền Cognito
- Xác minh Lambda execution role đã gắn

**Lỗi CORS**:
- Bật CORS trên API Gateway
- Kiểm tra Access-Control-Allow-Origin header
- Xác minh phương thức OPTIONS đã cấu hình

### Bước Tiếp theo

Với xác thực đã được cấu hình, tiến hành [Dịch vụ Backend](../5.7-backend-services/) để triển khai APIs nhân viên, hiệu suất và chatbot.
