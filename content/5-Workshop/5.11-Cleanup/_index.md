---
title: "Cleanup"
date: "2025-12-09"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Cleaning Up Resources

To avoid ongoing charges, it's important to delete all AWS resources created during this workshop. This section provides step-by-step instructions for cleaning up.

{{% notice warning %}}
**Important**: Deleting resources is irreversible. Make sure you've backed up any data you want to keep before proceeding.
{{% /notice %}}

### Cleanup Overview

We'll delete resources in the following order:

1. CloudFront Distribution
2. S3 Buckets
3. API Gateway
4. Lambda Functions
5. DynamoDB Tables
6. Cognito User Pool
7. CloudWatch Logs
8. IAM Roles and Policies
9. Route53 (if configured)
10. ACM Certificates (if created)

### Step 1: Delete CloudFront Distribution

CloudFront distributions must be disabled before deletion.

**Using AWS Console**:

1. Navigate to CloudFront
2. Select the InsightHR distribution
3. Click "Disable"
4. Wait for status to change to "Disabled" (may take 15-20 minutes)
5. Select the distribution again
6. Click "Delete"

**Using AWS CLI**:

```bash
# Get distribution ID
DIST_ID=$(aws cloudfront list-distributions \
    --query "DistributionList.Items[?Comment=='InsightHR CloudFront Distribution'].Id" \
    --output text)

# Get ETag
ETAG=$(aws cloudfront get-distribution --id $DIST_ID \
    --query 'ETag' --output text)

# Disable distribution
aws cloudfront get-distribution-config --id $DIST_ID > dist-config.json
# Edit dist-config.json: Set "Enabled": false
aws cloudfront update-distribution \
    --id $DIST_ID \
    --if-match $ETAG \
    --distribution-config file://dist-config.json

# Wait for distribution to be disabled
aws cloudfront wait distribution-deployed --id $DIST_ID

# Delete distribution
ETAG=$(aws cloudfront get-distribution --id $DIST_ID \
    --query 'ETag' --output text)
aws cloudfront delete-distribution --id $DIST_ID --if-match $ETAG
```

### Step 2: Delete S3 Buckets

**Empty and delete the web app bucket**:

```bash
# Empty bucket
aws s3 rm s3://insighthr-web-app-sg --recursive

# Delete bucket
aws s3 rb s3://insighthr-web-app-sg --force
```

**Delete canary artifacts bucket** (if created):

```bash
aws s3 rm s3://insighthr-canary-artifacts --recursive
aws s3 rb s3://insighthr-canary-artifacts --force
```

### Step 3: Delete API Gateway

**Using AWS Console**:

1. Navigate to API Gateway
2. Select "InsightHR API"
3. Click "Actions" → "Delete API"
4. Confirm deletion

**Using AWS CLI**:

```bash
# Get API ID
API_ID=$(aws apigateway get-rest-apis \
    --query "items[?name=='InsightHR API'].id" \
    --output text \
    --region ap-southeast-1)

# Delete API
aws apigateway delete-rest-api \
    --rest-api-id $API_ID \
    --region ap-southeast-1
```

### Step 4: Delete Lambda Functions

**Delete all Lambda functions**:

```bash
# List of functions to delete
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

# Delete each function
for func in "${FUNCTIONS[@]}"; do
    echo "Deleting $func..."
    aws lambda delete-function \
        --function-name $func \
        --region ap-southeast-1
done
```

### Step 5: Delete DynamoDB Tables

**Using AWS Console**:

1. Navigate to DynamoDB
2. Select each table
3. Click "Delete"
4. Confirm by typing "delete"

**Using AWS CLI**:

```bash
# List of tables to delete
TABLES=(
    "insighthr-users-dev"
    "insighthr-employees-dev"
    "insighthr-performance-scores-dev"
    "insighthr-attendance-history-dev"
    "insighthr-password-reset-requests-dev"
    "insighthr-kpis-dev"
    "insighthr-formulas-dev"
    "insighthr-data-tables-dev"
    "insighthr-notification-rules-dev"
)

# Delete each table
for table in "${TABLES[@]}"; do
    echo "Deleting $table..."
    aws dynamodb delete-table \
        --table-name $table \
        --region ap-southeast-1
done
```

### Step 6: Delete Cognito User Pool

**Using AWS Console**:

1. Navigate to Cognito
2. Select "User Pools"
3. Select the InsightHR user pool
4. Click "Delete user pool"
5. Type the pool name to confirm

**Using AWS CLI**:

```bash
# Get user pool ID
USER_POOL_ID=$(aws cognito-idp list-user-pools \
    --max-results 10 \
    --query "UserPools[?Name=='insighthr-user-pool'].Id" \
    --output text \
    --region ap-southeast-1)

# Delete user pool
aws cognito-idp delete-user-pool \
    --user-pool-id $USER_POOL_ID \
    --region ap-southeast-1
```

### Step 7: Delete CloudWatch Logs

**Delete log groups**:

```bash
# List log groups
aws logs describe-log-groups \
    --log-group-name-prefix "/aws/lambda/insighthr" \
    --region ap-southeast-1

# Delete each log group
LOG_GROUPS=$(aws logs describe-log-groups \
    --log-group-name-prefix "/aws/lambda/insighthr" \
    --query 'logGroups[*].logGroupName' \
    --output text \
    --region ap-southeast-1)

for log_group in $LOG_GROUPS; do
    echo "Deleting $log_group..."
    aws logs delete-log-group \
        --log-group-name $log_group \
        --region ap-southeast-1
done
```

**Delete CloudWatch Synthetics Canaries** (if created):

```bash
# List canaries
aws synthetics describe-canaries \
    --region ap-southeast-1

# Delete each canary
CANARIES=$(aws synthetics describe-canaries \
    --query 'Canaries[?starts_with(Name, `insighthr`)].Name' \
    --output text \
    --region ap-southeast-1)

for canary in $CANARIES; do
    echo "Deleting canary $canary..."
    aws synthetics delete-canary \
        --name $canary \
        --region ap-southeast-1
done
```

### Step 8: Delete IAM Roles and Policies

**Using AWS Console**:

1. Navigate to IAM
2. Go to "Roles"
3. Search for "insighthr"
4. Select each role
5. Detach policies
6. Delete role

**Using AWS CLI**:

```bash
# List roles
aws iam list-roles \
    --query 'Roles[?starts_with(RoleName, `insighthr`)].RoleName' \
    --output text

# For each role, detach policies and delete
ROLES=$(aws iam list-roles \
    --query 'Roles[?starts_with(RoleName, `insighthr`)].RoleName' \
    --output text)

for role in $ROLES; do
    echo "Processing role $role..."
    
    # Detach managed policies
    POLICIES=$(aws iam list-attached-role-policies \
        --role-name $role \
        --query 'AttachedPolicies[*].PolicyArn' \
        --output text)
    
    for policy in $POLICIES; do
        aws iam detach-role-policy \
            --role-name $role \
            --policy-arn $policy
    done
    
    # Delete inline policies
    INLINE_POLICIES=$(aws iam list-role-policies \
        --role-name $role \
        --query 'PolicyNames[*]' \
        --output text)
    
    for policy in $INLINE_POLICIES; do
        aws iam delete-role-policy \
            --role-name $role \
            --policy-name $policy
    done
    
    # Delete role
    aws iam delete-role --role-name $role
done
```

**Delete custom policies**:

```bash
# List custom policies
aws iam list-policies \
    --scope Local \
    --query 'Policies[?starts_with(PolicyName, `insighthr`)].Arn' \
    --output text

# Delete each policy
POLICIES=$(aws iam list-policies \
    --scope Local \
    --query 'Policies[?starts_with(PolicyName, `insighthr`)].Arn' \
    --output text)

for policy in $POLICIES; do
    echo "Deleting policy $policy..."
    
    # Delete all policy versions except default
    VERSIONS=$(aws iam list-policy-versions \
        --policy-arn $policy \
        --query 'Versions[?!IsDefaultVersion].VersionId' \
        --output text)
    
    for version in $VERSIONS; do
        aws iam delete-policy-version \
            --policy-arn $policy \
            --version-id $version
    done
    
    # Delete policy
    aws iam delete-policy --policy-arn $policy
done
```

### Step 9: Delete Route53 Resources (If Configured)

**Delete DNS records**:

```bash
# Get hosted zone ID
ZONE_ID=$(aws route53 list-hosted-zones \
    --query "HostedZones[?Name=='insight-hr.io.vn.'].Id" \
    --output text)

# List and delete records (except NS and SOA)
aws route53 list-resource-record-sets \
    --hosted-zone-id $ZONE_ID

# Delete A records for CloudFront
cat > delete-records.json << EOF
{
  "Changes": [{
    "Action": "DELETE",
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
    --hosted-zone-id $ZONE_ID \
    --change-batch file://delete-records.json

# Delete hosted zone
aws route53 delete-hosted-zone --id $ZONE_ID
```

### Step 10: Delete ACM Certificates (If Created)

**Using AWS Console**:

1. Navigate to Certificate Manager (in us-east-1 region)
2. Select the certificate
3. Click "Delete"

**Using AWS CLI**:

```bash
# List certificates
aws acm list-certificates \
    --region us-east-1

# Delete certificate
CERT_ARN=$(aws acm list-certificates \
    --query "CertificateSummaryList[?DomainName=='insight-hr.io.vn'].CertificateArn" \
    --output text \
    --region us-east-1)

aws acm delete-certificate \
    --certificate-arn $CERT_ARN \
    --region us-east-1
```

### Automated Cleanup Script

Create a comprehensive cleanup script:

**cleanup-all.sh**:

```bash
#!/bin/bash

set -e

echo "Starting InsightHR cleanup..."

# 1. Disable and delete CloudFront
echo "Step 1: CloudFront..."
# (Add CloudFront cleanup commands)

# 2. Delete S3 buckets
echo "Step 2: S3 Buckets..."
aws s3 rm s3://insighthr-web-app-sg --recursive
aws s3 rb s3://insighthr-web-app-sg --force

# 3. Delete API Gateway
echo "Step 3: API Gateway..."
# (Add API Gateway cleanup commands)

# 4. Delete Lambda functions
echo "Step 4: Lambda Functions..."
# (Add Lambda cleanup commands)

# 5. Delete DynamoDB tables
echo "Step 5: DynamoDB Tables..."
# (Add DynamoDB cleanup commands)

# 6. Delete Cognito
echo "Step 6: Cognito User Pool..."
# (Add Cognito cleanup commands)

# 7. Delete CloudWatch logs
echo "Step 7: CloudWatch Logs..."
# (Add CloudWatch cleanup commands)

# 8. Delete IAM roles
echo "Step 8: IAM Roles and Policies..."
# (Add IAM cleanup commands)

echo "Cleanup complete!"
```

### Verification

After cleanup, verify all resources are deleted:

```bash
# Check S3 buckets
aws s3 ls | grep insighthr

# Check Lambda functions
aws lambda list-functions \
    --query 'Functions[?starts_with(FunctionName, `insighthr`)].FunctionName' \
    --region ap-southeast-1

# Check DynamoDB tables
aws dynamodb list-tables \
    --query 'TableNames[?starts_with(@, `insighthr`)]' \
    --region ap-southeast-1

# Check CloudFront distributions
aws cloudfront list-distributions \
    --query "DistributionList.Items[?Comment=='InsightHR CloudFront Distribution'].Id"

# Check API Gateway
aws apigateway get-rest-apis \
    --query "items[?name=='InsightHR API'].id" \
    --region ap-southeast-1
```

### Cost Verification

After cleanup, monitor your AWS billing:

1. Go to AWS Billing Dashboard
2. Check "Bills" for current month
3. Verify no ongoing charges for deleted services
4. Check "Cost Explorer" for trends

### Cleanup Checklist

- [ ] CloudFront distribution disabled and deleted
- [ ] S3 buckets emptied and deleted
- [ ] API Gateway deleted
- [ ] All Lambda functions deleted
- [ ] All DynamoDB tables deleted
- [ ] Cognito User Pool deleted
- [ ] CloudWatch log groups deleted
- [ ] CloudWatch Synthetics canaries deleted
- [ ] IAM roles and policies deleted
- [ ] Route53 hosted zone deleted (if created)
- [ ] ACM certificates deleted (if created)
- [ ] Billing dashboard checked for ongoing charges

### Troubleshooting Cleanup

**CloudFront won't delete**:
- Ensure distribution is fully disabled
- Wait 15-20 minutes after disabling
- Check for associated resources

**S3 bucket won't delete**:
- Ensure bucket is completely empty
- Check for versioned objects
- Disable versioning before deleting

**IAM role won't delete**:
- Detach all managed policies first
- Delete all inline policies
- Check for service-linked roles

**DynamoDB table deletion fails**:
- Wait for table to be in ACTIVE state
- Check for ongoing operations
- Verify IAM permissions

### Final Notes

{{% notice tip %}}
**Best Practice**: Always clean up resources after completing a workshop to avoid unexpected charges. Set up billing alerts to notify you of any ongoing costs.
{{% /notice %}}

### Congratulations!

You've successfully completed the InsightHR workshop and cleaned up all resources. You've learned:

- ✅ Serverless architecture design
- ✅ AWS Lambda and API Gateway
- ✅ DynamoDB data modeling
- ✅ Cognito authentication
- ✅ AWS Bedrock AI integration
- ✅ CloudFront CDN deployment
- ✅ Infrastructure as Code principles
- ✅ Cost optimization strategies

Thank you for participating in this workshop!
