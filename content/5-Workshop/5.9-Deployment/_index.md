---
title: "Deployment"
date: "2025-12-09"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Deploying InsightHR to Production

This section covers deploying the complete InsightHR application to AWS, including frontend deployment to S3/CloudFront and backend Lambda functions.

### Deployment Overview

The deployment process consists of:

1. **Frontend Deployment**: Build and deploy React app to S3
2. **CloudFront Configuration**: Set up CDN for global distribution
3. **Lambda Deployment**: Deploy all backend functions
4. **API Gateway Configuration**: Set up REST API endpoints
5. **Environment Configuration**: Configure environment variables
6. **DNS Setup** (Optional): Configure custom domain

### Prerequisites

Before deploying, ensure you have:

- ✅ All DynamoDB tables created
- ✅ Cognito User Pool configured
- ✅ IAM roles and policies set up
- ✅ AWS CLI configured
- ✅ Node.js and npm installed
- ✅ Python 3.11+ installed

### Step 1: Frontend Deployment

#### Build the React Application

Navigate to the frontend directory and build the production bundle:

```bash
cd insighthr-web

# Install dependencies
npm install

# Build production bundle
npm run build
```

This creates an optimized production build in the `dist/` directory.

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

#### Create S3 Bucket

Create an S3 bucket for hosting the static website:

```bash
# Create bucket
aws s3 mb s3://insighthr-web-app-sg --region ap-southeast-1

# Enable static website hosting
aws s3 website s3://insighthr-web-app-sg \
    --index-document index.html \
    --error-document index.html
```

#### Configure Bucket Policy

Create a bucket policy to allow CloudFront access:

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

Apply the policy:
```bash
aws s3api put-bucket-policy \
    --bucket insighthr-web-app-sg \
    --policy file://bucket-policy.json
```

#### Upload Files to S3

```bash
# Sync dist folder to S3
aws s3 sync dist/ s3://insighthr-web-app-sg \
    --region ap-southeast-1 \
    --delete

# Verify upload
aws s3 ls s3://insighthr-web-app-sg --recursive
```

### Step 2: CloudFront Configuration

#### Create CloudFront Distribution

**Using AWS Console**:

1. Navigate to CloudFront
2. Click "Create Distribution"
3. Configure origin:
   - Origin domain: `insighthr-web-app-sg.s3.ap-southeast-1.amazonaws.com`
   - Origin path: (leave empty)
   - Name: `S3-insighthr-web-app`

4. Default cache behavior:
   - Viewer protocol policy: Redirect HTTP to HTTPS
   - Allowed HTTP methods: GET, HEAD, OPTIONS
   - Cache policy: CachingOptimized

5. Settings:
   - Price class: Use all edge locations
   - Alternate domain names (CNAMEs): `insight-hr.io.vn`, `www.insight-hr.io.vn`
   - Custom SSL certificate: Request or import certificate
   - Default root object: `index.html`

6. Click "Create Distribution"

**Using AWS CLI**:

```bash
# Create distribution configuration
cat > cloudfront-config.json << EOF
{
  "CallerReference": "insighthr-$(date +%s)",
  "Comment": "InsightHR CloudFront Distribution",
  "DefaultRootObject": "index.html",
  "Origins": {
    "Quantity": 1,
    "Items": [
      {
        "Id": "S3-insighthr-web-app",
        "DomainName": "insighthr-web-app-sg.s3.ap-southeast-1.amazonaws.com",
        "S3OriginConfig": {
          "OriginAccessIdentity": ""
        }
      }
    ]
  },
  "DefaultCacheBehavior": {
    "TargetOriginId": "S3-insighthr-web-app",
    "ViewerProtocolPolicy": "redirect-to-https",
    "AllowedMethods": {
      "Quantity": 3,
      "Items": ["GET", "HEAD", "OPTIONS"]
    },
    "ForwardedValues": {
      "QueryString": false,
      "Cookies": {"Forward": "none"}
    },
    "MinTTL": 0,
    "DefaultTTL": 86400,
    "MaxTTL": 31536000
  },
  "Enabled": true
}
EOF

# Create distribution
aws cloudfront create-distribution \
    --distribution-config file://cloudfront-config.json
```

#### Configure Custom Error Pages

Set up error pages for SPA routing:

1. Go to CloudFront distribution
2. Navigate to "Error Pages" tab
3. Create custom error response:
   - HTTP error code: 403
   - Customize error response: Yes
   - Response page path: `/index.html`
   - HTTP response code: 200

4. Repeat for error code 404

### Step 3: Lambda Deployment

#### Package Lambda Functions

For each Lambda function, create a deployment package:

**Example: Auth Login Handler**

```bash
cd lambda/auth

# Create deployment package
mkdir -p package
pip install -r requirements.txt -t package/
cd package
zip -r ../auth-login-handler.zip .
cd ..
zip -g auth-login-handler.zip auth_login_handler.py

# Deploy to Lambda
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

#### Deploy All Lambda Functions

Create a deployment script:

**deploy-all-lambdas.sh**:
```bash
#!/bin/bash

FUNCTIONS=(
    "auth-login-handler"
    "auth-register-handler"
    "auth-google-handler"
    "employees-handler"
    "employees-bulk-handler"
    "performance-scores-handler"
    "chatbot-handler"
    "attendance-handler"
)

for func in "${FUNCTIONS[@]}"; do
    echo "Deploying $func..."
    # Package and deploy logic here
done
```

### Step 4: API Gateway Configuration

#### Create REST API

```bash
# Create API
aws apigateway create-rest-api \
    --name "InsightHR API" \
    --description "InsightHR Backend API" \
    --region ap-southeast-1
```

#### Create Resources and Methods

**Example: Create /employees endpoint**

```bash
# Get API ID
API_ID=$(aws apigateway get-rest-apis \
    --query "items[?name=='InsightHR API'].id" \
    --output text)

# Get root resource ID
ROOT_ID=$(aws apigateway get-resources \
    --rest-api-id $API_ID \
    --query "items[?path=='/'].id" \
    --output text)

# Create /employees resource
EMPLOYEES_ID=$(aws apigateway create-resource \
    --rest-api-id $API_ID \
    --parent-id $ROOT_ID \
    --path-part employees \
    --query 'id' \
    --output text)

# Create GET method
aws apigateway put-method \
    --rest-api-id $API_ID \
    --resource-id $EMPLOYEES_ID \
    --http-method GET \
    --authorization-type COGNITO_USER_POOLS \
    --authorizer-id $AUTHORIZER_ID

# Integrate with Lambda
aws apigateway put-integration \
    --rest-api-id $API_ID \
    --resource-id $EMPLOYEES_ID \
    --http-method GET \
    --type AWS_PROXY \
    --integration-http-method POST \
    --uri arn:aws:apigateway:ap-southeast-1:lambda:path/2015-03-31/functions/arn:aws:lambda:ap-southeast-1:ACCOUNT_ID:function:insighthr-employees-handler/invocations
```

#### Deploy API

```bash
# Create deployment
aws apigateway create-deployment \
    --rest-api-id $API_ID \
    --stage-name dev \
    --description "Initial deployment"

# Get API endpoint
echo "API Endpoint: https://$API_ID.execute-api.ap-southeast-1.amazonaws.com/dev"
```

### Step 5: Environment Configuration

#### Frontend Environment Variables

Create `.env.production` file:

```bash
VITE_API_BASE_URL=https://lqk4t6qzag.execute-api.ap-southeast-1.amazonaws.com/dev
VITE_GOOGLE_CLIENT_ID=your-google-client-id
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=ap-southeast-1_rzDtdAhvp
VITE_COGNITO_CLIENT_ID=6suhk5huhe40o6iuqgsnmuucj5
```

Rebuild and redeploy frontend:

```bash
npm run build
aws s3 sync dist/ s3://insighthr-web-app-sg --delete
```

#### Lambda Environment Variables

Update Lambda functions with environment variables:

```bash
aws lambda update-function-configuration \
    --function-name insighthr-employees-handler \
    --environment Variables="{
        AWS_REGION=ap-southeast-1,
        EMPLOYEES_TABLE=insighthr-employees-dev,
        USERS_TABLE=insighthr-users-dev
    }"
```

### Step 6: CloudFront Cache Invalidation

After deploying frontend changes, invalidate CloudFront cache:

```bash
# Get distribution ID
DIST_ID=$(aws cloudfront list-distributions \
    --query "DistributionList.Items[?Comment=='InsightHR CloudFront Distribution'].Id" \
    --output text)

# Create invalidation
aws cloudfront create-invalidation \
    --distribution-id $DIST_ID \
    --paths "/*"
```

### Step 7: DNS Configuration (Optional)

If using a custom domain:

#### Request SSL Certificate

```bash
# Request certificate in us-east-1 (required for CloudFront)
aws acm request-certificate \
    --domain-name insight-hr.io.vn \
    --subject-alternative-names www.insight-hr.io.vn \
    --validation-method DNS \
    --region us-east-1
```

#### Configure Route53

```bash
# Create hosted zone
aws route53 create-hosted-zone \
    --name insight-hr.io.vn \
    --caller-reference $(date +%s)

# Create A record for CloudFront
cat > change-batch.json << EOF
{
  "Changes": [{
    "Action": "CREATE",
    "ResourceRecordSet": {
      "Name": "insight-hr.io.vn",
      "Type": "A",
      "AliasTarget": {
        "HostedZoneId": "Z2FDTNDATAQYW2",
        "DNSName": "d2z6tht6rq32uy.cloudfront.net",
        "EvaluateTargetHealth": false
      }
    }
  }]
}
EOF

aws route53 change-resource-record-sets \
    --hosted-zone-id ZONE_ID \
    --change-batch file://change-batch.json
```

### Deployment Checklist

Before going live, verify:

- [ ] All Lambda functions deployed and tested
- [ ] API Gateway endpoints configured
- [ ] DynamoDB tables populated with initial data
- [ ] Cognito User Pool configured
- [ ] Frontend built and deployed to S3
- [ ] CloudFront distribution active
- [ ] SSL certificate validated (if using custom domain)
- [ ] DNS records configured (if using custom domain)
- [ ] Environment variables set correctly
- [ ] CORS configured on API Gateway
- [ ] CloudWatch logs enabled
- [ ] IAM roles and permissions verified

### Testing Deployment

#### Test Frontend

```bash
# Access CloudFront URL
curl -I https://d2z6tht6rq32uy.cloudfront.net

# Or custom domain
curl -I https://insight-hr.io.vn
```

#### Test API Endpoints

```bash
# Test health check
curl https://lqk4t6qzag.execute-api.ap-southeast-1.amazonaws.com/dev/health

# Test authenticated endpoint (with token)
curl -H "Authorization: Bearer YOUR_JWT_TOKEN" \
    https://lqk4t6qzag.execute-api.ap-southeast-1.amazonaws.com/dev/employees
```

### Continuous Deployment

For automated deployments, consider:

- **GitHub Actions**: Automate build and deploy on push
- **AWS CodePipeline**: Full CI/CD pipeline
- **AWS Amplify**: Simplified frontend deployment

### Troubleshooting

**Frontend not loading**:
- Check S3 bucket policy
- Verify CloudFront distribution status
- Check browser console for errors

**API errors**:
- Verify Lambda function logs in CloudWatch
- Check API Gateway configuration
- Verify Cognito authorizer setup

**CORS issues**:
- Configure CORS on API Gateway
- Check allowed origins match frontend domain

### Next Steps

With deployment complete, proceed to [Testing & Monitoring](../5.10-testing/) to set up CloudWatch monitoring and synthetic canaries.
