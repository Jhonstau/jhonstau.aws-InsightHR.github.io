---
title: "Authentication Service"
date: "2025-12-09"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Authentication Service Setup

This section covers setting up AWS Cognito for user authentication and implementing authentication Lambda functions.

### Overview

The authentication system includes:

1. **AWS Cognito User Pool** - User management and authentication
2. **Login Handler** - Email/password authentication
3. **Registration Handler** - New user signup
4. **Google OAuth Handler** - Social login integration
5. **Password Reset** - Password recovery workflow

### Step 1: Create Cognito User Pool

#### Using AWS Console

1. Navigate to Amazon Cognito
2. Click "Create user pool"
3. Configure sign-in experience:
   - Sign-in options: Email
   - User name requirements: Allow email addresses
   - Click "Next"

4. Configure security requirements:
   - Password policy: Cognito defaults
   - Multi-factor authentication: Optional
   - User account recovery: Email only
   - Click "Next"

5. Configure sign-up experience:
   - Self-registration: Enabled
   - Required attributes: name, email
   - Click "Next"

6. Configure message delivery:
   - Email provider: Send email with Cognito
   - FROM email address: no-reply@verificationemail.com
   - Click "Next"

7. Integrate your app:
   - User pool name: `insighthr-user-pool`
   - App client name: `insighthr-web-client`
   - Client secret: Don't generate
   - Authentication flows: ALLOW_USER_PASSWORD_AUTH, ALLOW_REFRESH_TOKEN_AUTH
   - Click "Next"

8. Review and create

#### Using AWS CLI

```bash
# Create user pool
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

# Get user pool ID
USER_POOL_ID=$(aws cognito-idp list-user-pools \
    --max-results 10 \
    --query "UserPools[?Name=='insighthr-user-pool'].Id" \
    --output text \
    --region ap-southeast-1)

echo "User Pool ID: $USER_POOL_ID"
```

#### Create App Client

```bash
# Create app client
aws cognito-idp create-user-pool-client \
    --user-pool-id $USER_POOL_ID \
    --client-name insighthr-web-client \
    --no-generate-secret \
    --explicit-auth-flows ALLOW_USER_PASSWORD_AUTH ALLOW_REFRESH_TOKEN_AUTH ALLOW_USER_SRP_AUTH \
    --region ap-southeast-1

# Get client ID
CLIENT_ID=$(aws cognito-idp list-user-pool-clients \
    --user-pool-id $USER_POOL_ID \
    --query "UserPoolClients[?ClientName=='insighthr-web-client'].ClientId" \
    --output text \
    --region ap-southeast-1)

echo "Client ID: $CLIENT_ID"
```

### Step 2: Configure Cognito for Development

For development, disable email verification requirement:

```bash
# Update user pool to auto-verify emails
aws cognito-idp update-user-pool \
    --user-pool-id $USER_POOL_ID \
    --auto-verified-attributes email \
    --region ap-southeast-1
```

### Step 3: Create Login Lambda Function

#### Create Function Code

**auth_login_handler.py**:
```python
import json
import boto3
import os
from botocore.exceptions import ClientError

cognito_client = boto3.client('cognito-idp')
dynamodb = boto3.resource('dynamodb')

USER_POOL_ID = os.environ['USER_POOL_ID']
CLIENT_ID = os.environ['CLIENT_ID']
USERS_TABLE = os.environ['DYNAMODB_USERS_TABLE']

def lambda_handler(event, context):
    try:
        body = json.loads(event['body'])
        email = body['email']
        password = body['password']
        
        # Authenticate with Cognito
        response = cognito_client.admin_initiate_auth(
            UserPoolId=USER_POOL_ID,
            ClientId=CLIENT_ID,
            AuthFlow='ADMIN_NO_SRP_AUTH',
            AuthParameters={
                'USERNAME': email,
                'PASSWORD': password
            }
        )
        
        # Get user attributes
        user_response = cognito_client.admin_get_user(
            UserPoolId=USER_POOL_ID,
            Username=email
        )
        
        # Extract user info
        user_attributes = {attr['Name']: attr['Value'] 
                          for attr in user_response['UserAttributes']}
        
        # Get user from DynamoDB
        table = dynamodb.Table(USERS_TABLE)
        db_response = table.get_item(
            Key={'userId': user_attributes['sub']}
        )
        
        user_data = db_response.get('Item', {})
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps({
                'tokens': {
                    'accessToken': response['AuthenticationResult']['AccessToken'],
                    'idToken': response['AuthenticationResult']['IdToken'],
                    'refreshToken': response['AuthenticationResult']['RefreshToken']
                },
                'user': {
                    'userId': user_attributes['sub'],
                    'email': user_attributes['email'],
                    'name': user_attributes.get('name', ''),
                    'role': user_data.get('role', 'Employee'),
                    'department': user_data.get('department', ''),
                    'employeeId': user_data.get('employeeId', '')
                }
            })
        }
        
    except ClientError as e:
        error_code = e.response['Error']['Code']
        if error_code == 'NotAuthorizedException':
            return {
                'statusCode': 401,
                'headers': {'Access-Control-Allow-Origin': '*'},
                'body': json.dumps({'error': 'Invalid credentials'})
            }
        elif error_code == 'UserNotFoundException':
            return {
                'statusCode': 404,
                'headers': {'Access-Control-Allow-Origin': '*'},
                'body': json.dumps({'error': 'User not found'})
            }
        else:
            return {
                'statusCode': 500,
                'headers': {'Access-Control-Allow-Origin': '*'},
                'body': json.dumps({'error': str(e)})
            }
    except Exception as e:
        return {
            'statusCode': 500,
            'headers': {'Access-Control-Allow-Origin': '*'},
            'body': json.dumps({'error': str(e)})
        }
```

#### Create requirements.txt

```
boto3>=1.26.0
```

#### Package and Deploy

```bash
# Create deployment package
mkdir -p lambda/auth/package
cd lambda/auth
pip install -r requirements.txt -t package/
cd package
zip -r ../auth-login-handler.zip .
cd ..
zip -g auth-login-handler.zip auth_login_handler.py

# Deploy Lambda function
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

### Step 4: Create Registration Lambda Function

**auth_register_handler.py**:
```python
import json
import boto3
import os
import uuid
from datetime import datetime
from botocore.exceptions import ClientError

cognito_client = boto3.client('cognito-idp')
dynamodb = boto3.resource('dynamodb')

USER_POOL_ID = os.environ['USER_POOL_ID']
CLIENT_ID = os.environ['CLIENT_ID']
USERS_TABLE = os.environ['DYNAMODB_USERS_TABLE']

def lambda_handler(event, context):
    try:
        body = json.loads(event['body'])
        email = body['email']
        password = body['password']
        name = body['name']
        role = body.get('role', 'Employee')
        department = body.get('department', '')
        employee_id = body.get('employeeId', '')
        
        # Create user in Cognito
        cognito_response = cognito_client.sign_up(
            ClientId=CLIENT_ID,
            Username=email,
            Password=password,
            UserAttributes=[
                {'Name': 'email', 'Value': email},
                {'Name': 'name', 'Value': name}
            ]
        )
        
        user_id = cognito_response['UserSub']
        
        # Auto-confirm user for development
        cognito_client.admin_confirm_sign_up(
            UserPoolId=USER_POOL_ID,
            Username=email
        )
        
        # Store user in DynamoDB
        table = dynamodb.Table(USERS_TABLE)
        timestamp = datetime.utcnow().isoformat() + 'Z'
        
        table.put_item(
            Item={
                'userId': user_id,
                'email': email,
                'name': name,
                'role': role,
                'department': department,
                'employeeId': employee_id,
                'isActive': True,
                'createdAt': timestamp,
                'updatedAt': timestamp
            }
        )
        
        return {
            'statusCode': 201,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps({
                'message': 'User registered successfully',
                'userId': user_id
            })
        }
        
    except ClientError as e:
        error_code = e.response['Error']['Code']
        if error_code == 'UsernameExistsException':
            return {
                'statusCode': 409,
                'headers': {'Access-Control-Allow-Origin': '*'},
                'body': json.dumps({'error': 'User already exists'})
            }
        else:
            return {
                'statusCode': 500,
                'headers': {'Access-Control-Allow-Origin': '*'},
                'body': json.dumps({'error': str(e)})
            }
    except Exception as e:
        return {
            'statusCode': 500,
            'headers': {'Access-Control-Allow-Origin': '*'},
            'body': json.dumps({'error': str(e)})
        }
```

Deploy the registration handler:
```bash
zip -g auth-register-handler.zip auth_register_handler.py

aws lambda create-function \
    --function-name insighthr-auth-register-handler \
    --runtime python3.11 \
    --role arn:aws:iam::${ACCOUNT_ID}:role/insighthr-lambda-role \
    --handler auth_register_handler.lambda_handler \
    --zip-file fileb://auth-register-handler.zip \
    --timeout 30 \
    --memory-size 256 \
    --environment Variables="{
        USER_POOL_ID=${USER_POOL_ID},
        CLIENT_ID=${CLIENT_ID},
        DYNAMODB_USERS_TABLE=insighthr-users-dev
    }" \
    --region ap-southeast-1
```

### Step 5: Create API Gateway Endpoints

#### Create REST API

```bash
# Create API
API_ID=$(aws apigateway create-rest-api \
    --name "InsightHR API" \
    --description "InsightHR Backend API" \
    --region ap-southeast-1 \
    --query 'id' \
    --output text)

echo "API ID: $API_ID"
```

#### Create Cognito Authorizer

```bash
# Create authorizer
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

#### Create /auth Resource

```bash
# Get root resource ID
ROOT_ID=$(aws apigateway get-resources \
    --rest-api-id $API_ID \
    --query "items[?path=='/'].id" \
    --output text \
    --region ap-southeast-1)

# Create /auth resource
AUTH_ID=$(aws apigateway create-resource \
    --rest-api-id $API_ID \
    --parent-id $ROOT_ID \
    --path-part auth \
    --query 'id' \
    --output text \
    --region ap-southeast-1)

# Create /auth/login resource
LOGIN_ID=$(aws apigateway create-resource \
    --rest-api-id $API_ID \
    --parent-id $AUTH_ID \
    --path-part login \
    --query 'id' \
    --output text \
    --region ap-southeast-1)

# Create POST method for /auth/login
aws apigateway put-method \
    --rest-api-id $API_ID \
    --resource-id $LOGIN_ID \
    --http-method POST \
    --authorization-type NONE \
    --region ap-southeast-1

# Integrate with Lambda
aws apigateway put-integration \
    --rest-api-id $API_ID \
    --resource-id $LOGIN_ID \
    --http-method POST \
    --type AWS_PROXY \
    --integration-http-method POST \
    --uri arn:aws:apigateway:ap-southeast-1:lambda:path/2015-03-31/functions/arn:aws:lambda:ap-southeast-1:${ACCOUNT_ID}:function:insighthr-auth-login-handler/invocations \
    --region ap-southeast-1

# Grant API Gateway permission to invoke Lambda
aws lambda add-permission \
    --function-name insighthr-auth-login-handler \
    --statement-id apigateway-invoke \
    --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com \
    --source-arn "arn:aws:execute-api:ap-southeast-1:${ACCOUNT_ID}:${API_ID}/*/*" \
    --region ap-southeast-1
```

### Step 6: Enable CORS

```bash
# Enable CORS for /auth/login
aws apigateway put-method \
    --rest-api-id $API_ID \
    --resource-id $LOGIN_ID \
    --http-method OPTIONS \
    --authorization-type NONE \
    --region ap-southeast-1

aws apigateway put-integration \
    --rest-api-id $API_ID \
    --resource-id $LOGIN_ID \
    --http-method OPTIONS \
    --type MOCK \
    --request-templates '{"application/json": "{\"statusCode\": 200}"}' \
    --region ap-southeast-1

aws apigateway put-method-response \
    --rest-api-id $API_ID \
    --resource-id $LOGIN_ID \
    --http-method OPTIONS \
    --status-code 200 \
    --response-parameters '{
        "method.response.header.Access-Control-Allow-Headers": true,
        "method.response.header.Access-Control-Allow-Methods": true,
        "method.response.header.Access-Control-Allow-Origin": true
    }' \
    --region ap-southeast-1

aws apigateway put-integration-response \
    --rest-api-id $API_ID \
    --resource-id $LOGIN_ID \
    --http-method OPTIONS \
    --status-code 200 \
    --response-parameters '{
        "method.response.header.Access-Control-Allow-Headers": "'"'"'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'"'"'",
        "method.response.header.Access-Control-Allow-Methods": "'"'"'POST,OPTIONS'"'"'",
        "method.response.header.Access-Control-Allow-Origin": "'"'"'*'"'"'"
    }' \
    --region ap-southeast-1
```

### Step 7: Deploy API

```bash
# Create deployment
aws apigateway create-deployment \
    --rest-api-id $API_ID \
    --stage-name dev \
    --description "Initial deployment" \
    --region ap-southeast-1

# Get API endpoint
echo "API Endpoint: https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/dev"
```

### Step 8: Test Authentication

#### Test Registration

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

#### Test Login

```bash
curl -X POST \
    https://${API_ID}.execute-api.ap-southeast-1.amazonaws.com/dev/auth/login \
    -H "Content-Type: application/json" \
    -d '{
        "email": "test@example.com",
        "password": "Test123!"
    }'
```

### Authentication Flow Summary

```
1. User submits credentials → Frontend
2. Frontend calls /auth/login → API Gateway
3. API Gateway routes to Lambda → auth-login-handler
4. Lambda authenticates with Cognito → Cognito User Pool
5. Cognito returns JWT tokens → Lambda
6. Lambda retrieves user data → DynamoDB
7. Lambda returns tokens + user data → Frontend
8. Frontend stores tokens → localStorage
9. Subsequent requests include JWT → Authorization header
```

### Security Best Practices

✅ **Password Policy**: Strong password requirements
✅ **JWT Tokens**: Short-lived access tokens
✅ **Refresh Tokens**: Long-lived for token renewal
✅ **HTTPS Only**: All communication encrypted
✅ **CORS**: Properly configured for frontend domain
✅ **No Secrets**: Client doesn't use client secret

### Troubleshooting

**Cognito Authentication Fails**:
- Verify user pool ID and client ID
- Check password meets requirements
- Ensure user is confirmed

**Lambda Permission Denied**:
- Check IAM role has Cognito permissions
- Verify Lambda execution role attached

**CORS Errors**:
- Enable CORS on API Gateway
- Check Access-Control-Allow-Origin header
- Verify OPTIONS method configured

### Next Steps

With authentication configured, proceed to [Backend Services](../5.7-backend-services/) to implement employee, performance, and chatbot APIs.
