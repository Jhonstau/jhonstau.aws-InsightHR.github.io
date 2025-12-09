---
title: "Workshop: Data Science on AWS"
date: 2025-10-16
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

### Mục tiêu Sự kiện

-   Cung cấp cái nhìn tổng quan kỹ lưỡng về vòng đời của một giải pháp **Data Science** hiện đại, từ khi hình thành ý tưởng đến triển khai.
-   Chi tiết cấu trúc của **AWS Data Science Pipeline**, bao gồm các giai đoạn lưu trữ dữ liệu, chuyển đổi và triển khai mô hình.
-   Tạo điều kiện học tập thực hành bằng cách sử dụng tập dữ liệu thực (**IMDb**) để xây dựng một ứng dụng (**Sentiment Analysis**).
-   So sánh các lợi thế và bất lợi về hoạt động và tài chính của cơ sở hạ tầng **Cloud-based** so với **On-premise**.

### Diễn giả

-   **Ông Văn Hoàng Kha** – Cloud Solutions Architect, **AWS Community Builder**
-   **Ông Bạch Đoàn Vương** – Cloud **DevOps** Engineer, **AWS Community Builder**

### Điểm nhấn Chính

#### 1. Cloud trong Data Science & Tổng quan Pipeline
-   **Sự cần thiết của Cloud**: Khám phá lý do tại sao **data science** hiện đại cần đến cloud: yêu cầu sự nhanh nhẹn, khả năng mở rộng và tích hợp liền mạch, những điều mà các thiết lập **on-premise** tĩnh không thể cung cấp.
-   **Các Thành phần Cốt lõi của Pipeline**: 

[Image of AWS Data Science Pipeline Architecture]

    -   **Data Lake**: Nhấn mạnh **Amazon S3** là lớp lưu trữ thiết yếu, có khả năng mở rộng.
    -   **Serverless ETL**: Giới thiệu **AWS Glue** để chuyển đổi và làm sạch dữ liệu tự động, **serverless**.
    -   **Môi trường ML**: Tập trung vào **Amazon SageMaker** như nền tảng thống nhất cho toàn bộ vòng đời mô hình học máy. Ngăn xếp **AWS AI/ML** rộng hơn, bao gồm các dịch vụ **AI** chuyên biệt, cũng được xem xét.

#### 2. Demo Thực hành
-   **Demo 1: Chuẩn bị Dữ liệu**: Minh họa quy trình làm sạch và chuyển đổi tập dữ liệu **IMDb** bằng **AWS Glue**. Các kỹ thuật **feature engineering** được trình diễn, đối chiếu các tùy chọn **low-code** như **SageMaker Canvas** với các phương pháp truyền thống, theo hướng mã, sử dụng các thư viện như **Numpy/Pandas**.
-   **Demo 2: Triển khai Mô hình**: Trình diễn quy trình **ML** hoàn chỉnh cho một mô hình **Sentiment Analysis**. Điều này bao gồm trình tự "Train, Tune, Deploy" trong **SageMaker Studio**. Các diễn giả cũng đề cập đến tính linh hoạt của việc triển khai các mô hình tùy chỉnh thông qua phương pháp **Bring Your Own Model (BYOM)**, tương thích với các framework phổ biến như **TensorFlow** và **PyTorch**.

#### 3. Thảo luận Chiến lược
-   **So sánh Nền tảng**: Một cuộc thảo luận tập trung vào việc tối ưu hóa cho cả hiệu suất và ngân sách. Đã nhấn mạnh rằng **tính đàn hồi của cloud** làm cho nó vượt trội hơn trong việc thử nghiệm với các khối lượng công việc nặng về tính toán so với chi phí vốn đầu tư cấm đoán của phần cứng **on-premise** cố định.
-   **Dự án Tiếp theo**: Một dự án nhỏ được đề xuất để giúp người tham dự củng cố kiến thức thực hành có được trong buổi hội thảo.

### Những Điểm Rút ra Chính

#### Quy trình Kỹ thuật
-   **Tích hợp Kỹ thuật**: Một quy trình **ML** chức năng yêu cầu **tích hợp liền mạch, tự động** giữa các dịch vụ cốt lõi: lưu trữ **data lake (S3)**, xử lý (**Glue**) và môi trường mô hình hóa (**SageMaker**).
-   **Công cụ Chiến lược**: Năng lực bao gồm việc biết khi nào sử dụng các dịch vụ **AI** được quản lý cấp cao (ví dụ: **Comprehend**, **Textract**) và khi nào phát triển các mô hình tùy chỉnh trong **SageMaker** để tối đa hóa hiệu quả và **ROI**.

#### Ứng dụng Công nghiệp
-   **Trọng tâm Hoạt động**: Thách thức chính trong việc chuyển từ các khái niệm lớp học sang thực tế doanh nghiệp là đạt được **tự động hóa và hoạt động có khả năng mở rộng**.
-   **Cân bằng Chi phí-Lợi ích**: Các dự án dữ liệu thành công đòi hỏi sự cân bằng liên tục giữa việc đạt được độ chính xác mô hình cao và quản lý chi phí vận hành liên quan.

### Áp dụng vào Công việc

-   **Hiện đại hóa ETL**: Chuyển đổi các script chuyển đổi dữ liệu và **ETL** cục bộ hiện có để tận dụng **AWS Glue** cho khả năng tự động hóa **serverless** của nó, đặc biệt để xử lý dữ liệu quy mô lớn.
-   **Tiêu chuẩn hóa Mô hình hóa**: Áp dụng **SageMaker Studio** làm môi trường chuyên nghiệp, chính để lặp lại và triển khai các mô hình thử nghiệm, từ đó tiêu chuẩn hóa vòng đời **ML**.
-   **Củng cố Học tập**: Cam kết thực hiện dự án tiếp theo được đề xuất để củng cố kiến thức đầu cuối có được từ việc xử lý và phân tích các tập dữ liệu trong thế giới thực.

### Trải nghiệm Sự kiện

Buổi hội thảo **“Data Science on AWS”** đã phục vụ hiệu quả như điểm kết nối quan trọng giữa nghiên cứu học thuật lý thuyết và thực tế triển khai trong doanh nghiệp.
-   **Ứng dụng Thực tế**: Tôi có được sự tiếp xúc trực tiếp với các công nghệ cloud cụ thể được sử dụng bởi các tổ chức công nghệ hàng đầu toàn cầu.
-   **Sự rõ ràng về Quy trình làm việc**: Việc trình diễn trực tiếp làm sạch **dữ liệu IMDb** và triển khai **mô hình Sentiment Analysis** đã làm rõ các bước triển khai thực tế để xây dựng các hệ thống **AI** dựa trên cloud.
-   **Thông tin chi tiết Chiến lược**: Tương tác của tôi với các **AWS Community Builders** đã cung cấp ngữ cảnh chiến lược có giá trị cho các lập luận kinh tế và hoạt động hỗ trợ **chuyển đổi cloud** hơn là duy trì các giải pháp **on-premise** truyền thống.

#### Một số hình ảnh sự kiện

![Workshop: Data Science on AWS](/images/4-Event/blog_3_0.jpg)
![Workshop: Data Science on AWS](/images/4-Event/blog_3_1.jpg)
![Workshop: Data Science on AWS](/images/4-Event/blog_3_2.jpg)