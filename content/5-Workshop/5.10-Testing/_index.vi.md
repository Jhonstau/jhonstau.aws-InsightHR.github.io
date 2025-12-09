---
title: "Kiểm tra & Giám sát"
date: "2025-12-09"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Kiểm tra & Giám sát

Phần này bao gồm thiết lập kiểm tra và giám sát toàn diện cho nền tảng InsightHR.

### Tổng quan

Chúng ta sẽ triển khai:

1. **CloudWatch Logs** - Logging tập trung
2. **CloudWatch Metrics** - Giám sát hiệu suất
3. **CloudWatch Alarms** - Cảnh báo tự động
4. **CloudWatch Synthetics** - Kiểm tra tự động
5. **X-Ray Tracing** - Distributed tracing (tùy chọn)

### CloudWatch Logs

Tất cả Lambda functions tự động ghi log vào CloudWatch Logs.

**Xem Logs**:
```bash
# Liệt kê log groups
aws logs describe-log-groups \
    --log-group-name-prefix "/aws/lambda/insighthr" \
    --region ap-southeast-1

# Tail logs cho một function
aws logs tail /aws/lambda/insighthr-auth-login-handler \
    --follow \
    --region ap-southeast-1
```

**Truy vấn Log Insights**:

Tìm lỗi:
```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

Phân tích thời gian phản hồi:
```sql
fields @timestamp, @duration
| stats avg(@duration), max(@duration), min(@duration)
```

### CloudWatch Metrics

Giám sát các metrics chính:

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

**Xem Metrics**:
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

Tạo alarms cho các metrics quan trọng:

**High Error Rate Alarm**:
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name insighthr-high-error-rate \
    --alarm-description "Cảnh báo khi tỷ lệ lỗi Lambda vượt quá 5%" \
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
    --alarm-description "Cảnh báo khi độ trễ API vượt quá 2 giây" \
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

Kiểm tra tự động với synthetic canaries:

**1. Login Canary** - Kiểm tra luồng đăng nhập
**2. Dashboard Canary** - Kiểm tra loading dashboard
**3. Chatbot Canary** - Kiểm tra truy vấn chatbot
**4. Autoscoring Canary** - Kiểm tra tính toán hiệu suất

**Tạo Login Canary**:

```javascript
// login-canary.js
const synthetics = require('Synthetics');
const log = require('SyntheticsLogger');

const loginFlow = async function () {
    const page = await synthetics.getPage();
    
    // Điều hướng đến trang đăng nhập
    await page.goto('https://insight-hr.io.vn/login', {
        waitUntil: 'domcontentloaded',
        timeout: 30000
    });
    
    // Điền form đăng nhập
    await page.type('#email', 'test@example.com');
    await page.type('#password', 'Test123!');
    
    // Click nút đăng nhập
    await Promise.all([
        page.waitForNavigation(),
        page.click('button[type="submit"]')
    ]);
    
    // Xác minh đăng nhập thành công
    await page.waitForSelector('.dashboard', { timeout: 10000 });
    
    log.info('Luồng đăng nhập hoàn tất thành công');
};

exports.handler = async () => {
    return await synthetics.executeStep('LoginFlow', loginFlow);
};
```

**Triển khai Canary**:
```bash
# Tạo S3 bucket cho canary artifacts
aws s3 mb s3://insighthr-canary-artifacts --region ap-southeast-1

# Tạo canary
aws synthetics create-canary \
    --name insighthr-login-canary \
    --artifact-s3-location s3://insighthr-canary-artifacts \
    --execution-role-arn arn:aws:iam::${ACCOUNT_ID}:role/CloudWatchSyntheticsRole \
    --schedule Expression="rate(5 minutes)" \
    --runtime-version syn-nodejs-puppeteer-6.2 \
    --code file://login-canary.zip \
    --region ap-southeast-1
```

### Thiết lập Dashboard

Tạo CloudWatch Dashboard:

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

### Chiến lược Kiểm tra

**Unit Tests**:
- Kiểm tra các Lambda functions riêng lẻ
- Mock các lời gọi DynamoDB và Cognito
- Xác minh logic nghiệp vụ

**Integration Tests**:
- Kiểm tra API endpoints end-to-end
- Xác minh luồng dữ liệu qua các dịch vụ
- Kiểm tra xác thực và phân quyền

**Load Tests**:
- Sử dụng Artillery hoặc k6 cho load testing
- Kiểm tra các kịch bản người dùng đồng thời
- Xác định các điểm nghẽn hiệu suất

**Ví dụ Load Test**:
```yaml
# load-test.yml
config:
  target: 'https://your-api-id.execute-api.ap-southeast-1.amazonaws.com/dev'
  phases:
    - duration: 60
      arrivalRate: 10
scenarios:
  - name: "Đăng nhập và lấy nhân viên"
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

✅ **Structured Logging**: Sử dụng định dạng JSON cho logs
✅ **Correlation IDs**: Theo dõi requests qua các dịch vụ
✅ **Error Tracking**: Ghi log lỗi với ngữ cảnh
✅ **Performance Monitoring**: Theo dõi thời gian phản hồi
✅ **Automated Testing**: Chạy canaries thường xuyên
✅ **Alerting**: Thiết lập alarms cho các metrics quan trọng
✅ **Cost Monitoring**: Theo dõi chi phí AWS

### Xử lý Sự cố

**Tỷ lệ Lỗi Cao**:
- Kiểm tra CloudWatch Logs để biết chi tiết lỗi
- Xác minh quyền IAM
- Kiểm tra dung lượng DynamoDB
- Xem lại các thay đổi code gần đây

**Độ trễ Cao**:
- Tối ưu truy vấn DynamoDB
- Thêm caching khi phù hợp
- Xem lại phân bổ memory Lambda
- Kiểm tra thời gian cold start

**Canaries Thất bại**:
- Xem lại canary logs
- Kiểm tra tính khả dụng của ứng dụng
- Xác minh thông tin đăng nhập kiểm tra
- Cập nhật canary scripts nếu UI thay đổi

### Checklist Giám sát

- [ ] CloudWatch Logs đã được cấu hình cho tất cả Lambda functions
- [ ] Các metrics chính được theo dõi (invocations, errors, duration)
- [ ] Alarms đã được thiết lập cho các ngưỡng quan trọng
- [ ] Synthetics canaries đã được triển khai và chạy
- [ ] Dashboard đã được tạo để trực quan hóa
- [ ] Chính sách lưu giữ log đã được cấu hình
- [ ] Cảnh báo chi phí đã được cấu hình
- [ ] Luân phiên on-call đã được thiết lập (cho production)

### Bước Tiếp theo

Với giám sát đã được thiết lập, tiến hành [Dọn dẹp](../5.11-cleanup/) khi bạn hoàn thành workshop để tránh các khoản phí liên tục.
