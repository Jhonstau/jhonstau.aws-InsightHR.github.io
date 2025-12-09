---
title: "Blog 2"
date: ""
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Tùy chỉnh môi trường dự án Amazon SageMaker Unified Studio theo nhu cầu của bạn bằng cách sử dụng custom blueprints

Bởi Aditya Challa và Khushbu Agarwal | vào ngày 17 tháng 9 năm 2025 |
tại Amazon SageMaker Unified Studio, Announcements, Intermediate (200),
Technical How-to

Amazon SageMaker Unified Studio là một môi trường phát triển dữ liệu và
AI, kết hợp giữa data preparation, analytics, machine learning (ML) và
generative AI trong cùng một nền tảng. Bằng cách hợp nhất các quy trình
làm việc này, SageMaker Unified Studio giúp đội ngũ tránh việc phải quản
lý nhiều công cụ riêng lẻ, đồng thời giúp data scientists, analysts và
developers có thể xây dựng, huấn luyện và triển khai các mô hình ML và
ứng dụng AI dễ dàng hơn.

Trong SageMaker Unified Studio, một project được xem là ranh giới hợp
tác giữa nhiều người dùng cùng làm việc trên một business use case. Một
blueprint định nghĩa các công cụ và dịch vụ AWS mà các thành viên trong
dự án có thể sử dụng khi làm việc với dữ liệu của họ. Blueprints được
xác định bởi administrator và được vận hành bằng AWS CloudFormation.
Thay vì phải tự cấu hình thủ công cấu trúc dự án hoặc quy trình làm
việc, các nhóm có thể triển khai nhanh chóng các môi trường phân tích và
AI an toàn, theo một cách tuân thủ và đồng nhất. Cách tiếp cận này giúp
rút ngắn đáng kể thời gian thiết lập và cung cấp workspace tiêu chuẩn
hóa trên toàn tổ chức. Ngay khi khởi tạo, SageMaker Unified Studio đã
cung cấp nhiều blueprint mặc định (default blueprints).

Gần đây, AWS đã ra mắt tính năng custom blueprints trong SageMaker
Unified Studio. Các tổ chức giờ đây có thể tích hợp các dependency
riêng, các security controls sử dụng IAM policies được quản lý nội bộ,
và các best practices riêng, giúp họ dễ dàng tuân thủ theo quy định của
tổ chức. Vì được định nghĩa thông qua infrastructure as code (IaC), các
blueprint có thể dễ dàng version control, chia sẻ giữa các nhóm, và phát
triển theo thời gian. Tính năng này giúp tăng tốc quá trình onboarding,
đảm bảo tính nhất quán và kiểm soát trong các dự án dữ liệu quy mô lớn.

Đối với các doanh nghiệp, điều này đồng nghĩa việc dành nhiều thời gian
hơn cho việc quan sát, model và sự phát triển, thay vì phải xử lý hạ
tầng. Tính năng custom blueprints được thiết kế để giúp đội nhóm làm
việc nhanh hơn, nhất quán hơn, đồng thời duy trì các kiểm soát bảo mật
và best practices của tổ chức. Trong bài viết này, chúng tôi sẽ hướng
dẫn cách bắt đầu với custom blueprints trong SageMaker Unified Studio.

## Tổng quan giải pháp

Chúng tôi cung cấp một CloudFormation template để triển khai custom
blueprint trong SageMaker Unified Studio. Template này sẽ tạo các tài
nguyên sau trong môi trường dự án: 
- AWS Glue database 
- Amazon Redshift
Serverless namespace và workgroup 
- AWS Lake Formation permissions cho
phép dự án mới truy cập cơ sở dữ liệu Glue 
- Custom managed IAM policies
cho AWS Glue và Amazon Redshift

## Yêu cầu

Bài viết giả định rằng bạn đã có sẵn một domain của SageMaker Unified
Studio. Nếu chưa, vui lòng tham khảo hướng dẫn Create a Amazon SageMaker
Unified Studio domain – quick setup để tạo domain.

## Định nghĩa các tham số reserved environment

CloudFormation template sử dụng các tham số được SageMaker đặt trước
(reserved parameters) như: datazoneEnvironmentEnvironmentId,
datazoneEnvironmentProjectId, s3BucketArn, privateSubnets.

Ví dụ: định nghĩa reserved environment parameters

    "Parameters": {
        "datazoneEnvironmentEnvironmentId": {
            "Type": "String",
            "Description": "EnvironmentId for which the resource will be created for."
        },
        "datazoneEnvironmentProjectId": {
            "Type": "String",
            "Description": "DZ projectId for which project the resource will be created for."
        },
        "s3BucketArn": {
            "Type": "String",
            "Description": "Project S3 Bucket ARN"
        },
        "privateSubnets": {
            "Type": "String",
            "Description": "Project Private Subnets"
        }
    }

Ví dụ: sử dụng reserved parameters để import các giá trị cần thiết khác

    "SecurityGroupIds": [
        {
            "Fn::ImportValue": {
                "Fn::Join": [
                    "",
                    [
                        "securityGroup-",
                        { "Ref": "datazoneEnvironmentProjectId" },
                        "-dev"
                    ]
                ]
            }
        }
    ]

## Gắn custom IAM policies vào project role

Mặc định, SageMaker Unified Studio sẽ tạo một project role và đính kèm
nhiều managed policies vào role đó. Các policy mặc định này được định
nghĩa trong tooling blueprint. Với custom blueprints, bạn có thể thêm
IAM policies tùy chỉnh của riêng bạn (ngoài các policy mặc định) vào
project role. Để làm điều này, chỉ cần bao gồm IAM policies trong
CloudFormation template và sử dụng tính năng Export trong phần Outputs,
như trong ví dụ dưới đây.

    "GlueAccessManagedPolicy": {
        "Description": "ARN of the created managed policy",
        "Value": { "Ref": "GlueAccessManagedPolicy" },
        "Export": {
            "Name": {
                "Fn::Sub": "datazone-managed-policy-glue-${glueDbName}-${datazoneEnvironmentEnvironmentId}"
            }
        }
    },
    "RedshiftAccessManagedPolicy": {
        "Description": "ARN of the created Redshift managed policy",
        "Value": { "Ref": "RedshiftAccessManagedPolicy" },
        "Export": {
            "Name": {
                "Fn::Sub": "datazone-managed-policy-redshift-${redshiftWorkgroupName}-${datazoneEnvironmentEnvironmentId}"
            }
        }
    }

## Tạo custom blueprint

Thực hiện các bước sau để tạo custom blueprint bằng CloudFormation
template: 
1. Truy cập Amazon SageMaker console và mở domain nơi bạn muốn tạo custom blueprint.
2. Trên tab Blueprints, chọn Create.
3. Trong phần Name and description, nhập tên và mô tả (tùy chọn).
4. Trong phần Upload CloudFormation template, chọn Upload a template file và tải lên template đã cung cấp.
5. Chọn Next.
SageMaker sẽ tự động nhận diện các reserved parameters được định nghĩa trong template, như hình minh họa.
![Blog](/images/3-Blog/blog_2_0.png)
6. Với các Editable parameters, bạn có thể chỉnh giá trị trong cột Value và chỉ định liệu giá trị đó có thể chỉnh sửa khi tạo project hay không.
7. Chọn Next. 
Các reserved parameters đã được mô tả trước đó sẽ không hiển thị lại ở bước này
![Blog](/images/3-Blog/blog_2_1.png)
8. Chọn Enable blueprint.
9. Chọn provisioning role mà SageMaker sẽ sử dụng để cấp phát tài nguyên môi trường.
10. Chọn domain units được phép sử dụng blueprint này.
11. Chọn Next.
![Blog](/images/3-Blog/blog_2_2.png)
12. Kiểm tra thông tin blueprint và chọn Create blueprint.

## Tạo project profile
Thực hiện các bước sau để tạo custom project profile bao gồm custom blueprint vừa tạo:
1. Mở SageMaker console, truy cập vào domain của bạn.
2. Trên tab Project profiles, chọn Create.
![Blog](/images/3-Blog/blog_2_3.png)
3. Nhập tên và mô tả (tùy chọn) cho project profile.
4. Chọn Custom create.
5. Chọn các blueprints sẽ được bao gồm — trong đó có custom blueprint vừa tạo.
![Blog](/images/3-Blog/blog_2_4.png)
6. Chọn account và AWS Region sẽ được sử dụng.
7. Chọn authorized users.
8. Chọn Enable project profile on creation.
9. Chọn Create project profile.
## Tạo project
Thực hiện các bước sau để tạo project mới dựa trên custom project profile và blueprint vừa tạo:
1. Trong môi trường SageMaker Unified Studio, chọn Create project.
2. Nhập tên và mô tả (tùy chọn).
3. Trong mục Project profile, chọn profile đã tạo ở bước trước.
4. Chọn Continue.
![Blog](/images/3-Blog/blog_2_5.png)
5. Ở trang Customize blueprint parameters, xem lại và điều chỉnh các tham số nếu cần, chọn Continue.
![Blog](/images/3-Blog/blog_2_6.png)
6. Kiểm tra thông tin và chọn Create project.
![Blog](/images/3-Blog/blog_2_7.png)
SageMaker Unified Studio sẽ tạo môi trường dự án với các tài nguyên được định nghĩa trong custom blueprint của bạn.
![Blog](/images/3-Blog/blog_2_8.png)
Nó cũng sẽ gắn các IAM policies tùy chỉnh đã được định nghĩa và thêm chúng vào project role, như minh họa trong giao diện console.
![Blog](/images/3-Blog/blog_2_9.png)

## Dọn Dẹp Tài Nguyên

Để tránh phát sinh chi phí, hãy thực hiện:
1. Xóa project bạn đã tạo trong SageMaker Unified Studio.
2. Xóa custom project profile và custom blueprint.
3. Xóa CloudFormation template được sử dụng.


## Kết luận

Custom blueprints trong SageMaker Unified Studio giúp chuẩn hóa và tự
động hóa quy trình dữ liệu, phân tích và AI. Chúng giúp: - Rút ngắn thời
gian khởi tạo - Đảm bảo tuân thủ - Áp dụng IAM policies từ đầu - Version
control, chia sẻ

## Giới thiệu về tác giả

### Aditya Challa
![Blog](/images/3-Blog/blog_2_10.jpeg)
Aditya Challa là Senior Solutions Architect tại Amazon Web Services (AWS), với hơn một thập kỷ kinh nghiệm trong việc thiết kế và triển khai các giải pháp dựa trên nền tảng đám mây.
 Chuyên về data, analytics và machine learning, Aditya đã giúp nhiều doanh nghiệp chuyển đổi hạ tầng dữ liệu và xây dựng các giải pháp AI/ML có khả năng mở rộng trên AWS.
 Là một cố vấn đáng tin cậy cho khách hàng trong nhiều ngành khác nhau, Aditya luôn đam mê giúp các tổ chức điều hướng hành trình chuyển đổi số và khai mở giá trị kinh doanh thông qua đổi mới dựa trên dữ liệu. Ngoài công việc kỹ thuật, Aditya là một người yêu thích du lịch và lịch sử, tìm thấy nguồn cảm hứng trong những kỳ quan kỹ thuật của các nền văn minh. Anh duy trì tư duy phát triển (growth mindset) và tin tưởng vào học tập liên tục — triết lý này không chỉ thúc đẩy sự phát triển nghề nghiệp của anh mà còn định hướng cách anh giúp khách hàng đạt được mục tiêu công nghệ của họ thông qua AWS.

### Khushbu Agarwal
![Blog](/images/3-Blog/blog_2_11.jpeg)
Khushbu Agarwal là Senior Product Manager tại AWS.
 Cô tập trung vào việc nâng cao trải nghiệm khách hàng (customer onboarding) và mở rộng năng lực nền tảng trong Amazon SageMaker Unified Studio, nhằm biến SageMaker trở thành lựa chọn hàng đầu cho các dịch vụ analytics, generative AI, và machine learning trên AWS.
