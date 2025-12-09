---
title: "Backend Services"
date: "2025-12-09"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Backend Services Implementation

This section covers implementing the core backend Lambda functions for employee management, performance tracking, attendance, and AI chatbot.

### Overview

We'll implement four main services:

1. **Employee Service** - CRUD operations for employee records
2. **Performance Scores Service** - Quarterly performance tracking
3. **Attendance Service** - Check-in/check-out management
4. **Chatbot Service** - AI-powered natural language queries

### Employee Service

The employee service handles all employee-related operations with department and position filtering.

**Key Features**:
- CRUD operations (Create, Read, Update, Delete)
- Department filtering (DEV, QA, DAT, SEC, AI)
- Position filtering (Junior, Mid, Senior, Lead, Manager)
- Status filtering (active, inactive)
- Search by name or employee ID
- Bulk import from CSV

**API Endpoints**:
```
GET    /employees              - List all employees
GET    /employees/{id}         - Get employee by ID
POST   /employees              - Create new employee
PUT    /employees/{id}         - Update employee
DELETE /employees/{id}         - Delete employee
POST   /employees/bulk         - Bulk import from CSV
```

### Performance Scores Service

Manages quarterly performance scores with automatic calculation.

**Key Features**:
- Automatic score calculation (average of KPI, completed_task, feedback_360)
- Department and period filtering
- Role-based data access (Admin: all, Manager: department, Employee: own)
- Denormalized employee data for performance
- Bulk import from CSV

**API Endpoints**:
```
GET    /performance-scores                    - List all scores
GET    /performance-scores/{id}/{period}      - Get specific score
POST   /performance-scores                    - Create score
PUT    /performance-scores/{id}/{period}      - Update score
DELETE /performance-scores/{id}/{period}      - Delete score
POST   /performance-scores/bulk               - Bulk import
```

### Attendance Service

Tracks employee attendance with check-in/check-out functionality.

**Key Features**:
- Real-time check-in/check-out
- Date range filtering
- Department filtering
- Status tracking (present, absent, late, half-day)
- Bulk operations

**API Endpoints**:
```
GET    /attendance                    - List attendance records
GET    /attendance/{id}/{date}        - Get specific record
POST   /attendance/check-in           - Check in
POST   /attendance/check-out          - Check out
PUT    /attendance/{id}/{date}        - Update record
DELETE /attendance/{id}/{date}        - Delete record
```

### Chatbot Service

AI-powered chatbot using AWS Bedrock (Claude 3 Haiku).

**Key Features**:
- AWS Bedrock integration (Claude 3 Haiku)
- Natural language query processing
- Context building from DynamoDB
- Role-based data filtering
- Cost-effective ($0.0004 per query)

**Supported Queries**:
- Employee information ("Who is DEV-001?")
- Performance queries ("What's the average score for Q1 2025?")
- Department statistics ("Compare DEV and QA performance")
- Trend analysis ("Performance trends over quarters")

**API Endpoint**:
```
POST /chatbot/query
```

### Implementation Pattern

All services follow this pattern:

1. **Request Validation** - Validate input parameters
2. **Authorization** - Check user permissions based on role
3. **DynamoDB Operations** - Query or update data
4. **Response Formatting** - Return standardized JSON response
5. **Error Handling** - Catch and return appropriate errors

### Deployment

Deploy all backend services using the Lambda deployment scripts provided in the workshop materials. Each service is packaged with its dependencies and deployed to AWS Lambda with appropriate environment variables.

### Testing

Test each service using the provided test scripts or curl commands. Verify:
- CRUD operations work correctly
- Filtering and search functionality
- Role-based access control
- Error handling
- Performance and response times

### Next Steps

With backend services implemented, proceed to [Frontend Development](../5.8-frontend/) to build the React application.
