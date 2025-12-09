---
title: "Blog 1"
date: ""
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Triển khai cơ chế failover chi tiết trong môi trường Amazon EKS multi-Region
bới Dumlu Timuralp và Pratik R. Mankad | vào ngày 18 tháng 9 năm 2025 | tại Amazon Elastic Kubernetes Service, Amazon Route 53, Technical How-to | Permalink | Chia Sẻ

Các doanh nghiệp thuộc nhiều ngành khác nhau vận hành hàng chục triệu cụm Amazon Elastic Kubernetes Service (Amazon EKS) mỗi năm, phục vụ nhiều trường hợp sử dụng khác nhau như ứng dụng web/di động, xử lý dữ liệu, machine learning (ML), generative AI, gaming và Internet of Things (IoT). Khi các tổ chức ngày càng áp dụng mô hình nền tảng multi-tenant, nơi nhiều nhóm ứng dụng cùng chia sẻ một cụm EKS,  họ cần khả năng kiểm soát chi tiết hơn về tính sẵn sàng của ứng dụng trong các kiến trúc multi-Region, đặc biệt để phục vụ người dùng toàn cầu và đáp ứng các yêu cầu tuân thủ nghiêm ngặt.

Mặc dù mô hình multi-tenant giúp tối ưu hóa việc sử dụng tài nguyên và giảm chi phí vận hành, nhưng nó cũng đặt ra thách thức khi các ứng dụng cần mục tiêu riêng biệt về Recovery Point Objective (RPO) và Recovery Time Objective (RTO) trên nhiều vùng AWS. Các phương pháp truyền thống buộc phải áp dụng chính sách failover đồng nhất cho mọi ứng dụng, khiến chỉ cần một ứng dụng gặp sự cố là phải hy sinh giữa hiệu quả vận hành và độ bền vững riêng của từng ứng dụng.

Trong bài viết này, chúng tôi sẽ minh họa cách cấu hình Amazon Route 53 để cho phép hành vi failover riêng biệt cho từng ứng dụng trong môi trường multi-tenant Amazon EKS đa vùng. Giải pháp này giúp bạn duy trì lợi ích chi phí của hạ tầng dùng chung mà vẫn đáp ứng các yêu cầu sẵn sàng khác nhau của từng ứng dụng.

## Kiến trúc điển hình

Trước khi đi vào chi tiết giải pháp, chúng ta hãy cùng xem cách mà một kiến trúc Amazon EKS multi-Region, multi-tenant thường được cấu trúc, và các thành phần chính cho phép Regional traffic routing. Hình 1 mô tả các thành phần chính sau:
Route 53
Traffic routing đến Application Load Balancer (ALB) tại từng vùng tương ứng.
Đảm bảo tính sẵn sàng cao và khả năng phục hồi khi xảy ra sự cố.


AWS Load Balancer Controller
Cho biết các ứng dụng đang chạy trên cụm EKS thông qua ALB.
Sử dụng IP target type để đăng ký các Pod ứng dụng vào ALB.


Cấu hình Route 53:
Regional Routing:
Mỗi vùng AWS sử dụng một Route 53 alias record để route traffic đến ALB tương ứng.
Ví dụ: app1.example.com trỏ đến ALB ở Region 1
Ví dụ: app2.example.com trỏ đến ALB ở Region 2


Giám sát sức khỏe:
Thuộc tính “Evaluate target health” trong alias record:
Liên tục giám sát tình trạng của ALB
Tự động loại bỏ các ALB bất ổn định khỏi phản hồi DNS


Request routing:
Cung cấp các chính sách định tuyến cho kiến trúc active-active

![Blog](/images/3-Blog/blog_1_0.jpg)

Hình 1: Kiến trúc nền tảng Amazon EKS multi-Region, multi-tenant

Vấn đề “all-or-nothing health check”

Thuộc tính Evaluate target health trong alias record của Route 53 liên tục giám sát tình trạng của ALB được chỉ định. Một ALB chỉ được xem là ổn định nếu tất cả các target group được cấu hình đều có ít nhất một target khỏe mạnh.
 Nếu bất kỳ target group nào trở nên bất ổn định, Route 53 sẽ đánh dấu toàn bộ ALB là bất ổn định và loại nó khỏi phản hồi DNS, khiến người dùng bị chuyển hướng đến các vùng còn lại.

Hiện tượng này tạo ra hiệu ứng “lỗi dây chuyền” trong môi trường multi-tenant:
Khi một ứng dụng gặp lỗi → Route 53 đánh dấu toàn bộ ALB là bất ổn định.
Tất cả lưu lượng sẽ chuyển sang vùng AWS khác, kể cả đối với ứng dụng ổn định.
Ứng dụng đang hoạt động tốt bị ảnh hưởng bởi sự tăng cao không cần thiết của latency.


Ví dụ:
app1.example.com lỗi ở Region 1.
app2.example.com vẫn hoạt động tốt ở Region 1.
Route 53 chuyển toàn bộ lưu lượng sang Region 2.
Người dùng của app2 chịu tăng latency dù ứng dụng không lỗi.


Hành vi này buộc người dùng phải chọn giữa hai lựa chọn không tối ưu:
Chấp nhận toàn bộ vùng bị lỗi khi bất kỳ microservice nào hỏng.
Triển khai ALB riêng cho từng microservice → tăng độ phức tạp và chi phí.


Các phần tiếp theo trình bày một giải pháp khắc phục giúp đạt được mô hình triển khai đa vùng có độ bền cao hơn.

## Tổng quan giải pháp

Để khắc phục vấn đề này, chúng tôi đề xuất giải pháp cho phép kiểm soát failover chi tiết ở cấp độ ứng dụng.
Hình 2 minh họa kiến trúc của giải pháp. Giải pháp gồm hai cấu hình chính:

1. Health check riêng cho từng ứng dụng
Cấu hình Route 53 health check chuyên biệt cho từng ứng dụng ở mỗi vùng AWS.
Gán từng health check cho alias record tương ứng.
Thiết lập Evaluate Target Health = No trong alias record.


2. Kiểm soát failover chi tiết
Route 53 giám sát sức khỏe của từng ứng dụng độc lập.
Khi xảy ra lỗi, chỉ ứng dụng bị ảnh hưởng mới bị chuyển hướng lưu lượng.
Ứng dụng khỏe mạnh vẫn tiếp tục phục vụ tại vùng ban đầu.

![Blog](/images/3-Blog/blog_1_1.jpg)

Hình 2: Kiến trúc giải pháp
Lợi ích chính của kiến trúc:
Kiểm soát và giám sát nâng cao:
Giám sát chi tiết ở cấp độ ứng dụng (target group).
Loại bỏ giới hạn chỉ giám sát ở cấp ALB.


Quản lý failover thông minh:
Failover chọn lọc cho từng ứng dụng riêng biệt.
Bảo vệ ứng dụng ổn định khỏi bị ảnh hưởng bởi sự cố của ứng dụng khác.
Duy trì định tuyến tối ưu cho dịch vụ không bị lỗi.


Tổ chức có thể:
Duy trì lợi ích chi phí của hạ tầng dùng chung.
Đạt được độ bền riêng cho từng ứng dụng.
Đáp ứng yêu cầu sẵn sàng khác nhau.
Tránh định tuyến chéo vùng không cần thiết.

## Yêu cầu chuẩn bị

Bạn cần có:
Tài khoản AWS.
Một public hosted zone trong Route 53 (hoặc đăng ký miền mới / sử dụng miền hiện có).
AWS Command Line Interface (AWS CLI)
eksctl
kubectl
kubectx (tùy chọn)
AWS CloudShell (tùy chọn).

## Walkthrough

Cách bước sau đây sẽ hướng dẫn bạn về giải pháp này.

### Cấu hình biến môi trường

Thay thế các giá trị trong ví dụ bằng thông tin của bạn và tạo biến môi trường:
```bash
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export Region1=eu-west-1 
export Region2=eu-central-1
export ALBName=demoalb
export Region1ClusterName=cluster1
export Region2ClusterName=cluster2
export app1hostname=app1.example.com
export app2hostname=app2.example.com
export HostedZoneName=example.com
export HostedZoneId=$(aws route53 list-hosted-zones-by-name --dns-name $HostedZoneName --query 'HostedZones[0].Id' --output text | cut -d'/' -f3)
```

### Tạo cụm EKS tại Region 1 và Region 2

1. Tạo cụm EKS tại Region 1
```bash
eksctl create cluster --name=$Region1ClusterName --enable-auto-mode --region=$Region1
```

2. Chờ trạng thái cụm chuyển sang Active (khoảng 12 phút).
```bash
aws eks describe-cluster --name $Region1ClusterName --region $Region1 --output json --query 'cluster.status'
```

3. Lặp lại tương tự cho Region 2:
```bash
eksctl create cluster --name=$Region2ClusterName --enable-auto-mode --region=$Region2
```

4. Chờ trạng thái cụm chuyển sang Active (khoảng 12 phút).
```bash
aws eks describe-cluster --name $Region2ClusterName --region $Region2 --output json --query 'cluster.status'
```

5. Sau khi tạo xong, eksctl sẽ tự động cập nhật file kubeconfig. Sử dụng kubectl để liệt kê từng cụm. Sử dụng kubectx (hoặc tự tay đổi) để chuyển giữa hai cụm.
```bash
kubectl get nodes
```

### Tạo cấu hình IngressClass

1. Chạy câu lệnh này để tạo IngressClassParams và IngressClass tại cụm của Region 1
```bash
curl -s https://raw.githubusercontent.com/aws-samples/multi-region-ingress/refs/heads/main/ingressclassconfiguration.yaml | kubectl apply -f -
```

2. Xác nhận rằng tài nguyên đã được tạo
```bash
kubectl get ingressclass,ingressclassparams
```

3. Thực hiện tương tự trên cụm ở Region 2.

## Triển khai app1 và app2 trên hai cụm EKS
1. Chúng ta sẽ sử dụng “manifest” để triển khai ứng dụng trên cụm tại Region 1. “Manifest” sẽ tạo:
Triển khai với duy nhất 1 bản sao cho từng ứng dụng
Dịch vụ cho từng sự triển khai
“Ingress” để forward http requests lên dịch vụ	
```bash
curl -s https://github.com/aws-samples/multi-region-ingress/blob/main/multitenant.yaml | envsubst | kubectl apply -f -
```

2. Xác nhận rằng tài nguyên đã được tạo. Bạn sẽ thấy được 2 ứng dụng đã được triển khai, 2 dịch vụ, và 1 “ingress” trong output. Điều này cho biết rằng pod hiện đang chạy.
```bash
kubectl get all,ingress
```

3. Thực hiện tương tự trên cụm ở Region 2.
Ứng dụng mẫu hiển thị trang HTML với tên ứng dụng, tên Pod, AWS Region và AWS Availability Zone tương ứng. Cái này sẽ được làm mẫu ở hình 3 và hình 4. Kiểm tra kích hoạt sau khi xác định được Route 53 tại bước tiếp theo 

![Blog](/images/3-Blog/blog_1_2.jpg)

Hình 3: Giao diện App1

![Blog](/images/3-Blog/blog_1_3.jpg)

Hình 4: Giao diện App2

## Cấu hình Route 53 Health Check cho app1 và app2
1. Định nghĩa biến cho ALB:
```bash
export Region1ALB=$(aws elbv2 describe-load-balancers --region $Region1 --query "LoadBalancers[?contains(LoadBalancerName, '$ALBName')].DNSName | [0]" --output text)
export Region2ALB=$(aws elbv2 describe-load-balancers --region $Region2 --query "LoadBalancers[?contains(LoadBalancerName, '$ALBName')].DNSName | [0]" --output text)
```

2. Tạo health check cho app1 ở từng vùng:
```bash
aws route53 create-health-check --caller-reference "Region1App1-$(date +%s)" \
--health-check-config "{\"Port\":80,\"Type\":\"HTTP\",\"ResourcePath\":\"/app1health\",\"FullyQualifiedDomainName\":\"$Region1ALB\",\"RequestInterval\":30,\"FailureThreshold\":3,\"MeasureLatency\":true,\"Inverted\":false,\"Disabled\":false,\"EnableSNI\":false}"
aws route53 create-health-check --caller-reference "Region2App1-$(date +%s)" \
--health-check-config "{\"Port\":80,\"Type\":\"HTTP\",\"ResourcePath\":\"/app1health\",\"FullyQualifiedDomainName\":\"$Region2ALB\",\"RequestInterval\":30,\"FailureThreshold\":3,\"MeasureLatency\":true,\"Inverted\":false,\"Disabled\":false,\"EnableSNI\":false}"
```

3. Tạo health check cho app 2 ở từng vùng.
```bash
aws route53 create-health-check --caller-reference "Region1App1-$(date +%s)" \
--health-check-config "{\"Port\":80,\"Type\":\"HTTP\",\"ResourcePath\":\"/app2health\",\"FullyQualifiedDomainName\":\"$Region1ALB\",\"RequestInterval\":30,\"FailureThreshold\":3,\"MeasureLatency\":true,\"Inverted\":false,\"Disabled\":false,\"EnableSNI\":false}"
aws route53 create-health-check --caller-reference "Region2App1-$(date +%s)" \
--health-check-config "{\"Port\":80,\"Type\":\"HTTP\",\"ResourcePath\":\"/app2health\",\"FullyQualifiedDomainName\":\"$Region2ALB\",\"RequestInterval\":30,\"FailureThreshold\":3,\"MeasureLatency\":true,\"Inverted\":false,\"Disabled\":false,\"EnableSNI\":false}"
```

## Cấu hình alias record trong Route 53 cho app1 và app2
1. Định nghĩa các biến môi trường cho các health check đã tạo trước đó
```bash
export HealthCheckRegion1App1Id=$(aws route53 list-health-checks \
  --query "HealthChecks[?HealthCheckConfig.FullyQualifiedDomainName=='$Region1ALB' && HealthCheckConfig.ResourcePath=='/app1health'] | [0].Id" \
  --output text)
 
export HealthCheckRegion1App2Id=$(aws route53 list-health-checks \
  --query "HealthChecks[?HealthCheckConfig.FullyQualifiedDomainName=='$Region1ALB' && HealthCheckConfig.ResourcePath=='/app2health'] | [0].Id" \
  --output text)
 
export HealthCheckRegion2App1Id=$(aws route53 list-health-checks \
  --query "HealthChecks[?HealthCheckConfig.FullyQualifiedDomainName=='$Region2ALB' && HealthCheckConfig.ResourcePath=='/app1health'] | [0].Id" \
  --output text)
 
export HealthCheckRegion2App2Id=$(aws route53 list-health-checks \
  --query "HealthChecks[?HealthCheckConfig.FullyQualifiedDomainName=='$Region2ALB' && HealthCheckConfig.ResourcePath=='/app2health'] | [0].Id" \
  --output text)
```

2. Định nghĩa các biến môi trường cho CanonicalHostedZoneId của ALB. Chúng ta cần sử dụng các biến này khi tạo alias record ở các bước tiếp theo.
```bash
export Region1ALBHostedZoneId=$(aws elbv2 describe-load-balancers --region $Region1 --query "LoadBalancers[?DNSName=='$Region1ALB'].CanonicalHostedZoneId" --output text)
export Region2ALBHostedZoneId=$(aws elbv2 describe-load-balancers --region $Region2 --query "LoadBalancers[?DNSName=='$Region2ALB'].CanonicalHostedZoneId" --output text)
```

3. Tạo alias record cho app1. Trong ví dụ này, chúng ta sử dụng latency-based routing policy cho từng record.

```bash
aws route53 change-resource-record-sets --hosted-zone-id $HostedZoneId \
  --change-batch "{\"Changes\":[{\"Action\":\"CREATE\",\"ResourceRecordSet\":{\"Name\":\"$app1hostname\",\"Type\":\"A\",\"SetIdentifier\":\"$Region1\",\"Region\":\"$Region1\",\"HealthCheckId\":\"$HealthCheckRegion1App1Id\",\"AliasTarget\":{\"HostedZoneId\":\"$Region1ALBHostedZoneId\",\"DNSName\":\"$Region1ALB\",\"EvaluateTargetHealth\":false}}}]}"

aws route53 change-resource-record-sets --hosted-zone-id $HostedZoneId \
  --change-batch "{\"Changes\":[{\"Action\":\"CREATE\",\"ResourceRecordSet\":{\"Name\":\"$app1hostname\",\"Type\":\"A\",\"SetIdentifier\":\"$Region2\",\"Region\":\"$Region2\",\"HealthCheckId\":\"$HealthCheckRegion2App1Id\",\"AliasTarget\":{\"HostedZoneId\":\"$Region2ALBHostedZoneId\",\"DNSName\":\"$Region2ALB\",\"EvaluateTargetHealth\":false}}}]}"
```

4. Xác minh rằng các alias record cho app1 đã được tạo thành công trong Route 53 hosted zone

```bash
aws route53 list-resource-record-sets --hosted-zone-id $HostedZoneId --query "ResourceRecordSets[?Name=='$app1hostname.']"
```

5. Tạo alias record cho app2. Trong ví dụ này, chúng ta cũng sử dụng latency-based routing policy cho từng record.

```bash
aws route53 change-resource-record-sets --hosted-zone-id $HostedZoneId \
  --change-batch "{\"Changes\":[{\"Action\":\"CREATE\",\"ResourceRecordSet\":{\"Name\":\"$app2hostname\",\"Type\":\"A\",\"SetIdentifier\":\"$Region1\",\"Region\":\"$Region1\",\"HealthCheckId\":\"$HealthCheckRegion1App2Id\",\"AliasTarget\":{\"HostedZoneId\":\"$Region1ALBHostedZoneId\",\"DNSName\":\"$Region1ALB\",\"EvaluateTargetHealth\":false}}}]}"

aws route53 change-resource-record-sets --hosted-zone-id $HostedZoneId \
  --change-batch "{\"Changes\":[{\"Action\":\"CREATE\",\"ResourceRecordSet\":{\"Name\":\"$app2hostname\",\"Type\":\"A\",\"SetIdentifier\":\"$Region2\",\"Region\":\"$Region2\",\"HealthCheckId\":\"$HealthCheckRegion2App2Id\",\"AliasTarget\":{\"HostedZoneId\":\"$Region2ALBHostedZoneId\",\"DNSName\":\"$Region2ALB\",\"EvaluateTargetHealth\":false}}}]}"
```

6. Xác minh rằng các alias record cho app2 đã được tạo thành công trong Route 53 hosted zone

```bash
aws route53 list-resource-record-sets --hosted-zone-id $HostedZoneId --query "ResourceRecordSets[?Name=='$app2hostname.']"
```

Lưu ý: Việc cập nhật DNS có thể mất một khoảng thời gian để lan truyền trên toàn hệ thống.
Vì vậy, bạn có thể chưa truy cập được ngay vào các tên miền ứng dụng (app1.example.com và app2.example.com) sau khi tạo xong các bản ghi.

## Kiểm tra phản ứng của hệ thống khi một ứng dụng gặp sự cố

Với toàn bộ cấu hình đã được thiết lập, chúng ta có thể quan sát cách hệ thống phản ứng khi một ứng dụng riêng lẻ gặp lỗi. Hình 5 minh họa chuỗi sự kiện xảy ra trong kịch bản lỗi ứng dụng này. Trong điều kiện bình thường, người dùng ở Region 1 luôn được định tuyến đến các ứng dụng trong Region 1, vì chúng ta đã tạo Route 53 alias record sử dụng latency-based routing policy. Giả sử app1 gặp lỗi. Trong trường hợp đó, người dùng ở Region 1 truy cập app1 sẽ được chuyển hướng sang Region 2. Trong khi đó, người dùng ở Region 1 gửi yêu cầu đến app2 vẫn tiếp tục được phục vụ bởi app2 ở Region 1.

![Blog](/images/3-Blog/blog_1_4.jpg)

Hình 5: Kịch bản lỗi ứng dụng

Chuỗi sự kiện được thể hiện trong Hình 5 như sau:
1. Cả hai health check đều đang ở trạng thái ổn định.
2. Workload của app1 bị lỗi ở Region 1.
3. Route 53 health check (cho app1 ở Region 1) chuyển sang trạng thái bất ổn định.
4. Route 53 loại bỏ ALB của Region 1 ra khỏi DNS response.
5. Tất cả người dùng được chuyển hướng đến app1 ở Region 2.

Xác minh hành vi như Hình 5, để mô phỏng tình huống ứng dụng lỗi, bạn có thể giảm số replica của app1 về 0, bằng lệnh sau:
```bash
kubectl scale deployment demoapp1 --replicas=0
```

Trong ví dụ này, chúng ta sử dụng default timers của Route 53 health check. Do đó, sẽ mất vài phút để Route 53 bắt đầu chuyển hướng yêu cầu của người dùng từ Region 1 sang Region 2 cho app1. Bạn có thể cấu hình timer ngắn hơn (aggressive timers) để quá trình failover diễn ra nhanh hơn phù hợp với yêu cầu của mình.
Điểm khác biệt chính của phương pháp này là Phương pháp này có ưu thế nổi bật. Route 53 health check ở cấp ứng dụng chỉ phát hiện sự cố của app1 và chuyển hướng riêng traffic của app1 sang Region 2. Trong khi đó, app2 vẫn tiếp tục phục vụ người dùng ở Region 1, đảm bảo độ trễ tối ưu cho ứng dụng khỏe mạnh.

Hành vi failover chọn lọc này hoàn toàn khác biệt so với cơ chế giám sát ở cấp ALB truyền thống, nơi mà chỉ cần một ứng dụng lỗi sẽ khiến tất cả ứng dụng khác phải failover sang vùng khác, làm giảm hiệu năng không cần thiết cho các dịch vụ vẫn hoạt động bình thường. Giải pháp của chúng tôi đảm bảo rằng việc cách ly lỗi (failure isolation) diễn ra ở cấp ứng dụng, không phải ở cấp hạ tầng, giúp bạn kiểm soát chi tiết thực sự trong môi trường multi-tenant.

## Những điều cần biết

- Mỗi ứng dụng tiêu tốn 1 rule endpoint và 1 rule healthcheck trên ALB. Chú ý giới hạn số rule.
- Khi tạo alias record bằng console, Evaluate target health tự động sẽ là Yes. Nếu dùng AWS CLI hoặc các công cụ khác thì phải tự đặt.
- Cần cấu hình health check cho từng ứng dụng (có chi phí Route 53 health check).
- Route 53 health check dạng HTTPS không xác thực SSL/TLS certificate. Vậy nên check sẽ không thất bại tại certificate quá hạn hoặc invalid.
- Health check không giám sát IP private; nếu chạy trong private subnet, bạn có thể dùng custom health check với CloudWatch Alarm. Một custom health check sẽ sử dụng function Lambda (tham khảo bài Performing Route 53 health checks on private resources in a VPC with AWS Lambda and Amazon CloudWatch).
- Cấu hình HTTP keepalive duration của ALB quan trọng để client phát hiện lỗi nhanh. Nếu muốn biết thêm thông tin, tham khảo Application Load Balancer enables configuring HTTP client keepalive duration.
- Bài viết này không áp dụng cho mô hình path-based segregation (ví dụ: example.com/app1, example.com/app2).
- Tham khảo: Configuring DNS Failover để biết thêm về cách thức health check, alias records, failover work tại nhiều trường hợp.
- Dịch vụ Amazon Application Recovery Controller (ARC) cũng có thể giúp đạt Regional resiliency, nhưng ARC nằm ngoài phạm vi bài viết này. Để biết thêm thông tin chi tiết, vui lòng liên hệ ARC Developer Guide.

## Dọn dẹp tài nguyên

Bạn sẽ tiếp tục phát sinh chi phí cho đến khi xóa toàn bộ cơ sở hạ tầng được tạo trong quá trình thực hiện hướng dẫn này. Hãy sử dụng các lệnh sau để xóa các tài nguyên đã tạo.

Xóa các cụm EKS bằng các lệnh sau đây. eksctl sẽ tự động xóa toàn bộ tài nguyên ELB (bao gồm target groups, ALB, v.v.) — đây là các tài nguyên được ALB Controller tạo ra do yêu cầu Kubernetes ingress.
```bash
eksctl delete cluster --name=$Region1ClusterName --region=$Region1
eksctl delete cluster --name=$Region2ClusterName --region=$Region2
```
Xóa các Route 53 alias record
```bash
aws route53 change-resource-record-sets --hosted-zone-id $HostedZoneId \
  --change-batch "{\"Changes\":[{\"Action\":\"DELETE\",\"ResourceRecordSet\":{\"Name\":\"$app1hostname\",\"Type\":\"A\",\"SetIdentifier\":\"$Region1\",\"Region\":\"$Region1\",\"HealthCheckId\":\"$HealthCheckRegion1App1Id\",\"AliasTarget\":{\"HostedZoneId\":\"$Region1ALBHostedZoneId\",\"DNSName\":\"$Region1ALB\",\"EvaluateTargetHealth\":false}}}]}"
aws route53 change-resource-record-sets --hosted-zone-id $HostedZoneId \
  --change-batch "{\"Changes\":[{\"Action\":\"DELETE\",\"ResourceRecordSet\":{\"Name\":\"$app2hostname\",\"Type\":\"A\",\"SetIdentifier\":\"$Region1\",\"Region\":\"$Region1\",\"HealthCheckId\":\"$HealthCheckRegion1App2Id\",\"AliasTarget\":{\"HostedZoneId\":\"$Region1ALBHostedZoneId\",\"DNSName\":\"$Region1ALB\",\"EvaluateTargetHealth\":false}}}]}"
aws route53 change-resource-record-sets --hosted-zone-id $HostedZoneId \
  --change-batch "{\"Changes\":[{\"Action\":\"DELETE\",\"ResourceRecordSet\":{\"Name\":\"$app1hostname\",\"Type\":\"A\",\"SetIdentifier\":\"$Region2\",\"Region\":\"$Region2\",\"HealthCheckId\":\"$HealthCheckRegion2App1Id\",\"AliasTarget\":{\"HostedZoneId\":\"$Region2ALBHostedZoneId\",\"DNSName\":\"$Region2ALB\",\"EvaluateTargetHealth\":false}}}]}"
aws route53 change-resource-record-sets --hosted-zone-id $HostedZoneId \
  --change-batch "{\"Changes\":[{\"Action\":\"DELETE\",\"ResourceRecordSet\":{\"Name\":\"$app2hostname\",\"Type\":\"A\",\"SetIdentifier\":\"$Region2\",\"Region\":\"$Region2\",\"HealthCheckId\":\"$HealthCheckRegion2App2Id\",\"AliasTarget\":{\"HostedZoneId\":\"$Region2ALBHostedZoneId\",\"DNSName\":\"$Region2ALB\",\"EvaluateTargetHealth\":false}}}]}"
  ```
Xóa các Route 53 health check
```bash
aws route53 delete-health-check --health-check-id $HealthCheckRegion1App1Id
aws route53 delete-health-check --health-check-id $HealthCheckRegion1App2Id
aws route53 delete-health-check --health-check-id $HealthCheckRegion2App1Id
aws route53 delete-health-check --health-check-id $HealthCheckRegion2App2Id
```

Hãy xóa bất kỳ tài nguyên nào khác mà bạn đã tạo trong phần prerequisites, nếu chúng không còn cần thiết, để tránh phát sinh chi phí không mong muốn.

## Kết luận

Trong bài viết này, chúng tôi đã minh họa cách khắc phục hạn chế quan trọng trong kiến trúc multi-Region, multi-tenant Amazon EKS: hành vi failover “all-or-nothing” khiến ứng dụng khỏe mạnh cũng bị ảnh hưởng.
Bằng cách triển khai Route 53 health check riêng cho từng ứng dụng, bạn có thể đạt được độ chính xác cao trong chiến lược failover, chỉ chuyển hướng lưu lượng của ứng dụng bị lỗi và giữ hiệu năng tối ưu cho các ứng dụng còn lại.
Giải pháp này giúp bạn duy trì lợi ích chi phí của hạ tầng dùng chung, đáp ứng yêu cầu diverse availability, và loại bỏ cross-Region latency không cần thiết trong trường hợp lỗi cục bộ.
Đã sẵn sàng để nâng cao multi-Region resilience của bạn? Hãy làm các bước tiếp theo:
Mở rộng health check vượt ra ngoài phản hồi HTTP cơ bản, bao gồm kiểm tra các phụ thuộc nghiệp vụ quan trọng.
Tích hợp với Amazon Application Recovery Controller (ARC) để tự động hóa quá trình khôi phục sau thảm họa.


Xác định những ứng dụng trong hệ thống của bạn hưởng lợi nhiều nhất từ failover độc lập, đặc biệt là các ứng dụng có SLA khác nhau hoặc cơ sở người dùng theo vùng địa lý riêng biệt.


Tài liệu tham khảo:
Amazon EKS Best Practices Guide
Best Practices for Amazon Route 53
Designing for high availability and resiliency in Amazon EKS applications

## Giới thiệu về tác giả

### Dumlu Timuralp
![Blog](/images/3-Blog/blog_1_5.png)
Dumlu Timuralp là Senior Solutions Architect tại AWS, trụ sở tại Vương quốc Anh. Ông tư vấn kiến trúc về di chuyển lên cloud, hiện đại hóa ứng dụng và các mô hình cloud native. Ông yêu thích việc giúp người dùng đáp ứng nhu cầu kinh doanh thông qua công nghệ.

### Pratik Mankad
![Blog](/images/3-Blog/blog_1_6.jpeg)
Pratik Mankad là Network Solutions Architect tại AWS. Anh đam mê công nghệ mạng và luôn sáng tạo để giải quyết vấn đề của khách hàng. Anh thiết kế giải pháp và cung cấp hướng dẫn kỹ thuật nhằm giúp người dùng và đối tác đạt được mục tiêu kỹ thuật lẫn kinh doanh của họ.

