---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# DocuFlow AI
### 1. Tóm tắt dự án
DocuFlow AI là nền tảng bóc tách và chuẩn hóa thông tin hóa đơn/chứng từ tự động dựa trên mô hình phi tập trung Serverless và trí tuệ nhân tạo (LLM). Hệ thống cho phép người dùng tải lên các tệp hóa đơn dưới dạng PDF/PNG/JPG. Khi tệp được tải lên thành công vào S3 Raw Bucket, EventBridge sẽ nắm bắt sự kiện, đẩy yêu cầu vào SQS queue và kích hoạt Job Starter Lambda khởi chạy Step Functions State Machine. Workflow sẽ điều phối tuần tự các con Lambda: kiểm tra tính hợp lệ (Validate), bóc tách ký tự quang học qua Amazon Textract, gửi trường thô qua AI Proxy để chuẩn hóa dữ liệu bằng mô hình GPT-4o, và cuối cùng tính điểm tin cậy (Confidence Status) để quyết định lưu trữ dữ liệu vào DynamoDB / S3 Processed hoặc gửi cảnh báo qua SES/SNS đến quản trị viên nếu điểm tin cậy thấp cần phê duyệt thủ công.

### 2. Tuyên bố vấn đề
* **Quy trình thủ công kém hiệu quả**: Các doanh nghiệp thường phải nhập liệu thủ công thông tin hóa đơn từ khách hàng hoặc đối tác, gây tốn thời gian, dễ sai sót và khó quản lý khi số lượng hóa đơn lên đến hàng ngàn tệp mỗi ngày.
* **Chi phí cao và giải pháp phức tạp**: Các công cụ OCR truyền thống có chi phí bản quyền đắt đỏ, đòi hỏi cấu hình hạ tầng máy chủ cồng kềnh và không tự động chuẩn hóa được dữ liệu thô về một schema định dạng thống nhất (ví dụ: ngày tháng, tên nhà cung cấp, tổng tiền).
* **Giải pháp**: DocuFlow AI giải quyết triệt để vấn đề này bằng mô hình trả phí theo lượng sử dụng (Pay-for-value) của AWS Serverless, kết hợp sức mạnh xử lý OCR của Amazon Textract và khả năng chuẩn hóa ngữ nghĩa linh hoạt của External Large Language Model (GPT-4o), đồng thời bảo đảm an ninh dữ liệu tuyệt đối qua Secrets Manager và IAM Least Privilege.

### 3. Kiến trúc giải pháp
Kiến trúc hệ thống tận dụng tối đa mô hình xử lý bất đồng bộ (Asynchronous Event-Driven) để đảm bảo tính chịu tải cao và không gây nghẽn cổ chai:

1. **Frontend Client**: Người dùng đăng nhập qua Cognito User Pool, yêu cầu API Gateway cấp Presigned URL rồi đẩy file trực tiếp lên S3 Raw Bucket.
2. **Event Routing**: S3 Raw Bucket bắn sự kiện ObjectCreated sang EventBridge Rule ➔ đẩy vào SQS Processing Queue (có cấu hình Dead Letter Queue để hứng các message lỗi).
3. **Orchestration Workflow**: Job Starter Lambda poll message từ SQS và kích hoạt AWS Step Functions State Machine.
4. **AI Processing Pipeline**:
   * **Validate Lambda**: Kiểm tra định dạng và kích thước tệp tin.
   * **Textract Lambda**: Gọi Amazon Textract AnalyzeExpense API để bóc tách thông tin thô từ ảnh hóa đơn.
   * **AI Proxy Lambda**: Lấy API Key từ Secrets Manager và gọi External AI (GPT-4o) để chuẩn hóa cấu trúc dữ liệu theo schema DocuFlow.
   * **Confidence Status Lambda**: Chấm điểm tin cậy, lưu kết quả `result.json` vào S3 Processed Bucket và lưu metadata trạng thái (EXTRACTED, REVIEW_REQUIRED, FAILED) vào DynamoDB documents table.
5. **Observability & Alerts**: CloudWatch ghi nhận logs, X-Ray truy vết luồng xử lý, SNS/SES gửi cảnh báo email khi có lỗi hoặc khi trạng thái là REVIEW_REQUIRED.

### 4. Các dịch vụ AWS sử dụng
* **Amazon Cognito**: Quản lý định danh và cấp quyền đăng nhập cho người dùng.
* **Amazon API Gateway**: Cung cấp RESTful endpoints cấp Presigned URL và truy vấn dữ liệu.
* **Amazon S3**: Lưu trữ tài liệu gốc (Raw Bucket) và kết quả chuẩn hóa JSON (Processed Bucket).
* **Amazon EventBridge**: Định tuyến sự kiện tạo file bất đồng bộ.
* **Amazon SQS & DLQ**: Hàng đợi tin nhắn chịu tải và hàng đợi chứa tin nhắn lỗi để xử lý lại.
* **AWS Lambda**: Thực thi logic xử lý nghiệp vụ, tích hợp và chuyển đổi dữ liệu.
* **AWS Step Functions**: Điều phối quy trình xử lý AI tuần tự theo trạng thái (State Machine).
* **Amazon Textract**: Bóc tách hóa đơn, biên lai qua OCR trí tuệ nhân tạo chuyên biệt.
* **AWS Secrets Manager**: Lưu trữ an toàn các thông tin nhạy cảm và API Key.
* **Amazon DynamoDB**: Cơ sở dữ liệu NoSQL lưu trữ metadata hóa đơn (Single-Table Design).
* **Amazon SNS & SES**: Gửi thông báo và email cảnh báo cho quản trị viên.
* **Amazon CloudWatch & AWS X-Ray**: Giám sát hiệu năng hệ thống, lưu trữ log và truy vết luồng hoạt động.
* **AWS Budgets**: Thiết lập giới hạn chi phí bảo vệ tài chính cho toàn bộ hạ tầng dự án.
* **AWS CloudTrail**: Ghi nhận nhật ký audit phục vụ quản trị an toàn thông tin hạ tầng.
![image](/images/2-Proposal/image.png)