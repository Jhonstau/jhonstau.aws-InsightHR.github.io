---
title: "AWS Security Specialty Workshop"
date: 2025-11-29
weight: 6
chapter: false
pre: " <b> 4.6. </b> "
---

### Mục tiêu Sự kiện

-   Phát triển sự hiểu biết thấu đáo về **Trụ cột Bảo mật** trong **AWS Well-Architected Framework**.
-   Nắm vững năm lĩnh vực bảo mật: **Identity**, **Detection**, **Infrastructure Protection**, **Data Protection**, và **Incident Response**.
-   Nhận bản cập nhật về các mối đe dọa bảo mật phổ biến nhất đang đối mặt với thị trường cloud địa phương.
-   Tham gia các bài tập thực hành tập trung vào việc kiểm tra quyền truy cập (**IAM**) và tạo **Incident Response (IR) Playbooks** có cấu trúc.

### Diễn giả

-   **Đội ngũ Chuyên gia Bảo mật AWS**

### Điểm nhấn Chính

#### Nền tảng & Identity (Trụ cột 1)
-   **Các Nguyên tắc Bảo mật Cốt lõi**: Tái khẳng định tầm quan trọng nghiêm trọng của việc áp dụng các phương pháp **Least Privilege**, **Zero Trust**, và **Defense in Depth**.
-   **Quản lý Danh tính Hiện đại**: Nhấn mạnh sự chuyển đổi từ việc sử dụng thông tin xác thực dài hạn (**IAM Users**) sang sử dụng **IAM Roles** và tập trung hóa danh tính thông qua **AWS Identity Center (SSO)**.
-   **Quản lý Quyền**: Học cách sử dụng **Service Control Policies (SCPs)** và **Permission Boundaries** để xác định và hạn chế phạm vi quyền tối đa trong các môi trường đa tài khoản, phức tạp.
-   **Xác thực Thực hành**: Tiến hành một bài tập ngắn về **Validating IAM Policies** để chủ động xác định và giải quyết các lỗi bảo mật tiềm ẩn.

#### Detection & Infrastructure (Trụ cột 2 & 3)
-   **Phát hiện Mối đe dọa Liên tục**: Tập trung vào việc kích hoạt các dịch vụ chính như **CloudTrail** (để kiểm tra cấp độ tổ chức), **GuardDuty** (để phát hiện mối đe dọa thông minh), và **Security Hub** (để giám sát tuân thủ hợp nhất).
-   **Ghi nhật ký Toàn diện**: Thảo luận về các phương pháp hay nhất để thu thập nhật ký ở mọi lớp, bao gồm **VPC Flow Logs** (lưu lượng mạng), **ALB logs** (cân bằng tải ứng dụng), và **S3 access logs** (lưu trữ dữ liệu).
-   **Bảo vệ Vành đai Mạng**: Các chiến lược để **phân đoạn VPC** kết hợp với **Security Groups** và **NACLs**. Bảo vệ cạnh tích hợp sử dụng **WAF**, **Shield**, và **Network Firewall** cũng được đề cập.

#### Data Protection & Incident Response (Trụ cột 4 & 5)
-   **Chiến lược Mã hóa**: Chi tiết các bước cần thiết để mã hóa dữ liệu cả **in-transit và at-rest** trên các dịch vụ cốt lõi (**S3**, **EBS**, **RDS**) bằng cách sử dụng **KMS (Key Management Service)**.
-   **Vệ sinh Thông tin Xác thực**: Nhấn mạnh nhu cầu loại bỏ thông tin xác thực được mã hóa cứng bằng cách tận dụng **Secrets Manager** và **Parameter Store**, bao gồm các cơ chế xoay vòng tự động.
-   **IR Tự động hóa**: Quy trình xây dựng **IR Playbooks** có cấu trúc cho các mối đe dọa phổ biến (ví dụ: thông tin xác thực bị xâm phạm, **malware**) và tự động hóa các bước phản ứng và cô lập bằng cách sử dụng các dịch vụ như **Lambda** và **Step Functions**.

### Những Điểm Rút ra Chính

#### Mô hình Zero Trust
-   **Identity là Vành đai**: Trong kiến trúc **cloud-native**, ranh giới bảo mật chuyển từ cạnh mạng sang **Identity**, làm cho nó trở thành lớp phòng thủ quan trọng nhất.
-   Nguyên tắc rõ ràng: không bao giờ tin tưởng bất kỳ người dùng hoặc tài nguyên nào theo mặc định; luôn xác minh tất cả các yêu cầu truy cập.

#### Tự động hóa cho Quy mô
-   Các thực hành bảo mật thủ công không bền vững ở quy mô và tốc độ cloud. Việc triển khai **Detection-as-Code** và **Auto-remediation** là bắt buộc để giảm thiểu khoảng thời gian tiếp xúc với các mối đe dọa.

### Áp dụng vào Công việc

-   **Kiểm tra Chính sách IAM**: Bắt đầu kiểm tra toàn bộ tất cả **IAM Users** và quyền của họ, loại bỏ các khóa cũ và chuyển sang **IAM Roles** dựa trên ứng dụng.
-   **Kích hoạt Threat Intelligence**: Kích hoạt **GuardDuty** trên toàn cầu trên tất cả các tài khoản/region của tổ chức để hưởng lợi từ việc phát hiện mối đe dọa và sự bất thường liên tục.
-   **Áp dụng Secrets Management**: Tái cấu trúc các ứng dụng để truy xuất thông tin xác thực cơ sở dữ liệu từ **Secrets Manager** thay vì lưu trữ chúng trong các tệp cấu hình.
-   **Sẵn sàng IR**: Bắt đầu phát triển một **Incident Response Playbook** chính thức cụ thể cho kịch bản "**Compromised IAM Key**", bao gồm một kế hoạch diễn tập phản ứng tự động.

### Trải nghiệm Sự kiện

Buổi hội thảo này là một tìm hiểu sâu về kỹ thuật trong bảo mật cloud, cung cấp kiến thức toàn diện cần thiết của một Cloud Engineer hiện đại.

#### Học tập Có cấu trúc
-   Việc tổ chức chương trình giảng dạy xoay quanh **5 Trụ cột** đã chuyển đổi thành công các khái niệm bảo mật rải rác thành một khuôn khổ duy nhất, gắn kết và tiêu chuẩn.
-   Thông tin chi tiết được cung cấp về **Các mối đe dọa Hàng đầu** cụ thể cho bối cảnh thị trường địa phương là ngay lập tức thực tế cho việc giảm thiểu rủi ro.

#### Phát triển Kỹ năng Thực hành
-   Các bản demo, đặc biệt là những bản sử dụng **IAM Policy Validator** và **Access Analyzer**, đã cung cấp các kỹ năng hữu ích ngay lập tức cho việc gỡ lỗi chính sách và tăng cường bảo mật hàng ngày.
-   Việc tập trung vào **tự động hóa Incident Response** đã nhấn mạnh sự chuyển đổi từ giám sát đơn thuần sang phòng thủ hệ thống tự động, chủ động.

#### Một số hình ảnh sự kiện

![AWS DevOps & Modern Operations](/images/4-Event/blog_6_0.jpg)
![AWS DevOps & Modern Operations](/images/4-Event/blog_6_1.jpg)
![AWS DevOps & Modern Operations](/images/4-Event/blog_6_2.jpg)