---
title: "Testing & Monitoring"
date: "2025-12-09"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Testing & Monitoring

This section covers setting up comprehensive testing and monitoring for the InsightHR platform.

### Overview

We'll implement:

1. **CloudWatch Logs** - Centralized logging
2. **CloudWatch Metrics** - Performance monitoring
3. **CloudWatch Alarms** - Automated alerting
4. **CloudWatch Synthetics** - Automated testing
5. **X-Ray Tracing** - Distributed tracing (optional)

### CloudWatch Logs

All Lambda functions automatically log to CloudWatch Logs.

**View Logs**:
```bash
# List log groups
aws logs describe-log-groups \
    --log-group-name-prefix "/aws/lambda/insighthr" \
    --region ap-southeast-1

# Tail logs for a function
aws logs tail /aws/lambda/insighthr-auth-login-handler \
    --follow \
    --region ap-southeast-1
```

**Log Insights Queries**:

Find errors:
```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

Analyze response times:
```sql
fields @timestamp, @duration
| stats avg(@duration), max(@duration), min(@duration)
```

### CloudWatch Metrics

Monitor key metrics:

**Lambda Metrics**:
- Invocations
- Duration
- Errors
- Throttles
- Concurrent executions

**DynamoDB Metrics**:
- Read/Write capacity units
- Throttled requests
- System errors

**API Gateway Metrics**:
- Request count
- Latency
- 4XX/5XX errors

**View Metrics**:
```bash
# Lambda invocations
aws cloudwatch get-metric-statistics \
    --namespace AWS/Lambda \
    --metric-name Invocations \
    --dimensions Name=FunctionName,Value=insighthr-auth-login-handler \
    --start-time 2025-12-09T00:00:00Z \
    --end-time 2025-12-09T23:59:59Z \
    --period 3600 \
    --statistics Sum \
    --region ap-southeast-1
```

### CloudWatch Alarms

Create alarms for critical metrics:

**High Error Rate Alarm**:
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name insighthr-high-error-rate \
    --alarm-description "Alert when Lambda error rate exceeds 5%" \
    --metric-name Errors \
    --namespace AWS/Lambda \
    --statistic Sum \
    --period 300 \
    --evaluation-periods 2 \
    --threshold 5 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=FunctionName,Value=insighthr-auth-login-handler \
    --region ap-southeast-1
```

**High Latency Alarm**:
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name insighthr-high-latency \
    --alarm-description "Alert when API latency exceeds 2 seconds" \
    --metric-name Latency \
    --namespace AWS/ApiGateway \
    --statistic Average \
    --period 300 \
    --evaluation-periods 2 \
    --threshold 2000 \
    --comparison-operator GreaterThanThreshold \
    --region ap-southeast-1
```

### CloudWatch Synthetics Canaries

Automated testing with synthetic canaries:

**1. Login Canary** - Tests login flow
**2. Dashboard Canary** - Tests dashboard loading
**3. Chatbot Canary** - Tests chatbot queries
**4. Autoscoring Canary** - Tests performance calculations

**Create Login Canary**:

```javascript
// login-canary.js
const synthetics = require('Synthetics');
const log = require('SyntheticsLogger');

const loginFlow = async function () {
    const page = await synthetics.getPage();
    
    // Navigate to login page
    await page.goto('https://insight-hr.io.vn/login', {
        waitUntil: 'domcontentloaded',
        timeout: 30000
    });
    
    // Fill login form
    await page.type('#email', 'test@example.com');
    await page.type('#password', 'Test123!');
    
    // Click login button
    await Promise.all([
        page.waitForNavigation(),
        page.click('button[type="submit"]')
    ]);
    
    // Verify successful login
    await page.waitForSelector('.dashboard', { timeout: 10000 });
    
    log.info('Login flow completed successfully');
};

exports.handler = async () => {
    return await synthetics.executeStep('LoginFlow', loginFlow);
};
```

**Deploy Canary**:
```bash
# Create S3 bucket for canary artifacts
aws s3 mb s3://insighthr-canary-artifacts --region ap-southeast-1

# Create canary
aws synthetics create-canary \
    --name insighthr-login-canary \
    --artifact-s3-location s3://insighthr-canary-artifacts \
    --execution-role-arn arn:aws:iam::${ACCOUNT_ID}:role/CloudWatchSyntheticsRole \
    --schedule Expression="rate(5 minutes)" \
    --runtime-version syn-nodejs-puppeteer-6.2 \
    --code file://login-canary.zip \
    --region ap-southeast-1
```

### Dashboard Setup

Create a CloudWatch Dashboard:

```bash
aws cloudwatch put-dashboard \
    --dashboard-name InsightHR-Dashboard \
    --dashboard-body file://dashboard-config.json \
    --region ap-southeast-1
```

**dashboard-config.json**:
```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/Lambda", "Invocations", {"stat": "Sum"}],
          [".", "Errors", {"stat": "Sum"}],
          [".", "Duration", {"stat": "Average"}]
        ],
        "period": 300,
        "stat": "Average",
        "region": "ap-southeast-1",
        "title": "Lambda Metrics"
      }
    },
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/ApiGateway", "Count", {"stat": "Sum"}],
          [".", "4XXError", {"stat": "Sum"}],
          [".", "5XXError", {"stat": "Sum"}]
        ],
        "period": 300,
        "stat": "Sum",
        "region": "ap-southeast-1",
        "title": "API Gateway Metrics"
      }
    }
  ]
}
```

### Testing Strategy

**Unit Tests**:
- Test individual Lambda functions
- Mock DynamoDB and Cognito calls
- Verify business logic

**Integration Tests**:
- Test API endpoints end-to-end
- Verify data flow through services
- Test authentication and authorization

**Load Tests**:
- Use Artillery or k6 for load testing
- Test concurrent user scenarios
- Identify performance bottlenecks

**Example Load Test**:
```yaml
# load-test.yml
config:
  target: 'https://your-api-id.execute-api.ap-southeast-1.amazonaws.com/dev'
  phases:
    - duration: 60
      arrivalRate: 10
scenarios:
  - name: "Login and fetch employees"
    flow:
      - post:
          url: "/auth/login"
          json:
            email: "test@example.com"
            password: "Test123!"
      - get:
          url: "/employees"
```

### Best Practices

✅ **Structured Logging**: Use JSON format for logs
✅ **Correlation IDs**: Track requests across services
✅ **Error Tracking**: Log errors with context
✅ **Performance Monitoring**: Track response times
✅ **Automated Testing**: Run canaries regularly
✅ **Alerting**: Set up alarms for critical metrics
✅ **Cost Monitoring**: Track AWS costs

### Troubleshooting

**High Error Rates**:
- Check CloudWatch Logs for error details
- Verify IAM permissions
- Check DynamoDB capacity
- Review recent code changes

**High Latency**:
- Optimize DynamoDB queries
- Add caching where appropriate
- Review Lambda memory allocation
- Check cold start times

**Failed Canaries**:
- Review canary logs
- Check application availability
- Verify test credentials
- Update canary scripts if UI changed

### Monitoring Checklist

- [ ] CloudWatch Logs configured for all Lambda functions
- [ ] Key metrics tracked (invocations, errors, duration)
- [ ] Alarms set up for critical thresholds
- [ ] Synthetics canaries deployed and running
- [ ] Dashboard created for visualization
- [ ] Log retention policies configured
- [ ] Cost alerts configured
- [ ] On-call rotation established (for production)

### Next Steps

With monitoring in place, proceed to [Cleanup](../5.11-cleanup/) when you're done with the workshop to avoid ongoing charges.
