---
title: "Yêu cầu tiên quyết"
date: "2025-12-09"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Yêu cầu tiên quyết cho Workshop InsightHR

Trước khi bắt đầu workshop này, hãy đảm bảo bạn đã thiết lập các công cụ và tài khoản sau.

### 1. Tài khoản AWS

Bạn cần một tài khoản AWS với quyền phù hợp để tạo và quản lý tài nguyên.

**Quyền truy cập Dịch vụ AWS Yêu cầu:**
- IAM (Identity and Access Management)
- DynamoDB
- Lambda
- API Gateway
- S3
- CloudFront
- Cognito
- Bedrock
- CloudWatch
- Route53 (tùy chọn, cho custom domain)

**Ước tính Chi phí:** $2-5/tháng trong quá trình phát triển

{{% notice info %}}
Nếu bạn đang sử dụng AWS Free Tier, nhiều dịch vụ trong workshop này được bao phủ. Tuy nhiên, một số dịch vụ như Bedrock có thể phát sinh phí nhỏ.
{{% /notice %}}

### 2. AWS CLI

Cài đặt và cấu hình AWS Command Line Interface.

**Cài đặt:**

**Windows:**
```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

**macOS:**
```bash
curl "https://awscli.amazonaws.com/AWSCLI2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

**Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Cấu hình:**
```bash
aws configure
```

Nhập thông tin:
- AWS Access Key ID
- AWS Secret Access Key
- Default region (ví dụ: `ap-southeast-1`)
- Default output format (ví dụ: `json`)

**Xác minh Cài đặt:**
```bash
aws --version
# Kết quả mong đợi: aws-cli/2.x.x Python/3.x.x ...
```

### 3. Node.js và npm

Yêu cầu cho phát triển frontend.

**Phiên bản Tối thiểu:** Node.js 18+

**Cài đặt:**

Tải từ [nodejs.org](https://nodejs.org/) hoặc sử dụng version manager:

**Sử dụng nvm (khuyến nghị):**
```bash
# Cài đặt nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Cài đặt Node.js
nvm install 18
nvm use 18
```

**Xác minh Cài đặt:**
```bash
node --version
# Mong đợi: v18.x.x hoặc cao hơn

npm --version
# Mong đợi: 9.x.x hoặc cao hơn
```

### 4. Python

Yêu cầu cho phát triển Lambda function.

**Phiên bản Tối thiểu:** Python 3.11+

**Cài đặt:**

Tải từ [python.org](https://www.python.org/downloads/) hoặc sử dụng package manager của hệ thống.

**Xác minh Cài đặt:**
```bash
python --version
# hoặc
python3 --version
# Mong đợi: Python 3.11.x hoặc cao hơn
```

**Cài đặt pip (nếu chưa có):**
```bash
python -m ensurepip --upgrade
```

### 5. Text Editor / IDE

Chọn môi trường phát triển ưa thích của bạn:

**Khuyến nghị: Visual Studio Code**
- Tải từ [code.visualstudio.com](https://code.visualstudio.com/)
- Cài đặt các extension khuyến nghị:
  - AWS Toolkit
  - Python
  - ESLint
  - Prettier
  - Tailwind CSS IntelliSense

**Lựa chọn khác:**
- PyCharm
- Sublime Text
- Atom
- WebStorm

### 6. Git (Tùy chọn nhưng Khuyến nghị)

Cho version control và truy cập code repositories.

**Cài đặt:**

Tải từ [git-scm.com](https://git-scm.com/downloads)

**Xác minh Cài đặt:**
```bash
git --version
# Mong đợi: git version 2.x.x
```

### 7. Kiến thức Yêu cầu

**JavaScript/TypeScript:**
- Cú pháp cơ bản và tính năng ES6+
- Async/await và Promises
- React cơ bản (components, hooks, state)

**Python:**
- Cú pháp cơ bản và cấu trúc dữ liệu
- Functions và xử lý lỗi
- Làm việc với JSON

**Khái niệm AWS:**
- Hiểu biết cơ bản về cloud computing
- Quen thuộc với AWS Console
- Hiểu về kiến trúc serverless (hữu ích nhưng không bắt buộc)

**Web Development:**
- HTML/CSS cơ bản
- Khái niệm REST API
- HTTP methods (GET, POST, PUT, DELETE)

### 8. Thiết lập Tài khoản AWS

#### Tạo IAM User

Để tuân thủ phương pháp bảo mật tốt nhất, tạo IAM user thay vì sử dụng root credentials:

1. Đăng nhập vào AWS Console
2. Điều hướng đến dịch vụ IAM
3. Click "Users" → "Add users"
4. Nhập username (ví dụ: `insighthr-admin`)
5. Chọn "Programmatic access" và "AWS Management Console access"
6. Gắn policies:
   - `AdministratorAccess` (cho mục đích workshop)
   - Hoặc tạo custom policy với quyền yêu cầu

7. Tải credentials (Access Key ID và Secret Access Key)
8. Cấu hình AWS CLI với credentials này

#### Kích hoạt Dịch vụ Yêu cầu

Đảm bảo các dịch vụ sau có sẵn trong region của bạn:

- ✅ AWS Lambda
- ✅ Amazon DynamoDB
- ✅ Amazon API Gateway
- ✅ Amazon S3
- ✅ Amazon CloudFront
- ✅ Amazon Cognito
- ✅ Amazon Bedrock (kiểm tra tính khả dụng theo region)
- ✅ Amazon CloudWatch

{{% notice tip %}}
**Region Khuyến nghị:** `ap-southeast-1` (Singapore) - Tất cả dịch vụ đều có sẵn và độ trễ tốt cho Đông Nam Á.
{{% /notice %}}

### 9. Quyền truy cập Bedrock Model

AWS Bedrock yêu cầu yêu cầu quyền truy cập model rõ ràng.

**Các bước Kích hoạt:**

1. Vào AWS Console → Amazon Bedrock
2. Điều hướng đến "Model access" ở sidebar bên trái
3. Click "Manage model access"
4. Tìm "Claude 3 Haiku" của Anthropic
5. Đánh dấu checkbox và click "Request model access"
6. Chờ phê duyệt (thường là ngay lập tức cho Haiku)

{{% notice warning %}}
Không có quyền truy cập Bedrock, tính năng AI chatbot sẽ không hoạt động. Tuy nhiên, bạn vẫn có thể hoàn thành phần còn lại của workshop.
{{% /notice %}}

### 10. Tùy chọn: Tên miền

Nếu bạn muốn sử dụng custom domain (như `insight-hr.io.vn`):

- Mua domain từ registrar (ví dụ: Route53, GoDaddy, Namecheap)
- Có quyền truy cập quản lý DNS
- Ngân sách cho SSL certificate (miễn phí với AWS Certificate Manager)

### Checklist Trước Workshop

Trước khi tiếp tục phần tiếp theo, xác minh bạn có:

- [ ] Tài khoản AWS với quyền admin
- [ ] AWS CLI đã cài đặt và cấu hình
- [ ] Node.js 18+ và npm đã cài đặt
- [ ] Python 3.11+ đã cài đặt
- [ ] Text editor/IDE đã thiết lập
- [ ] Git đã cài đặt (tùy chọn)
- [ ] Kiến thức cơ bản về JavaScript/TypeScript và Python
- [ ] Hiểu biết về REST APIs
- [ ] Đã yêu cầu quyền truy cập AWS Bedrock model
- [ ] Quen thuộc với AWS Console

### Khắc phục Sự cố

**Vấn đề Cấu hình AWS CLI:**
```bash
# Kiểm tra cấu hình hiện tại
aws configure list

# Test quyền truy cập AWS
aws sts get-caller-identity
```

**Vấn đề Phiên bản Node.js:**
```bash
# Kiểm tra các phiên bản đã cài đặt
nvm list

# Chuyển sang phiên bản đúng
nvm use 18
```

**Vấn đề Phiên bản Python:**
```bash
# Kiểm tra đường dẫn Python
which python3

# Tạo virtual environment
python3 -m venv venv
source venv/bin/activate  # Trên Windows: venv\Scripts\activate
```

### Bước tiếp theo

Sau khi hoàn thành tất cả yêu cầu tiên quyết, tiếp tục đến [Kiến trúc Dự án](../5.3-architecture/) để hiểu thiết kế hệ thống.

### Tài nguyên Bổ sung

- [Tài liệu AWS CLI](https://docs.aws.amazon.com/cli/)
- [Tài liệu Node.js](https://nodejs.org/docs/)
- [Tài liệu Python](https://docs.python.org/3/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Tài liệu AWS Bedrock](https://docs.aws.amazon.com/bedrock/)
