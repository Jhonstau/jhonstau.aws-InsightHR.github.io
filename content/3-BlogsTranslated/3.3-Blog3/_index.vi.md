---
title: "Blog 3"
date: ""
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Nâng cao khả năng kiểm tra TLS với SNI session holding trong AWS Network Firewall
bởi Amit Gaur, Olu Adeleke, Srivalsan Mannoor Sudhagar, và Vikram Saurabh | vào ngày 17 tháng 9 năm 2025 | tại Advanced (300), AWS Network Firewall, Security, Identity, & Compliance, Technical How-to | Permalink | Bình Luận | Chia Sẻ

AWS Network Firewall là một dịch vụ managed firewall, có nhiệm vụ lọc và kiểm soát lưu lượng mạng trong Amazon Virtual Private Cloud (Amazon VPC). Không giống như các cơ chế kiểm soát mạng truyền thống như security groups hay network access control lists (NACLs), Network Firewall có thể kiểm tra và ra quyết định dựa trên thông tin từ các tầng cao hơn của mô hình OSI, bao gồm Transport đến Application layers. Ngoài ra, bạn có thể sử dụng TLS inspection capability của Network Firewall để tạo các quy tắc firewall dựa trên nội dung của lưu lượng được mã hóa TLS. Network Firewall sẽ giải mã lưu lượng bằng chứng chỉ (certificate) bạn cấu hình và so sánh payload đã giải mã với các quy tắc trong firewall policy.

Bài viết này giới thiệu Server Name Indication (SNI) session holding, một tính năng mới giúp tăng cường khả năng kiểm tra TLS bằng cách ngăn chặn các gói TCP hoặc TLS handshake đến server đích cho đến khi các quy tắc kiểm tra TLS/SNI được áp dụng. Khi SNI session holding được bật, Network Firewall sẽ không khởi tạo kết nối TCP ra ngoài (outbound) cho đến khi nó nhận được client hello và so khớp thông tin domain trong SNI với các quy tắc firewall. Chỉ sau khi xác thực domain hợp lệ, firewall mới bắt đầu tạo kết nối TCP đến server upstream. Điều này giúp tăng cường kiểm soát bảo mật cho lưu lượng outbound mà không ảnh hưởng đáng kể đến độ trễ hoặc hiệu năng, giúp bảo vệ khỏi các kết nối độc hại.

## Cách TLS inspection hoạt động trước khi có SNI session holding

Khi bật TLS inspection, Network Firewall hoạt động như một proxy trung gian giữa client và server, duy trì hai kết nối riêng biệt với từng đầu mối. Trong suốt quá trình này, firewall sẽ đánh giá lưu lượng outbound dựa trên các quy tắc đã cấu hình để xác định xem có cho phép lưu lượng đó ra ngoài hay không. Trước khi SNI session holding ra mắt, các bước được thực hiện như minh họa trong Hình 1:
1. Client tạo một kết nối TCP, và Network Firewall đánh giá các quy tắc stateless để xác định xem lưu lượng có được phép hay không. Nếu không, kết nối sẽ bị chấm dứt.
2. Network Firewall tạo một kết nối TCP đến máy chủ đích.
3. Client gửi một thông điệp ClientHello, bao gồm thông tin SNI, đến Network Firewall. Tường lửa xác thực rằng SNI hợp lệ, nếu không kết nối sẽ bị chấm dứt.
4. Network Firewall chuyển tiếp thông điệp ClientHello đến máy chủ đích.
5. Máy chủ đích phản hồi bằng một thông điệp ServerHello và chứng chỉ của nó.
6. Network Firewall xác thực các chứng chỉ được tải xuống từ máy chủ đích.
7. Tại thời điểm này, server name indication được xác thực so với tên chủ thể của chứng chỉ.
8. Network Firewall chuyển tiếp chứng chỉ của máy chủ đến client và hoàn tất kết nối TLS với client.
9. Client mã hóa application payload bằng các session key mà nó đã thương lượng trong quá trình bắt tay TLS và gửi nó đến Network Firewall.
10. Network Firewall giải mã lưu lượng, sử dụng stateful engine của nó để đánh giá các quy tắc so với lưu lượng, và xác định xem nó có được phép hay không.
11. Nếu lưu lượng được phép, Network Firewall mã hóa lại application layer payload bằng các session key của máy chủ đích và chuyển tiếp nó đến máy chủ đích.
12. Máy chủ đích gửi dữ liệu phản hồi trở lại Network Firewall.
13. Stateful engine của Network Firewall phân tích phản hồi của máy chủ đích.
14. Network Firewall chuyển tiếp phản hồi của máy chủ đến client. Quá trình giao tiếp tiếp tục cho đến khi client hoặc máy chủ đích chấm dứt kết nối.

![Blog](/images/3-Blog/blog_3_0.png)
Hình 1: Các bước trước khi có SNI session holding

Trong quy trình trên, kết nối TCP đã được thiết lập trước khi firewall đánh giá SNI, điều này đồng nghĩa server có thể nhận biết kết nối trước khi firewall kiểm tra tính hợp lệ của domain.
Ví dụ, nếu người dùng cấu hình rule để chặn lưu lượng dựa trên TLS SNI (ví dụ example.com), họ kỳ vọng kết nối sẽ bị chặn trước khi kết nối TCP đến server được mở.
 Tuy nhiên, theo chuỗi xử lý cũ, TCP connection được tạo ngắn ngủi trước khi SNI rule validation diễn ra. Điều này tạo ra một “khoảng trống nhỏ về bảo mật”, mà kẻ tấn công tinh vi có thể lợi dụng để vượt qua kiểm soát ngăn chặn rò rỉ dữ liệu (data exfiltration prevention), ngay cả khi cấu hình SNI rule đã chính xác.
Session holding giải quyết vấn đề này bằng cách đảm bảo rằng lưu lượng từ trong VPC sẽ không thể kết nối ra ngoài cho đến khi Network Firewall xác thực SNI thành công.

## Cách TLS inspection hoạt động khi bật session holding

SNI session holding thực hiện quy trình xác thực hai bước. Đầu tiên, firewall kiểm tra tầng TLS và xác thực SNI khi nhận ClientHello từ client. Sau đó, chỉ khi được phê duyệt, firewall mới tạo kết nối TCP đến server đích, cho phép các giao thức tầng trên (như HTTP, SMTP) bắt đầu trao đổi. Cách tiếp cận này giúp tách biệt rõ ràng giữa quá trình xác thực TLS và kiểm tra giao thức, đảm bảo rằng chỉ lưu lượng TLS hợp lệ mới được phép khởi tạo trao đổi. Các bước minh họa trong Hình 2 như sau: 
Ghi chú: Các bước 2–5 là phần của SNI session holding
1. Client tạo một kết nối TCP, và Network Firewall đánh giá các quy tắc stateless để xác định xem lưu lượng có được phép hay không. Nếu không, kết nối sẽ bị chấm dứt.
2. Client gửi một thông điệp ClientHello bao gồm thông tin SNI đến Network Firewall. Network Firewall thực hiện xác thực SNI.
3. Tường lửa đánh giá các quy tắc kiểm tra TLS, bao gồm cả quy tắc SNI, để xác định xem lưu lượng có được phép hay không. Nếu không, kết nối sẽ bị chấm dứt.
4. Network Firewall tạo một kết nối TCP đến máy chủ đích.
5. Network Firewall chuyển tiếp thông điệp ClientHello đến máy chủ đích.
6. Máy chủ đích phản hồi bằng một thông điệp ServerHello và chứng chỉ của nó.
7. Network Firewall xác thực các chứng chỉ được tải xuống từ máy chủ đích.
8. Network Firewall chuyển tiếp chứng chỉ của máy chủ đến client và hoàn tất kết nối TLS với client.
9. Client mã hóa application payload bằng các session key mà nó đã thương lượng trong quá trình bắt tay TLS và gửi nó đến Network Firewall.
10. Network Firewall giải mã lưu lượng, sử dụng stateful engine của nó để đánh giá các quy tắc so với lưu lượng, và xác định xem nó có được phép hay không.
11. Nếu lưu lượng được phép, Network Firewall mã hóa lại application layer payload bằng các session key của máy chủ đích và chuyển tiếp nó đến máy chủ đích.
12. Máy chủ đích gửi dữ liệu phản hồi trở lại Network Firewall.
13. Stateful engine của Network Firewall phân tích phản hồi của máy chủ đích.
14. Network Firewall chuyển tiếp phản hồi của máy chủ đến client. Quá trình giao tiếp tiếp tục cho đến khi client hoặc máy chủ đích chấm dứt kết nối.

![Blog](/images/3-Blog/blog_3_1.jpeg)
Hình 2: Các bước sau khi bật session holding

## Bắt đầu sử dụng

Bạn có thể bật session holding khi tạo cấu hình TLS inspection trong Network Firewall policy, thông qua AWS Management Console, AWS CLI, hoặc AWS SDK.

## Yêu cầu

Để bắt đầu cấu hình policy có session holding, hãy truy cập AWS Network Firewall console hoặc tham khảo AWS Network Firewall Developer Guide. Session holding được hỗ trợ ở tất cả AWS Regions nơi Network Firewall khả dụng, bao gồm cả AWS GovCloud (US) và China Regions.
Nếu đây là lần đầu tiên bạn sử dụng Network Firewall, cần hoàn thành các bước chuẩn bị sau:
1. Tạo firewall
2. Tạo TLS inspection configuration

## Bật session holding

Khi tạo firewall policy, ở bước Add TLS Inspection configuration, bạn sẽ thấy tùy chọn Enable session holding, như trong Hình 3.
![Blog](/images/3-Blog/blog_3_2.png)
Hình 3: Bật session holding
Sau khi thêm cấu hình TLS inspection và chọn hộp kiểm Enable session holding, tiếp tục tạo firewall policy mới, rồi gán policy này cho firewall.
Nếu bạn đã có policy hiện có gắn với cấu hình TLS inspection, chọn Manage TLS Inspection Configuration để mở tùy chọn bật session holding.
![Blog](/images/3-Blog/blog_3_3.png)
Hình 4: TLS inspection configuration
Cái này sẽ cho bạn lựa chọn để bật session holding như ở Hình 3.

## Giá Cả

SNI session holding được bao gồm trong chi phí TLS advanced inspection. Xem chi tiết tại AWS Network Firewall pricing.

## Lưu ý

Khi bật session holding, cần chú ý:
Keywords: Session holding chỉ áp dụng cho các quy tắc Suricata sử dụng từ khóa TLS.SNI. Không áp dụng cho TLS.CERT hoặc TLS.VERSION.
Hiệu năng: Vì các gói TCP handshake bị giữ lại cho đến khi xác thực SNI xong, nên có thể tăng độ trễ khi có nhiều kết nối mới đồng thời.
Tương thích: Khi bật session holding, TLS.SNI sẽ ưu tiên cao hơn http.host. Khi tắt, cả hai có thể áp dụng song song. Khi bật, chỉ quy tắc chứa TLS.SNI được áp dụng trước, còn http.host chỉ hoạt động nếu traffic giải mã không khớp quy tắc SNI nào.

## Kết luận

Như một biện pháp phòng ngừa, SNI session holding giúp đảm bảo SNI được xác thực trước khi tạo kết nối với server đích, tránh việc liên hệ ban đầu với endpoint độc hại.
 Tính năng này tăng cường bảo mật cho lưu lượng outbound mà không cần thay đổi hạ tầng hiện tại. Để biết thêm thông tin, hãy xem: What is AWS Network Firewall?

## Giới thiệu về tác giả

### Amit Gaur
![Blog](/images/3-Blog/blog_3_4.jpg)
Amit là Cloud Infrastructure Architect tại AWS, đam mê công nghệ và chia sẻ kiến thức trong cộng đồng networking. Anh chuyên về thiết kế kiến trúc mạng, giúp khách hàng xây dựng môi trường có khả năng mở rộng cao (highly scalable) và độ tin cậy lớn (resilient) trên AWS. Thông qua kiến thức kỹ thuật và kinh nghiệm thiết kế, Amit hỗ trợ khách hàng đẩy nhanh hành trình lên cloud đồng thời đảm bảo hệ thống được tối ưu cho scale và reliability.

### Srivalsan Mannoor Sudhagar
![Blog](/images/3-Blog/blog_3_5.jpg)
Srivalsan là Sr. Cloud Infrastructure Architect tại AWS Professional Services, với chuyên môn trong Cloud Infrastructure và MLOps. Anh đam mê networking và container technologies, luôn tìm cách đổi mới để giải quyết vấn đề của khách hàng. Anh thích thiết kế giải pháp và cung cấp hướng dẫn kỹ thuật giúp khách hàng và đối tác đạt được mục tiêu kỹ thuật và kinh doanh.

### Vikram Saurabh
![Blog](/images/3-Blog/blog_3_6.jpeg)
Vikram là engineering leader có 20 năm kinh nghiệm trong lĩnh vực software engineering, đặc biệt trong xây dựng sản phẩm và dịch vụ firewall. Hiện anh dẫn dắt đội ngũ kỹ sư của AWS Network Firewall, và trước đó từng lãnh đạo đội Route53 DNS Firewall. Ngoài công việc, Vikram yêu thích chơi cricket, leo núi (hiking) và giải đố toán học.

### Olu Adeleke
![Blog](/images/3-Blog/blog_3_7.jpeg)
Olu là Senior Software Engineer với hơn 10 năm kinh nghiệm trong phát triển phần mềm và mạng máy tính. Anh là technical lead của nhiều dự án và tính năng trong AWS Network Firewall, đồng thời sở hữu bằng Tiến sĩ (Ph.D.) về Khoa học Máy tính. Ngoài công việc, Olu yêu thích bóng đá, vẽ phong cảnh, và dành thời gian cho gia đình và bạn bè.
