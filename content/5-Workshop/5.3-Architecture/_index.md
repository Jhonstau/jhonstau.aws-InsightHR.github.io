---
title: "Project Architecture"
date: "2025-12-09"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## InsightHR Architecture Deep Dive

This section provides a detailed look at the InsightHR platform architecture, explaining how different AWS services work together to create a scalable, secure, and cost-effective serverless application.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Browser                             │
│                    (React + TypeScript)                          │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CloudFront CDN                                │
│              - Global edge locations                             │
│              - SSL/TLS termination                               │
│              - Caching static assets                             │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    S3 Static Website                             │
│              - React SPA hosting                                 │
│              - Static assets (JS, CSS, images)                   │
└─────────────────────────────────────────────────────────────────┘
                         │ REST API calls
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    API Gateway (REST)                            │
│              - Request routing                                   │
│              - Cognito authorization                             │
│              - Request/response transformation                   │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┬───────────────┐
         ▼               ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Lambda     │  │   Lambda     │  │   Lambda     │  │   Lambda     │
│   Auth       │  │  Employees   │  │ Performance  │  │  Chatbot     │
│              │  │              │  │   Scores     │  │              │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        DynamoDB                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │  Users   │ │Employees │ │  Scores  │ │Attendance│          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AWS Bedrock                                   │
│              (Claude 3 Haiku Model)                              │
│              - Natural language processing                       │
│              - Context-aware responses                           │
└─────────────────────────────────────────────────────────────────┘
```

### Component Breakdown

#### 1. Frontend Layer

**Amazon S3 + CloudFront**

- **S3 Bucket**: Hosts the React SPA as static files
  - Configured for static website hosting
  - Stores HTML, JavaScript, CSS, and image assets
  - Versioning enabled for rollback capability

- **CloudFront Distribution**: Global CDN for fast content delivery
  - Edge locations worldwide for low latency
  - SSL/TLS certificate from ACM
  - Custom domain support (insight-hr.io.vn)
  - Caching policies for optimal performance
  - Origin Access Identity (OAI) for S3 security

**React Application**
- Single Page Application (SPA) architecture
- Client-side routing with React Router
- State management with Zustand
- TypeScript for type safety
- Tailwind CSS for styling

#### 2. API Layer

**Amazon API Gateway (REST)**

- **Endpoints**: RESTful API design
  - `/auth/*` - Authentication endpoints
  - `/employees/*` - Employee management
  - `/performance-scores/*` - Performance tracking
  - `/attendance/*` - Attendance management
  - `/chatbot/*` - AI chatbot queries

- **Features**:
  - Request validation
  - CORS configuration
  - Rate limiting and throttling
  - Request/response transformation
  - API keys and usage plans

- **Authorization**: Cognito User Pool Authorizer
  - JWT token validation
  - Role-based access control
  - Automatic token refresh

#### 3. Compute Layer

**AWS Lambda Functions**

**Authentication Service** (`auth-*-handler`)
- Login with Cognito
- User registration
- Google OAuth integration
- Password reset workflow
- Token management

**Employee Service** (`employees-*-handler`)
- CRUD operations for employees
- Department and position filtering
- Bulk import from CSV
- Search functionality

**Performance Scores Service** (`performance-scores-handler`)
- Score calculation and management
- Quarterly performance tracking
- KPI aggregation
- Role-based data access

**Chatbot Service** (`chatbot-handler`)
- Natural language query processing
- Context building from DynamoDB
- Bedrock API integration
- Response formatting

**Attendance Service** (`attendance-handler`)
- Check-in/check-out operations
- Attendance history tracking
- Status management
- Bulk operations

**Lambda Configuration**:
- Runtime: Python 3.11
- Memory: 256-512 MB
- Timeout: 30-60 seconds
- Environment variables for configuration
- IAM roles with least privilege

#### 4. Data Layer

**Amazon DynamoDB**

**Tables Structure**:

1. **insighthr-users-dev**
   - Primary Key: `userId`
   - GSI: `email-index`
   - Purpose: User authentication and profiles
   - Attributes: email, name, role, employeeId, department

2. **insighthr-employees-dev**
   - Primary Key: `employeeId`
   - GSI: `department-index`
   - Purpose: Employee master data
   - Attributes: name, email, department, position, status

3. **insighthr-performance-scores-dev**
   - Primary Key: `employeeId`
   - Sort Key: `period`
   - GSI: `department-period-index`
   - Purpose: Quarterly performance scores
   - Attributes: overallScore, kpiScores, calculatedAt

4. **insighthr-attendance-history-dev**
   - Primary Key: `employeeId`
   - Sort Key: `date`
   - GSI: `date-index`, `department-date-index`
   - Purpose: Check-in/check-out history
   - Attributes: checkInTime, checkOutTime, status

**DynamoDB Features**:
- On-demand billing mode
- Point-in-time recovery
- Encryption at rest
- Global secondary indexes for efficient queries
- TTL for automatic data expiration (if needed)

#### 5. Authentication Layer

**Amazon Cognito User Pools**

- **User Management**:
  - Email/password authentication
  - Google OAuth integration
  - User attributes (name, role, department)
  - Password policies and MFA support

- **Token Management**:
  - JWT tokens (ID, Access, Refresh)
  - Token expiration and refresh
  - Custom claims for roles

- **Security Features**:
  - Password complexity requirements
  - Account recovery workflows
  - User verification
  - Brute force protection

#### 6. AI/ML Layer

**Amazon Bedrock (Claude 3 Haiku)**

- **Capabilities**:
  - Natural language understanding
  - Context-aware responses
  - Data querying and analysis
  - Conversational interface

- **Integration**:
  - Invoked from Lambda function
  - Context built from DynamoDB data
  - Role-based data filtering
  - Response formatting and validation

- **Cost Optimization**:
  - Haiku model for cost-effectiveness
  - Efficient prompt engineering
  - Response caching where applicable

#### 7. Monitoring Layer

**Amazon CloudWatch**

- **Logs**:
  - Lambda function logs
  - API Gateway access logs
  - Error tracking and debugging

- **Metrics**:
  - API request counts
  - Lambda invocations and duration
  - DynamoDB read/write capacity
  - Error rates and latency

- **Alarms**:
  - High error rate alerts
  - Performance degradation
  - Cost threshold warnings

**CloudWatch Synthetics Canaries**:
- Login flow testing
- Dashboard availability
- Chatbot functionality
- Performance score calculations

### Data Flow Examples

#### User Login Flow

```
1. User enters credentials in React app
2. React app calls API Gateway /auth/login
3. API Gateway routes to auth-login-handler Lambda
4. Lambda authenticates with Cognito
5. Cognito returns JWT tokens
6. Lambda stores user session in DynamoDB
7. Tokens returned to React app
8. React app stores tokens in localStorage
9. Subsequent requests include JWT in Authorization header
```

#### Employee Query Flow

```
1. User requests employee list in React app
2. React app calls API Gateway /employees with filters
3. API Gateway validates JWT token with Cognito
4. Request routed to employees-handler Lambda
5. Lambda queries DynamoDB employees table
6. Results filtered based on user role
7. Data returned to React app
8. React app displays employees in table
```

#### Chatbot Query Flow

```
1. User types question in chatbot interface
2. React app calls API Gateway /chatbot/query
3. API Gateway validates JWT and routes to chatbot-handler
4. Lambda retrieves relevant data from DynamoDB
5. Lambda builds context and calls Bedrock API
6. Bedrock processes query with Claude 3 Haiku
7. Response formatted and returned to React app
8. React app displays answer in chat interface
```

### Security Architecture

**Defense in Depth**:

1. **Network Security**:
   - HTTPS only (enforced by CloudFront)
   - API Gateway with AWS WAF (optional)
   - VPC endpoints for private connectivity (optional)

2. **Authentication & Authorization**:
   - Cognito for user authentication
   - JWT tokens for API authorization
   - Role-based access control (RBAC)
   - Least privilege IAM roles

3. **Data Security**:
   - Encryption at rest (DynamoDB, S3)
   - Encryption in transit (TLS 1.2+)
   - Secure credential management
   - No hardcoded secrets

4. **Application Security**:
   - Input validation
   - SQL injection prevention (NoSQL)
   - XSS protection
   - CORS configuration

### Scalability & Performance

**Auto-Scaling**:
- Lambda: Automatic concurrent execution scaling
- DynamoDB: On-demand capacity mode
- CloudFront: Global edge network
- API Gateway: Automatic request handling

**Performance Optimization**:
- CloudFront caching for static assets
- DynamoDB GSIs for efficient queries
- Lambda function optimization
- API response caching

**High Availability**:
- Multi-AZ deployment (automatic)
- CloudFront global distribution
- DynamoDB replication
- Lambda fault tolerance

### Cost Optimization

**Strategies**:
- On-demand pricing for variable workloads
- Lambda free tier utilization
- CloudFront caching to reduce origin requests
- DynamoDB query optimization
- Bedrock Haiku model for cost-effectiveness

**Estimated Monthly Costs** (Development):
- DynamoDB: $0.50
- Lambda: Free tier
- S3 + CloudFront: $1-2
- API Gateway: $0.10
- Bedrock: $0.0004 per query
- **Total**: $2-5/month

### Next Steps

Now that you understand the architecture, let's proceed to [Setup AWS Environment](../5.4-setup-aws/) to start building the platform.
