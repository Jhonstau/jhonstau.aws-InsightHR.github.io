---
title: "Prerequisites"
date: "2025-12-09"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Prerequisites for InsightHR Workshop

Before starting this workshop, ensure you have the following tools and accounts set up.

### 1. AWS Account

You'll need an AWS account with appropriate permissions to create and manage resources.

**Required AWS Services Access:**
- IAM (Identity and Access Management)
- DynamoDB
- Lambda
- API Gateway
- S3
- CloudFront
- Cognito
- Bedrock
- CloudWatch
- Route53 (optional, for custom domain)

**Estimated Costs:** $2-5/month during development

{{% notice info %}}
If you're using AWS Free Tier, many services in this workshop are covered. However, some services like Bedrock may incur small charges.
{{% /notice %}}

### 2. AWS CLI

Install and configure the AWS Command Line Interface.

**Installation:**

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

**Configuration:**
```bash
aws configure
```

Enter your:
- AWS Access Key ID
- AWS Secret Access Key
- Default region (e.g., `ap-southeast-1`)
- Default output format (e.g., `json`)

**Verify Installation:**
```bash
aws --version
# Expected output: aws-cli/2.x.x Python/3.x.x ...
```

### 3. Node.js and npm

Required for frontend development.

**Minimum Version:** Node.js 18+

**Installation:**

Download from [nodejs.org](https://nodejs.org/) or use a version manager:

**Using nvm (recommended):**
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install Node.js
nvm install 18
nvm use 18
```

**Verify Installation:**
```bash
node --version
# Expected: v18.x.x or higher

npm --version
# Expected: 9.x.x or higher
```

### 4. Python

Required for Lambda function development.

**Minimum Version:** Python 3.11+

**Installation:**

Download from [python.org](https://www.python.org/downloads/) or use your system's package manager.

**Verify Installation:**
```bash
python --version
# or
python3 --version
# Expected: Python 3.11.x or higher
```

**Install pip (if not included):**
```bash
python -m ensurepip --upgrade
```

### 5. Text Editor / IDE

Choose your preferred development environment:

**Recommended: Visual Studio Code**
- Download from [code.visualstudio.com](https://code.visualstudio.com/)
- Install recommended extensions:
  - AWS Toolkit
  - Python
  - ESLint
  - Prettier
  - Tailwind CSS IntelliSense

**Alternatives:**
- PyCharm
- Sublime Text
- Atom
- WebStorm

### 6. Git (Optional but Recommended)

For version control and accessing code repositories.

**Installation:**

Download from [git-scm.com](https://git-scm.com/downloads)

**Verify Installation:**
```bash
git --version
# Expected: git version 2.x.x
```

### 7. Required Knowledge

**JavaScript/TypeScript:**
- Basic syntax and ES6+ features
- Async/await and Promises
- React fundamentals (components, hooks, state)

**Python:**
- Basic syntax and data structures
- Functions and error handling
- Working with JSON

**AWS Concepts:**
- Basic understanding of cloud computing
- Familiarity with AWS Console
- Understanding of serverless architecture (helpful but not required)

**Web Development:**
- HTML/CSS basics
- REST API concepts
- HTTP methods (GET, POST, PUT, DELETE)

### 8. AWS Account Setup

#### Create IAM User

For security best practices, create an IAM user instead of using root credentials:

1. Sign in to AWS Console
2. Navigate to IAM service
3. Click "Users" → "Add users"
4. Enter username (e.g., `insighthr-admin`)
5. Select "Programmatic access" and "AWS Management Console access"
6. Attach policies:
   - `AdministratorAccess` (for workshop purposes)
   - Or create a custom policy with required permissions

7. Download credentials (Access Key ID and Secret Access Key)
8. Configure AWS CLI with these credentials

#### Enable Required Services

Ensure the following services are available in your region:

- ✅ AWS Lambda
- ✅ Amazon DynamoDB
- ✅ Amazon API Gateway
- ✅ Amazon S3
- ✅ Amazon CloudFront
- ✅ Amazon Cognito
- ✅ Amazon Bedrock (check regional availability)
- ✅ Amazon CloudWatch

{{% notice tip %}}
**Recommended Region:** `ap-southeast-1` (Singapore) - All services are available and latency is good for Southeast Asia.
{{% /notice %}}

### 9. Bedrock Model Access

AWS Bedrock requires explicit model access request.

**Steps to Enable:**

1. Go to AWS Console → Amazon Bedrock
2. Navigate to "Model access" in the left sidebar
3. Click "Manage model access"
4. Find "Claude 3 Haiku" by Anthropic
5. Check the box and click "Request model access"
6. Wait for approval (usually instant for Haiku)

{{% notice warning %}}
Without Bedrock access, the AI chatbot feature won't work. However, you can still complete the rest of the workshop.
{{% /notice %}}

### 10. Optional: Domain Name

If you want to use a custom domain (like `insight-hr.io.vn`):

- Purchase a domain from a registrar (e.g., Route53, GoDaddy, Namecheap)
- Have access to DNS management
- Budget for SSL certificate (free with AWS Certificate Manager)

### Pre-Workshop Checklist

Before proceeding to the next section, verify you have:

- [ ] AWS account with admin access
- [ ] AWS CLI installed and configured
- [ ] Node.js 18+ and npm installed
- [ ] Python 3.11+ installed
- [ ] Text editor/IDE set up
- [ ] Git installed (optional)
- [ ] Basic knowledge of JavaScript/TypeScript and Python
- [ ] Understanding of REST APIs
- [ ] AWS Bedrock model access requested
- [ ] Familiarity with AWS Console

### Troubleshooting

**AWS CLI Configuration Issues:**
```bash
# Check current configuration
aws configure list

# Test AWS access
aws sts get-caller-identity
```

**Node.js Version Issues:**
```bash
# Check installed versions
nvm list

# Switch to correct version
nvm use 18
```

**Python Version Issues:**
```bash
# Check Python path
which python3

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### Next Steps

Once you've completed all prerequisites, proceed to [Project Architecture](../5.3-architecture/) to understand the system design.

### Additional Resources

- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
- [Node.js Documentation](https://nodejs.org/docs/)
- [Python Documentation](https://docs.python.org/3/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
