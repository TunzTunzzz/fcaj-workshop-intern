---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---


###  [Blog 1 - Xây dựng lớp tìm kiếm người dùng có khả năng mở rộng trên Amazon Cognito](3.1-Blog1/)  
Bài viết giới thiệu một cách tiếp cận thực tế để nâng cấp trải nghiệm tìm kiếm người dùng trên hệ sinh thái AWS, cụ thể là tích hợp thêm sức mạnh truy vấn cho dịch vụ Amazon Cognito. Bằng cách phối hợp nhịp nhàng các công cụ Serverless mạnh mẽ như Lambda, DynamoDB và OpenSearch Service, kiến trúc này không chỉ hỗ trợ tự động đồng bộ dữ liệu mà còn cung cấp khả năng tìm kiếm linh hoạt (như tìm kiếm gần đúng, lọc nhiều tiêu chí cùng lúc). Đặc biệt, hệ thống đảm bảo thời gian phản hồi gần như ngay lập tức và tự động co giãn tài nguyên theo tải thực tế.

###  [Blog 2 - Xây dựng Cơ chế Retry Thông minh cho Serverless Queue Consumer trên AWS](3.2-Blog2/)
Tài liệu hướng dẫn này cung cấp một bản thiết kế kiến trúc chi tiết, giúp tăng cường độ tin cậy và khả năng chống chịu lỗi (fault tolerance) cho các ứng dụng Serverless. Trọng tâm của bài viết là cách kết hợp hiệu quả giữa AWS Lambda, Amazon SQS và EventBridge Scheduler nhằm xử lý gọn gàng những gián đoạn tạm thời từ các dịch vụ đích (downstream). Giải pháp nổi bật ở việc tách biệt hoàn toàn phần code xử lý lỗi (retry logic) khỏi phần nghiệp vụ chính (business logic). Điều này mang lại sự chủ động trong việc lên lịch gửi lại request, đồng thời phối hợp mượt mà với Dead Letter Queue nhằm loại bỏ rủi ro thất thoát dữ liệu.

###  [Blog 3 - Cách ALS GeoAnalytics LITHOLENS™ Cách Mạng Hóa Việc Ghi Chép Mẫu Lõi Khoan Bằng Machine Learning Trên Amazon EKS](3.3-Blog3/)
Đây là một case study điển hình về việc ứng dụng trí tuệ nhân tạo (Machine Learning) cùng với nền tảng điện toán mềm dẻo Amazon EKS để chuyển đổi số cho ngành công nghiệp nặng. Cụ thể, thông qua hệ thống LITHOLENS™, công ty ALS GeoAnalytics đã thay thế hoàn toàn công việc ghi chép mẫu lõi khoan vốn dĩ rất thủ công, tốn kém và mất nhiều thời gian. Sự chuyển dịch sang mô hình tự động hóa này giúp cải thiện đáng kể độ chính xác của dữ liệu. Hơn thế nữa, việc tận dụng kiến trúc Container và Serverless của AWS còn giúp doanh nghiệp cắt giảm chi phí vận hành và dễ dàng mở rộng mô hình này sang các mảng kỹ thuật khác.

###  [Blog 4 - XÂY DỰNG DASHBOARD THEO DÕI PATCH COMPLIANCE TRÊN NHIỀU AWS ACCOUNT VỚI KIRO SPECS](3.4-Blog4/)
Nội dung bài viết đi sâu vào quy trình Kiro Specs nhằm thiết lập một hệ thống Dashboard quản lý tuân thủ bản vá (patch compliance) tập trung dành cho các hệ thống có nhiều tài khoản AWS. Cốt lõi của giải pháp là việc trích xuất dữ liệu gốc bằng AWS Systems Manager Patch Manager rồi lưu về Amazon S3, sau đó dùng Lambda để nhào nặn lại thông tin. Cuối cùng, dữ liệu được hiển thị trên một giao diện nội bộ bảo mật nhờ Application Load Balancer và Session Manager. Thông qua ba bước của Kiro Specs (Yêu cầu -> Thiết kế -> Công việc), bài viết minh họa sinh động cách chúng ta có thể tận dụng AI để sinh ra một bản thiết kế Serverless vừa hiệu quả vừa tuân thủ các quy tắc bảo mật khắt khe.