---
title: "Triển khai"
date: "2025-12-09"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Triển khai InsightHR lên Production

Phần này bao gồm triển khai ứng dụng InsightHR hoàn chỉnh lên AWS, bao gồm triển khai frontend lên S3/CloudFront và các Lambda functions backend.

### Tổng quan Triển khai

Quy trình triển khai bao gồm:

1. **Triển khai Frontend**: Build và triển khai ứng dụng React lên S3
2. **Cấu hình CloudFront**: Thiết lập CDN cho phân phối toàn cầu
3. **Triển khai Lambda**: Triển khai tất cả các backend functions
4. **Cấu hình API Gateway**: Thiết lập các REST API endpoints
5. **Cấu hình Môi trường**: Cấu hình biến môi trường
6. **Thiết lập DNS** (Tùy chọn): Cấu hình tên miền tùy chỉnh

### Yêu cầu Trước

Trước khi triển khai, đảm bảo bạn có:

- ✅ Tất cả bảng DynamoDB đã được tạo
- ✅ Cognito User Pool đã được cấu hình
- ✅ IAM roles và policies đã được thiết lập
- ✅ AWS CLI đã được cấu hình
- ✅ Node.js và npm đã được cài đặt
- ✅ Python 3.11+ đã được cài đặt

### Bước 1: Triển khai Frontend

#### Build Ứng dụng React

Điều hướng đến thư mục frontend và build production bundle:

```bash
cd insighthr-web

# Cài đặt dependencies
npm install

# Build production bundle
npm run build
```

Điều này tạo một production build được tối ưu trong thư mục `dist/`.

**Build Output**:
```
dist/
├── index.html
├── assets/
│   ├── index-[hash].js
│   ├── index-[hash].css
│   └── [other assets]
└── [other files]
```

#### Tạo S3 Bucket

Tạo S3 bucket để lưu trữ static website:

```bash
# Tạo bucket
aws s3 mb s3://insighthr-web-app-sg --region ap-southeast-1

# Bật static website hosting
aws s3 website s3://insighthr-web-app-sg \
    --index-document index.html \
    --error-document index.html
```

#### Cấu hình Bucket Policy

Tạo bucket policy để cho phép CloudFront truy cập:

**bucket-policy.json**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::insighthr-web-app-sg/*"
    }
  ]
}
```

Áp dụng policy:
```bash
aws s3api put-bucket-policy \
    --bucket insighthr-web-app-sg \
    --policy file://bucket-policy.json
```

#### Upload Files lên S3

```bash
# Sync thư mục dist lên S3
aws s3 sync dist/ s3://insighthr-web-app-sg \
    --region ap-southeast-1 \
    --delete

# Xác minh upload
aws s3 ls s3://insighthr-web-app-sg --recursive
```

### Bước 2: Cấu hình CloudFront

#### Tạo CloudFront Distribution

**Sử dụng AWS Console**:

1. Điều hướng đến CloudFront
2. Click "Create Distribution"
3. Cấu hình origin:
   - Origin domain: `insighthr-web-app-sg.s3.ap-southeast-1.amazonaws.com`
   - Origin path: (để trống)
   - Name: `S3-insighthr-web-app`

4. Default cache behavior:
   - Viewer protocol policy: Redirect HTTP to HTTPS
   - Allowed HTTP methods: GET, HEAD, OPTIONS
   - Cache policy: CachingOptimized

5. Settings:
   - Price class: Use all edge locations
   - Alternate domain names (CNAMEs): `insight-hr.io.vn`, `www.insight-hr.io.vn`
   - Custom SSL certificate: Yêu cầu hoặc import certificate
   - Default root object: `index.html`

6. Click "Create Distribution"

#### Cấu hình Custom Error Pages

Thiết lập error pages cho SPA routing:

1. Đi đến CloudFront distribution
2. Điều hướng đến tab "Error Pages"
3. Tạo custom error response:
   - HTTP error code: 403
   - Customize error response: Yes
   - Response page path: `/index.html`
   - HTTP response code: 200

4. Lặp lại cho error code 404

### Bước 3: Triển khai Lambda

#### Đóng gói Lambda Functions

Cho mỗi Lambda function, tạo deployment package:

**Ví dụ: Auth Login Handler**

```bash
cd lambda/auth

# Tạo deployment package
mkdir -p package
pip install -r requirements.txt -t package/
cd package
zip -r ../auth-login-handler.zip .
cd ..
zip -g auth-login-handler.zip auth_login_handler.py

# Triển khai lên Lambda
aws lambda create-function \
    --function-name insighthr-auth-login-handler \
    --runtime python3.11 \
    --role arn:aws:iam::ACCOUNT_ID:role/insighthr-lambda-role \
    --handler auth_login_handler.lambda_handler \
    --zip-file fileb://auth-login-handler.zip \
    --timeout 30 \
    --memory-size 256 \
    --environment Variables="{
        USER_POOL_ID=ap-southeast-1_rzDtdAhvp,
        CLIENT_ID=6suhk5huhe40o6iuqgsnmuucj5,
        DYNAMODB_USERS_TABLE=insighthr-users-dev
    }" \
    --region ap-southeast-1
```

### Bước 4: Cấu hình API Gateway

#### Tạo REST API

```bash
# Tạo API
aws apigateway create-rest-api \
    --name "InsightHR API" \
    --description "InsightHR Backend API" \
    --region ap-southeast-1
```

#### Triển khai API

```bash
# Tạo deployment
aws apigateway create-deployment \
    --rest-api-id $API_ID \
    --stage-name dev \
    --description "Initial deployment"

# Lấy API endpoint
echo "API Endpoint: https://$API_ID.execute-api.ap-southeast-1.amazonaws.com/dev"
```

### Bước 5: Cấu hình Môi trường

#### Biến Môi trường Frontend

Tạo file `.env.production`:

```bash
VITE_API_BASE_URL=https://lqk4t6qzag.execute-api.ap-southeast-1.amazonaws.com/dev
VITE_GOOGLE_CLIENT_ID=your-google-client-id
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=ap-southeast-1_rzDtdAhvp
VITE_COGNITO_CLIENT_ID=6suhk5huhe40o6iuqgsnmuucj5
```

Rebuild và triển khai lại frontend:

```bash
npm run build
aws s3 sync dist/ s3://insighthr-web-app-sg --delete
```

### Bước 6: CloudFront Cache Invalidation

Sau khi triển khai thay đổi frontend, invalidate CloudFront cache:

```bash
# Lấy distribution ID
DIST_ID=$(aws cloudfront list-distributions \
    --query "DistributionList.Items[?Comment=='InsightHR CloudFront Distribution'].Id" \
    --output text)

# Tạo invalidation
aws cloudfront create-invalidation \
    --distribution-id $DIST_ID \
    --paths "/*"
```

### Bước 7: Cấu hình DNS (Tùy chọn)

Nếu sử dụng tên miền tùy chỉnh:

#### Yêu cầu SSL Certificate

```bash
# Yêu cầu certificate ở us-east-1 (bắt buộc cho CloudFront)
aws acm request-certificate \
    --domain-name insight-hr.io.vn \
    --subject-alternative-names www.insight-hr.io.vn \
    --validation-method DNS \
    --region us-east-1
```

### Checklist Triển khai

Trước khi đưa vào sử dụng, xác minh:

- [ ] Tất cả Lambda functions đã được triển khai và kiểm tra
- [ ] API Gateway endpoints đã được cấu hình
- [ ] Bảng DynamoDB đã được điền dữ liệu ban đầu
- [ ] Cognito User Pool đã được cấu hình
- [ ] Frontend đã được build và triển khai lên S3
- [ ] CloudFront distribution đang hoạt động
- [ ] SSL certificate đã được xác thực (nếu sử dụng tên miền tùy chỉnh)
- [ ] DNS records đã được cấu hình (nếu sử dụng tên miền tùy chỉnh)
- [ ] Biến môi trường đã được đặt đúng
- [ ] CORS đã được cấu hình trên API Gateway
- [ ] CloudWatch logs đã được bật
- [ ] IAM roles và permissions đã được xác minh

### Kiểm tra Triển khai

#### Kiểm tra Frontend

```bash
# Truy cập CloudFront URL
curl -I https://d2z6tht6rq32uy.cloudfront.net

# Hoặc tên miền tùy chỉnh
curl -I https://insight-hr.io.vn
```

#### Kiểm tra API Endpoints

```bash
# Kiểm tra health check
curl https://lqk4t6qzag.execute-api.ap-southeast-1.amazonaws.com/dev/health

# Kiểm tra authenticated endpoint (với token)
curl -H "Authorization: Bearer YOUR_JWT_TOKEN" \
    https://lqk4t6qzag.execute-api.ap-southeast-1.amazonaws.com/dev/employees
```

### Continuous Deployment

Để triển khai tự động, xem xét:

- **GitHub Actions**: Tự động build và deploy khi push
- **AWS CodePipeline**: Pipeline CI/CD đầy đủ
- **AWS Amplify**: Triển khai frontend đơn giản hóa

### Xử lý Sự cố

**Frontend không load**:
- Kiểm tra S3 bucket policy
- Xác minh trạng thái CloudFront distribution
- Kiểm tra browser console để tìm lỗi

**Lỗi API**:
- Xác minh Lambda function logs trong CloudWatch
- Kiểm tra cấu hình API Gateway
- Xác minh thiết lập Cognito authorizer

**Vấn đề CORS**:
- Cấu hình CORS trên API Gateway
- Kiểm tra allowed origins khớp với domain frontend

### Bước Tiếp theo

Với triển khai hoàn tất, tiến hành [Kiểm tra & Giám sát](../5.10-testing/) để thiết lập giám sát CloudWatch và synthetic canaries.
