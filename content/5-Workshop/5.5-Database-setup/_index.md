---
title: "Database Setup (DynamoDB)"
date: "2025-12-09"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## DynamoDB Database Setup

In this section, you'll create and configure the DynamoDB tables required for the InsightHR platform.

### Overview

InsightHR uses Amazon DynamoDB, a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability. We'll create 4 main tables:

1. **Users Table** - Authentication and user profiles
2. **Employees Table** - Employee master data
3. **Performance Scores Table** - Quarterly performance tracking
4. **Attendance History Table** - Check-in/check-out records

### Table Design Principles

**DynamoDB Best Practices**:
- Use partition keys for even data distribution
- Add sort keys for range queries
- Create GSIs for alternate query patterns
- Denormalize data for read performance
- Use on-demand billing for variable workloads

### Step 1: Create Users Table

**Table Configuration**:
- **Table Name**: `insighthr-users-dev`
- **Partition Key**: `userId` (String)
- **Billing Mode**: On-demand

**Using AWS Console**:

1. Navigate to DynamoDB in AWS Console
2. Click "Create table"
3. Enter table name: `insighthr-users-dev`
4. Partition key: `userId` (String)
5. Table settings: Use default settings
6. Billing mode: On-demand
7. Click "Create table"

**Add Global Secondary Index**:

1. Select the table
2. Go to "Indexes" tab
3. Click "Create index"
4. Index name: `email-index`
5. Partition key: `email` (String)
6. Projected attributes: All
7. Click "Create index"

**Using AWS CLI**:

```bash
# Create table
aws dynamodb create-table \
    --table-name insighthr-users-dev \
    --attribute-definitions \
        AttributeName=userId,AttributeType=S \
        AttributeName=email,AttributeType=S \
    --key-schema \
        AttributeName=userId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1

# Create GSI
aws dynamodb update-table \
    --table-name insighthr-users-dev \
    --attribute-definitions \
        AttributeName=email,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"email-index\",\"KeySchema\":[{\"AttributeName\":\"email\",\"KeyType\":\"HASH\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1
```

**Sample Data Structure**:
```json
{
  "userId": "user-123",
  "email": "admin@example.com",
  "name": "Admin User",
  "role": "Admin",
  "employeeId": "DEV-001",
  "department": "DEV",
  "isActive": true,
  "createdAt": "2025-12-09T00:00:00Z",
  "updatedAt": "2025-12-09T00:00:00Z"
}
```

### Step 2: Create Employees Table

**Table Configuration**:
- **Table Name**: `insighthr-employees-dev`
- **Partition Key**: `employeeId` (String)
- **Billing Mode**: On-demand

**Using AWS CLI**:

```bash
# Create table
aws dynamodb create-table \
    --table-name insighthr-employees-dev \
    --attribute-definitions \
        AttributeName=employeeId,AttributeType=S \
        AttributeName=department,AttributeType=S \
    --key-schema \
        AttributeName=employeeId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1

# Create GSI for department queries
aws dynamodb update-table \
    --table-name insighthr-employees-dev \
    --attribute-definitions \
        AttributeName=department,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"department-index\",\"KeySchema\":[{\"AttributeName\":\"department\",\"KeyType\":\"HASH\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1
```

**Sample Data Structure**:
```json
{
  "employeeId": "DEV-001",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "department": "DEV",
  "position": "Senior",
  "status": "active",
  "hireDate": "2024-01-01",
  "managerId": "DEV-000",
  "createdAt": "2025-12-09T00:00:00Z",
  "updatedAt": "2025-12-09T00:00:00Z"
}
```

### Step 3: Create Performance Scores Table

**Table Configuration**:
- **Table Name**: `insighthr-performance-scores-dev`
- **Partition Key**: `employeeId` (String)
- **Sort Key**: `period` (String)
- **Billing Mode**: On-demand

**Using AWS CLI**:

```bash
# Create table
aws dynamodb create-table \
    --table-name insighthr-performance-scores-dev \
    --attribute-definitions \
        AttributeName=employeeId,AttributeType=S \
        AttributeName=period,AttributeType=S \
        AttributeName=department,AttributeType=S \
    --key-schema \
        AttributeName=employeeId,KeyType=HASH \
        AttributeName=period,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1

# Create GSI for department-period queries
aws dynamodb update-table \
    --table-name insighthr-performance-scores-dev \
    --attribute-definitions \
        AttributeName=department,AttributeType=S \
        AttributeName=period,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"department-period-index\",\"KeySchema\":[{\"AttributeName\":\"department\",\"KeyType\":\"HASH\"},{\"AttributeName\":\"period\",\"KeyType\":\"RANGE\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1
```

**Sample Data Structure**:
```json
{
  "scoreId": "score-123",
  "employeeId": "DEV-001",
  "period": "2025-Q1",
  "employeeName": "John Doe",
  "department": "DEV",
  "position": "Senior",
  "overallScore": 85.5,
  "kpiScores": {
    "KPI": 85.0,
    "completed_task": 88.0,
    "feedback_360": 83.5
  },
  "calculatedAt": "2025-12-09T00:00:00Z",
  "createdAt": "2025-12-09T00:00:00Z",
  "updatedAt": "2025-12-09T00:00:00Z"
}
```

### Step 4: Create Attendance History Table

**Table Configuration**:
- **Table Name**: `insighthr-attendance-history-dev`
- **Partition Key**: `employeeId` (String)
- **Sort Key**: `date` (String)
- **Billing Mode**: On-demand

**Using AWS CLI**:

```bash
# Create table
aws dynamodb create-table \
    --table-name insighthr-attendance-history-dev \
    --attribute-definitions \
        AttributeName=employeeId,AttributeType=S \
        AttributeName=date,AttributeType=S \
        AttributeName=department,AttributeType=S \
    --key-schema \
        AttributeName=employeeId,KeyType=HASH \
        AttributeName=date,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1

# Create GSI for date queries
aws dynamodb update-table \
    --table-name insighthr-attendance-history-dev \
    --attribute-definitions \
        AttributeName=date,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"date-index\",\"KeySchema\":[{\"AttributeName\":\"date\",\"KeyType\":\"HASH\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1

# Create GSI for department-date queries
aws dynamodb update-table \
    --table-name insighthr-attendance-history-dev \
    --attribute-definitions \
        AttributeName=department,AttributeType=S \
        AttributeName=date,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\":\"department-date-index\",\"KeySchema\":[{\"AttributeName\":\"department\",\"KeyType\":\"HASH\"},{\"AttributeName\":\"date\",\"KeyType\":\"RANGE\"}],\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --region ap-southeast-1
```

**Sample Data Structure**:
```json
{
  "employeeId": "DEV-001",
  "date": "2025-12-09",
  "employeeName": "John Doe",
  "department": "DEV",
  "checkInTime": "09:00:00",
  "checkOutTime": "18:00:00",
  "status": "present",
  "notes": "On time",
  "createdAt": "2025-12-09T09:00:00Z",
  "updatedAt": "2025-12-09T18:00:00Z"
}
```

### Step 5: Verify Table Creation

**Using AWS Console**:
1. Navigate to DynamoDB
2. Check that all 4 tables are listed
3. Verify each table status is "Active"
4. Check GSIs are created and active

**Using AWS CLI**:

```bash
# List all tables
aws dynamodb list-tables --region ap-southeast-1

# Describe specific table
aws dynamodb describe-table \
    --table-name insighthr-users-dev \
    --region ap-southeast-1
```

### Step 6: Load Sample Data (Optional)

For testing purposes, you can load sample data into the tables.

**Create sample data file** (`sample-users.json`):
```json
{
  "insighthr-users-dev": [
    {
      "PutRequest": {
        "Item": {
          "userId": {"S": "user-001"},
          "email": {"S": "admin@example.com"},
          "name": {"S": "Admin User"},
          "role": {"S": "Admin"},
          "employeeId": {"S": "DEV-001"},
          "department": {"S": "DEV"},
          "isActive": {"BOOL": true},
          "createdAt": {"S": "2025-12-09T00:00:00Z"},
          "updatedAt": {"S": "2025-12-09T00:00:00Z"}
        }
      }
    }
  ]
}
```

**Load data**:
```bash
aws dynamodb batch-write-item \
    --request-items file://sample-users.json \
    --region ap-southeast-1
```

### Database Configuration Summary

| Table | Partition Key | Sort Key | GSIs | Purpose |
|-------|--------------|----------|------|---------|
| insighthr-users-dev | userId | - | email-index | User authentication |
| insighthr-employees-dev | employeeId | - | department-index | Employee data |
| insighthr-performance-scores-dev | employeeId | period | department-period-index | Performance tracking |
| insighthr-attendance-history-dev | employeeId | date | date-index, department-date-index | Attendance records |

### Best Practices Implemented

✅ **Partition Key Design**: Even distribution of data
✅ **Sort Keys**: Enable range queries for time-series data
✅ **GSIs**: Support alternate query patterns
✅ **On-Demand Billing**: Cost-effective for variable workloads
✅ **Naming Convention**: Consistent table naming with environment suffix

### Troubleshooting

**Table Creation Fails**:
- Check IAM permissions for DynamoDB
- Verify region is correct
- Ensure table name doesn't already exist

**GSI Creation Fails**:
- Wait for table to be ACTIVE before creating GSI
- Check attribute definitions match
- Verify IAM permissions

**High Costs**:
- Use on-demand billing for development
- Monitor read/write capacity units
- Optimize query patterns

### Next Steps

With the database tables created, proceed to [Authentication Service](../5.6-authentication/) to set up Cognito and authentication Lambda functions.
