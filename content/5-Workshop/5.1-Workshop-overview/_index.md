---
title: "Workshop Overview"
date: "2025-12-09"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## InsightHR Platform Overview

**InsightHR** is a comprehensive serverless HR automation platform that demonstrates modern cloud-native application development on AWS. This workshop will guide you through building a production-ready application from scratch.

### Project Scope

The InsightHR platform provides:

- **Employee Management System**: Complete CRUD operations for employee records with advanced filtering by department, position, and status
- **Performance Tracking**: Quarterly performance scores with automatic calculation based on KPIs, completed tasks, and 360-degree feedback
- **Attendance Management**: Real-time check-in/check-out system with historical tracking and status monitoring
- **AI-Powered Chatbot**: Natural language query interface using AWS Bedrock (Claude 3 Haiku) for intelligent data insights
- **Analytics Dashboard**: Interactive visualizations with charts, tables, and export capabilities
- **Role-Based Access Control**: Three-tier access system (Admin, Manager, Employee) with appropriate data filtering
- **Secure Authentication**: Email/password and Google OAuth integration via AWS Cognito

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Browser                             │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CloudFront CDN                                │
│              Custom Domain: insight-hr.io.vn                     │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    S3 Static Website                             │
│              React SPA (Vite + TypeScript)                       │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    API Gateway (REST)                            │
│                  Cognito Authorizer                              │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Lambda     │  │   Lambda     │  │   Lambda     │
│   Auth       │  │  Employees   │  │  Chatbot     │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       ▼                 ▼                 ▼
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
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack

**Frontend:**
- React 18 with TypeScript
- Vite 7.2 (build tool)
- Tailwind CSS 3.4 (Frutiger Aero theme)
- Zustand 5.0 (state management)
- React Hook Form + Zod (form validation)
- Recharts 3.4 (data visualization)

**Backend:**
- Python 3.11 (Lambda runtime)
- AWS Lambda (serverless compute)
- AWS API Gateway (REST API)
- AWS DynamoDB (NoSQL database)
- AWS Cognito (authentication)
- AWS Bedrock (AI/ML)

**Infrastructure:**
- AWS S3 (static hosting)
- AWS CloudFront (CDN)
- AWS Route53 (DNS)
- AWS CloudWatch (monitoring)
- AWS IAM (security)

### Key Features

#### 1. Fully Serverless Architecture
- No EC2 instances to manage
- Automatic scaling based on demand
- Pay-per-use pricing model
- High availability built-in

#### 2. Modern Development Stack
- TypeScript for type safety
- React for responsive UI
- Python for backend logic
- Infrastructure as Code principles

#### 3. Production-Ready
- Custom domain with SSL
- CloudWatch monitoring
- Synthetic canaries for testing
- Role-based access control

#### 4. Cost-Effective
- DynamoDB on-demand pricing
- Lambda free tier eligible
- Minimal monthly costs (~$2-5)
- No idle resource charges

### Workshop Structure

This workshop is divided into 11 modules:

1. **Workshop Overview** (Current) - Understanding the project scope
2. **Prerequisites** - Setting up your environment
3. **Project Architecture** - Deep dive into system design
4. **Setup AWS Environment** - Configuring AWS account and credentials
5. **Database Setup** - Creating and populating DynamoDB tables
6. **Authentication Service** - Implementing Cognito and auth Lambda functions
7. **Backend Services** - Building employee, performance, and chatbot APIs
8. **Frontend Development** - Creating the React application
9. **Deployment** - Deploying to S3 and CloudFront
10. **Testing & Monitoring** - Setting up CloudWatch and canaries
11. **Cleanup** - Removing resources to avoid charges

### Learning Objectives

By the end of this workshop, you will be able to:

- Design and implement serverless architectures on AWS
- Build RESTful APIs using Lambda and API Gateway
- Model data effectively in DynamoDB
- Implement authentication with AWS Cognito
- Integrate AI capabilities using AWS Bedrock
- Deploy static websites with S3 and CloudFront
- Monitor applications with CloudWatch
- Apply security best practices with IAM
- Optimize costs for serverless applications

### Prerequisites Check

Before proceeding, ensure you have:

- ✅ AWS Account with admin access
- ✅ AWS CLI installed and configured
- ✅ Node.js 18+ and npm installed
- ✅ Python 3.11+ installed
- ✅ Basic understanding of React and TypeScript
- ✅ Familiarity with REST APIs
- ✅ Text editor or IDE (VS Code recommended)

### Estimated Costs

Running this workshop will incur minimal costs:

| Service | Estimated Cost |
|---------|---------------|
| DynamoDB | $0.50/month |
| Lambda | Free tier |
| S3 + CloudFront | $1-2/month |
| API Gateway | $0.10/month |
| Bedrock | $0.0004/query |
| **Total** | **$2-5/month** |

{{% notice warning %}}
Remember to complete the cleanup module at the end to avoid ongoing charges.
{{% /notice %}}

### Next Steps

Ready to begin? Let's move to the [Prerequisites](../5.2-prerequisite/) section to set up your development environment.
