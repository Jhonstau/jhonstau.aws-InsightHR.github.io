---
title: "AWS DevOps & Modern Operations"
date: 2025-11-17
weight: 5
chapter: false
pre: " <b> 4.5. </b> "

---

### Mục tiêu Sự kiện

-   Thấm nhuần **văn hóa DevOps** toàn diện và nắm vững các chỉ số hiệu suất chính (**DORA metrics**).
-   Xây dựng một **Continuous Integration/Continuous Delivery (CI/CD)** pipeline chức năng bằng cách sử dụng các **AWS Developer Tools** gốc.
-   Hiện đại hóa quản lý cơ sở hạ tầng bằng cách sử dụng các công cụ **Infrastructure as Code (IaC)** như **CloudFormation** và **CDK**.
-   Khám phá việc triển khai các ứng dụng container hóa (**Docker**) trên nhiều dịch vụ tính toán **AWS** (**ECS**, **EKS**, **App Runner**).
-   Thiết lập các phương pháp hay nhất cho một hệ thống **Observability** hoàn chỉnh cho các ứng dụng phân tán, phức tạp.

### Diễn giả

-   **Đội ngũ Chuyên gia AWS**

### Điểm nhấn Chính

#### Văn hóa DevOps và Tự động hóa
-   **Chỉ số Hiệu quả**: Xác định và giải thích sự liên quan của **DORA Metrics** (**Lead Time**, **Deployment Frequency**, **MTTR**, **Change Failure Rate**) để đo lường sức khỏe hoạt động.
-   **Xây dựng Pipeline**: Trình diễn việc tạo ra một quy trình **CI/CD** hoàn chỉnh, tích hợp **CodeCommit**, **CodeBuild**, **CodeDeploy**, và điều phối chúng bằng **CodePipeline**.
-   **Triển khai An toàn**: Xem xét các kỹ thuật triển khai quan trọng để giảm thiểu rủi ro, bao gồm **Blue/Green**, **Canary**, và **Rolling updates**. 

#### Làm chủ Infrastructure as Code (IaC)
-   **Nguyên tắc cơ bản của CloudFormation**: Đề cập đến việc xác định các tài nguyên cơ sở hạ tầng bằng các **template** khai báo và tận dụng các tính năng như **Drift Detection**.
-   **Lợi thế của AWS CDK**: Trình diễn việc sử dụng các ngôn ngữ lập trình quen thuộc (**TypeScript**, **Python**, v.v.) để xác định cơ sở hạ tầng thông qua các "**Constructs**" mạnh mẽ, có thể tái sử dụng.
-   **Chiến lược IaC**: Thảo luận về tiêu chí để lựa chọn công cụ **IaC** thích hợp (ví dụ: **CloudFormation** so với **CDK** so với **Terraform**) dựa trên độ phức tạp của dự án và trình độ thành thạo của nhóm.

#### Containerization và Các Tùy chọn Tính toán
-   **Phổ Triển khai**: Xem xét các tùy chọn **container AWS** khác nhau, từ quản lý image (**ECR**) đến các lựa chọn điều phối như **ECS** được đơn giản hóa, **EKS** được tiêu chuẩn hóa (**Kubernetes**) và **App Runner** được trừu tượng hóa cao.

#### Giám sát và Observability
-   **Khả năng hiển thị Toàn bộ Stack**: Nhấn mạnh việc chuyển từ giám sát cơ bản sang **Observability** hoàn chỉnh, tích hợp **CloudWatch** (**Logs**, **Metrics**) với **X-Ray** (**Distributed Tracing**) để có được thông tin chi tiết sâu hơn về hiệu suất **microservice**.
-   **Các Phương pháp hay nhất về Hoạt động**: Hướng dẫn về việc tạo **dashboard** giám sát hiệu quả và thiết lập các quy trình **on-call** đáng tin cậy.

### Những Điểm Rút ra Chính

#### Tự động hóa Bắt buộc
-   **Nền tảng CI/CD**: Việc áp dụng **CI/CD** là một yêu cầu bắt buộc về mặt hoạt động, giúp giảm lỗi thủ công và tăng đáng kể tốc độ phát hành.
-   **Tiêu chuẩn IaC**: **Infrastructure as Code** là bắt buộc để đảm bảo tính nhất quán và khả năng lặp lại của cấu hình trên tất cả các môi trường phát triển, kiểm thử và sản xuất.

#### Nguyên tắc Vận hành Xuất sắc
-   **Vượt ra ngoài Giám sát**: **Observability** là điều cần thiết để chẩn đoán lỗi và hiểu hành vi phức tạp trong các kiến trúc **microservice** hiện đại.
-   **Chiến lược Zero-Downtime**: Nắm vững các kỹ thuật triển khai nâng cao, chẳng hạn như **Blue/Green**, là chìa khóa để đạt được các bản phát hành liền mạch, không thời gian chết.

### Áp dụng vào Công việc

-   **Tự động hóa Quy trình Thủ công**: Chuyển đổi các bước xây dựng và triển khai thủ công hiện có để sử dụng **AWS CodePipeline**, kết hợp các giai đoạn kiểm thử tự động.
-   **Áp dụng CDK cho Dự án Mới**: Bắt đầu xác định cơ sở hạ tầng cho các dịch vụ mới bằng **AWS CDK** để đảm bảo các tài nguyên được cấp phát theo cách lập trình và có thể lặp lại.
-   **Containerize & Thử nghiệm**: **Dockerize** các thành phần ứng dụng phù hợp và thử nghiệm triển khai chúng trên các nền tảng đơn giản như **AWS App Runner**.
-   **Triển khai Tracing**: Tích hợp **AWS X-Ray** vào các ứng dụng phân tán để có được khả năng hiển thị quan trọng về độ trễ và giao tiếp giữa các **microservice**.

### Trải nghiệm Sự kiện

Sự kiện kéo dài cả ngày này đã cung cấp một cái nhìn có cấu trúc và có tính liên kết cao về bối cảnh vận hành hiện đại.

#### Học tập Tích hợp
-   Bản trình diễn trực tiếp về "**Full CI/CD pipeline**" đã hình dung hiệu quả hành trình của mã từ kho lưu trữ của nhà phát triển vào môi trường sản xuất.
-   Việc làm rõ các trường hợp sử dụng và đánh đổi khác biệt giữa **ECS** và **EKS** là vô cùng có giá trị cho thiết kế kiến trúc giải pháp trong tương lai.

#### Những Điểm Rút ra Thực tế
-   Việc tập trung vào các **Chiến lược Triển khai** khác nhau (**Canary deployments**, **feature flags**) đã cung cấp các giải pháp thực tế, tức thì để giảm thiểu rủi ro triển khai trong nhóm.
-   Phiên này đã củng cố mối liên hệ cần thiết giữa **Code, Infrastructure, và Monitoring** để đạt được hiệu quả hoạt động thực sự.

#### Một số hình ảnh sự kiện

![AWS DevOps & Modern Operations](/images/4-Event/blog_5_0.jpg)
![AWS DevOps & Modern Operations](/images/4-Event/blog_5_1.jpg)
![AWS DevOps & Modern Operations](/images/4-Event/blog_5_2.jpg)