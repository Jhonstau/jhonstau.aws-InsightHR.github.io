---
title: "Workshop"
date: "2025-12-09"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# InsightHR - Serverless HR Automation Platform Workshop

## Overview

**InsightHR** is a modern, fully serverless HR automation platform built on AWS that demonstrates best practices for cloud-native application development. This workshop guides you through building and deploying a complete production-ready application using AWS services.

## What You'll Build

A comprehensive HR management system featuring:

- **Employee Management**: Full CRUD operations with advanced filtering
- **Performance Tracking**: Quarterly performance scores and KPI management
- **Attendance System**: Real-time check-in/check-out with history tracking
- **AI Chatbot**: Natural language queries powered by AWS Bedrock (Claude 3)
- **Dashboard Analytics**: Interactive performance visualization
- **Role-Based Access Control**: Admin, Manager, and Employee roles
- **Authentication**: Email/password and Google OAuth via AWS Cognito

## Architecture Highlights

- ✅ **100% Serverless** - No EC2 instances to manage
- ✅ **Scalable** - Auto-scales with demand
- ✅ **Cost-Effective** - Pay only for what you use
- ✅ **Secure** - Built-in security with Cognito and IAM
- ✅ **Modern Stack** - React + TypeScript + Python
- ✅ **Production-Ready** - CloudWatch monitoring and custom domain

## AWS Services Used

- **Frontend**: S3 + CloudFront + Route53
- **Backend**: Lambda + API Gateway + DynamoDB
- **Authentication**: Cognito User Pools
- **AI/ML**: Amazon Bedrock (Claude 3 Haiku)
- **Monitoring**: CloudWatch + Synthetics Canaries
- **Security**: IAM + ACM (SSL Certificates)

## Workshop Content

1. [Workshop Overview](5.1-workshop-overview/)
2. [Prerequisites](5.2-prerequisite/)
3. [Project Architecture](5.3-architecture/)
4. [Setup AWS Environment](5.4-setup-aws/)
5. [Database Setup (DynamoDB)](5.5-database-setup/)
6. [Authentication Service](5.6-authentication/)
7. [Backend Services](5.7-backend-services/)
8. [Frontend Development](5.8-frontend/)
9. [Deployment](5.9-deployment/)
10. [Testing & Monitoring](5.10-testing/)
11. [Cleanup](5.11-cleanup/)

## Learning Outcomes

By completing this workshop, you will learn:

- How to design and implement serverless architectures
- Best practices for AWS Lambda and API Gateway
- DynamoDB data modeling and optimization
- AWS Cognito authentication flows
- Integration with AWS Bedrock for AI capabilities
- CloudFront CDN configuration
- Infrastructure as Code principles
- Production deployment strategies
- Cost optimization techniques

## Prerequisites

- AWS Account with appropriate permissions
- Basic knowledge of JavaScript/TypeScript and Python
- Familiarity with React framework
- Understanding of REST APIs
- AWS CLI installed and configured

## Estimated Time

- **Full Workshop**: 4-6 hours
- **Core Features Only**: 2-3 hours

## Cost Estimate

Running this workshop will incur minimal AWS costs:

- **DynamoDB**: ~$0.50/month (on-demand pricing)
- **Lambda**: Free tier covers most usage
- **S3 + CloudFront**: ~$1-2/month
- **API Gateway**: ~$0.10/month
- **Bedrock**: ~$0.0004 per query
- **Total**: ~$2-5/month for development

{{% notice tip %}}
Remember to clean up resources after completing the workshop to avoid ongoing charges.
{{% /notice %}}

## Support

For questions or issues during the workshop:

- Check the troubleshooting sections in each module
- Review AWS documentation links provided
- Refer to the GitHub repository for code samples

Let's get started! 🚀
