---
title: "Worklog Tuần 9"
date: ""
weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---

## Mục tiêu Tuần 9:

* Thiết lập cơ sở hạ tầng cốt lõi của dự án bao gồm cơ sở dữ liệu và giám sát.

***

## Các công việc cần triển khai trong tuần này:
| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| :--- | :--- | :--- | :--- | :--- |
| Hai | - Cấu hình **Thiết lập DynamoDB Cơ bản** (các bảng và dung lượng đọc/ghi ban đầu) <br> - Triển khai **Thiết lập Cloudwatch** ban đầu để giám sát hoạt động **EC2**/**Lambda** | 11/03/2025 | 11/03/2025      | Tài liệu **AWS**: <https://docs.aws.amazon.com/> |
| Ba | - Thiết lập **Cloudwatch Canaries** để giám sát tổng hợp các điểm cuối của ứng dụng <br> - Cấu hình các topics và subscriptions của **SNS** (Simple Notification Service) cho cảnh báo | 11/04/2025 | 11/04/2025      | |
| Tư | Nghỉ. | 11/05/2025 | 11/05/2025      | |
| Năm | - Cập nhật **Cấu hình Cloudwatch** để phù hợp với các tính năng bổ sung và yêu cầu metrics mới của dự án | 11/06/2025 | 11/06/2025      |Tài liệu **AWS**: <https://docs.aws.amazon.com/> |
| Sáu | - Thực hiện **Tái cấu hình DynamoDB** để tối ưu hóa key schema cho các mẫu truy vấn thường xuyên nhất | 11/07/2025 | 11/07/2025 | Tài liệu **AWS**: <https://docs.aws.amazon.com/> |
| Bảy | Nghỉ. | 11/08/2025 | 11/08/2025 |  |
***

## Kết quả đạt được Tuần 9:

* Cơ sở hạ tầng cơ bản cho **dữ liệu lâu dài** và **giám sát** đã được thiết lập bằng **DynamoDB** và **CloudWatch**.
* Thiết lập giám sát nâng cao với **CloudWatch Canaries** và thông báo bằng **SNS**.

***
***