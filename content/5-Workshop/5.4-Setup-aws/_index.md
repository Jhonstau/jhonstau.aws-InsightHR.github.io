---
title: "Setup AWS Environment"
date: "2025-12-09"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Setting Up AWS Environment

This section guides you through configuring your AWS environment for the InsightHR platform, including IAM roles, policies, and initial service configuration.

### Overview

We'll set up:

1. IAM roles for Lambda functions
2. IAM policies for service access
3. AWS region configuration
4. Service quotas verification
5. Bedrock model access

### Step 1: Configure AWS Region

Set your default region to Singapore (ap-southeast-1):

```bash
# Configure AWS CLI
aws configure set region ap-southeast-1

# Verify configuration
aws configure get region
```

**Why Singapore?**
- All required services available
- Good latency for Southeast Asia
- Bedrock Claude 3 Haiku available

### Step 2: Create IAM Role for Lambda

Lambda functions need an execution role to access AWS services.

#### Create Trust Policy

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

#### Create the Role

```bash
# Create IAM role
aws iam create-role \
    --role-name insighthr-lambda-role \
    --assume-role-policy-document file://lambda-trust-policy.json \
    --description "Execution role for InsightHR Lambda functions"
```

### Step 3: Create IAM Policies

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

Create the policy:
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

Create the policy:
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

Create the policy:
```bash
aws iam create-policy \
    --policy-name insighthr-bedrock-policy \
    --policy-document file://bedrock-policy.json
```

### Step 4: Attach Policies to Role

```bash
# Get AWS account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Attach AWS managed policy for Lambda basic execution
aws iam attach-role-policy \
    --role-name insighthr-lambda-role \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Attach custom policies
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

### Step 5: Create IAM Role for API Gateway

API Gateway needs a role to write logs to CloudWatch.

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

Create the role:
```bash
aws iam create-role \
    --role-name insighthr-apigateway-role \
    --assume-role-policy-document file://api-gateway-trust-policy.json

# Attach CloudWatch logs policy
aws iam attach-role-policy \
    --role-name insighthr-apigateway-role \
    --policy-arn arn:aws:iam::aws:policy/service-role/AmazonAPIGatewayPushToCloudWatchLogs
```

### Step 6: Enable Bedrock Model Access

Request access to Claude 3 Haiku model:

**Using AWS Console**:

1. Navigate to Amazon Bedrock
2. Click "Model access" in left sidebar
3. Click "Manage model access"
4. Find "Claude 3 Haiku" by Anthropic
5. Check the box
6. Click "Request model access"
7. Wait for approval (usually instant)

**Verify Access**:
```bash
aws bedrock list-foundation-models \
    --region ap-southeast-1 \
    --query 'modelSummaries[?contains(modelId, `claude-3-haiku`)].modelId'
```

### Step 7: Verify Service Quotas

Check service limits for your account:

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

### Step 8: Create S3 Bucket for Deployment Artifacts

```bash
# Create bucket for Lambda deployment packages
aws s3 mb s3://insighthr-deployment-artifacts-${ACCOUNT_ID} \
    --region ap-southeast-1

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket insighthr-deployment-artifacts-${ACCOUNT_ID} \
    --versioning-configuration Status=Enabled
```

### Step 9: Set Up CloudWatch Log Groups

Pre-create log groups for Lambda functions:

```bash
# Create log groups
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
    
    # Set retention to 7 days
    aws logs put-retention-policy \
        --log-group-name $log_group \
        --retention-in-days 7 \
        --region ap-southeast-1
done
```

### Step 10: Configure Environment Variables

Create a configuration file for environment variables:

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

# Cognito (will be set after Cognito setup)
export USER_POOL_ID=
export CLIENT_ID=

# Bedrock
export BEDROCK_MODEL_ID=anthropic.claude-3-haiku-20240307-v1:0
export BEDROCK_REGION=ap-southeast-1

# Lambda Role ARN
export LAMBDA_ROLE_ARN=arn:aws:iam::${AWS_ACCOUNT_ID}:role/insighthr-lambda-role
```

Load the configuration:
```bash
source config.env
```

### Verification Checklist

Verify your AWS environment is ready:

```bash
# Check IAM role exists
aws iam get-role --role-name insighthr-lambda-role

# Check policies are attached
aws iam list-attached-role-policies --role-name insighthr-lambda-role

# Check Bedrock access
aws bedrock list-foundation-models \
    --region ap-southeast-1 \
    --query 'modelSummaries[?contains(modelId, `claude`)].modelId'

# Check S3 bucket exists
aws s3 ls s3://insighthr-deployment-artifacts-${ACCOUNT_ID}

# Check CloudWatch log groups
aws logs describe-log-groups \
    --log-group-name-prefix "/aws/lambda/insighthr" \
    --region ap-southeast-1
```

### IAM Setup Summary

| Resource | Purpose | Policies Attached |
|----------|---------|-------------------|
| insighthr-lambda-role | Lambda execution | AWSLambdaBasicExecutionRole, DynamoDB, Cognito, Bedrock |
| insighthr-apigateway-role | API Gateway logging | AmazonAPIGatewayPushToCloudWatchLogs |
| insighthr-dynamodb-policy | DynamoDB access | Custom policy for table operations |
| insighthr-cognito-policy | Cognito access | Custom policy for user management |
| insighthr-bedrock-policy | Bedrock access | Custom policy for model invocation |

### Security Best Practices

✅ **Least Privilege**: Policies grant only necessary permissions
✅ **Resource Restrictions**: Policies limited to insighthr-* resources
✅ **Separate Roles**: Different roles for different services
✅ **CloudWatch Logging**: All Lambda functions log to CloudWatch
✅ **Versioning**: S3 bucket versioning enabled for artifacts

### Troubleshooting

**IAM Role Creation Fails**:
- Check IAM permissions
- Verify trust policy syntax
- Ensure role name is unique

**Policy Attachment Fails**:
- Verify policy ARN is correct
- Check role exists
- Ensure you have iam:AttachRolePolicy permission

**Bedrock Access Denied**:
- Request model access in Bedrock console
- Wait for approval (usually instant for Haiku)
- Verify region supports Bedrock

**Service Quota Issues**:
- Request quota increase in Service Quotas console
- Use different region if needed
- Contact AWS support for urgent increases

### Next Steps

With the AWS environment configured, proceed to [Database Setup](../5.5-database-setup/) to create DynamoDB tables.
