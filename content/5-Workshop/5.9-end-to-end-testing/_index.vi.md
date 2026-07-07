---
title: "Kiểm thử tích hợp End-to-End"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

# Kiểm thử tích hợp End-to-End

### 1. Mục tiêu (Goal)
Tiến hành chạy thử nghiệm liên thông toàn bộ hệ thống từ giao diện người dùng đến luồng xử lý ngầm, kiểm tra hoạt động của các nhánh xử lý logic của Step Functions và thu thập bằng chứng kiểm thử (evidence).

### 2. Các kịch bản kiểm thử (Test Cases)

#### Kịch bản 1: Luồng trích xuất thành công (Happy Path - Hóa đơn rõ nét)
* **Mô phỏng thao tác**: Người dùng đăng nhập qua Cognito trên UI, tải lên tệp tin `invoice.pdf` có chất lượng hình ảnh sắc nét.
* **Quy trình hệ thống**: 
  1. Frontend gửi yêu cầu sinh Presigned URL từ `upload-url` Lambda.
  2. Frontend tải tệp gốc lên S3 Raw Bucket.
  3. S3 bắn sự kiện tải tệp -> EventBridge chuyển tiếp -> SQS Queue nhận tin nhắn.
  4. Job Starter Lambda poll tin nhắn SQS, ra lệnh khởi chạy Step Functions.
  5. Validate Lambda kiểm tra tệp tin hợp lệ.
  6. Textract Lambda gọi API `AnalyzeExpense` lấy trường thô.
  7. AI Proxy Lambda gọi External AI chuẩn hóa cấu trúc dữ liệu thô thành JSON.
  8. Confidence & Status Lambda tính toán điểm tin cậy trung bình đạt trên 90% (ví dụ: `0.99`). Hệ thống cập nhật bản ghi DynamoDB thành trạng thái `EXTRACTED` và lưu file `result.json` vào S3 Processed.
  9. Frontend tự động làm mới và tải dữ liệu kết quả hiển thị lên bảng Dashboard.
* **Bằng chứng**: Chụp màn hình trang chi tiết tài liệu hiển thị trạng thái `EXTRACTED` cùng đầy đủ bảng thông tin trích xuất.

  *(Lưu ảnh chạy thành công Happy Path vào `static/images/5-Workshop/5.9-end-to-end-testing/happy-path.png`)*
  ![Happy Path](/images/5-Workshop/5.9-end-to-end-testing/happy-path.png)

#### Kịch bản 2: Luồng kiểm duyệt thủ công (Low-Confidence Path - Hóa đơn mờ/thiếu trường)
* **Mô phỏng thao tác**: Tải lên tệp tin hóa đơn bị mờ hoặc thiếu một số trường quan trọng như tổng tiền (totalAmount).
* **Quy trình hệ thống**:
  1. Các bước xử lý trung gian diễn ra tương tự kịch bản Happy Path.
  2. Tại bước tính điểm tin cậy, Confidence & Status Lambda phát hiện điểm tin cậy trung bình < 90% hoặc thiếu trường bắt buộc (`vendorName`, `totalAmount`).
  3. Lambda chuyển trạng thái bản ghi trong DynamoDB thành `REVIEW_REQUIRED` và gán trạng thái review là `PENDING`.
  4. Hệ thống kích hoạt CloudWatch Alarm bắn cảnh báo lỗi sang SNS, gửi email cảnh báo về điện thoại/hộp thư Admin.
  5. Admin/Reviewer mở trang Phê duyệt (Reviewer Dashboard) trên UI, xem biểu mẫu chỉnh sửa thông tin bị nhận diện sai.
  6. Sau khi sửa lại số tiền và nhấn **Approve** (Duyệt), Frontend gọi API `review` Lambda cập nhật trạng thái bản ghi trong DynamoDB thành `APPROVED` và ghi đè kết quả `result.json` đã sửa lên S3.
* **Bằng chứng**: Chụp giao diện Reviewer Form hiển thị các trường chỉnh sửa và ảnh email cảnh báo từ SNS.

  *(Lưu ảnh kiểm duyệt thủ công vào `static/images/5-Workshop/5.9-end-to-end-testing/review-required-path.png`)*
  ![Review Path](/images/5-Workshop/5.9-end-to-end-testing/review-required-path.png)

#### Kịch bản 3: Luồng xử lý lỗi (Failed Path - File sai định dạng/quá tải)
* **Mô phỏng thao tác**: Tải lên tệp tin định dạng `.txt` (hoặc tệp ảnh dung lượng > 10MB).
* **Quy trình hệ thống**:
  1. Tệp tin được đẩy lên S3 Raw bucket và kích hoạt luồng Step Functions.
  2. Tại bước đầu tiên, Validate Lambda thực hiện kiểm duyệt và phát hiện tệp tin vi phạm luật nghiệp vụ (sai định dạng tệp tin cho phép).
  3. Validate Lambda thực hiện cập nhật trạng thái bản ghi trong DynamoDB thành `FAILED`, ghi rõ lý do lỗi (ví dụ: `INVALID_FILE_TYPE`).
  4. Lambda quăng lỗi, Step Functions catch lỗi và chuyển nhánh sang **Failed state** để dừng chạy.
  5. CloudWatch Alarm kích hoạt SNS gửi email cảnh báo lỗi hệ thống cho Admin.
* **Bằng chứng**: Chụp màn hình sơ đồ Step Functions hiển thị nút đỏ (Fail) tại bước Validate và ảnh bản ghi DynamoDB có status `FAILED`.

  *(Lưu ảnh chạy kịch bản Failed Path vào `static/images/5-Workshop/5.9-end-to-end-testing/failed-path.png`)*
  ![Failed Path](/images/5-Workshop/5.9-end-to-end-testing/failed-path.png)

#### Kịch bản 4: Tin nhắn rơi vào hàng đợi chết (SQS DLQ Path)
* **Mô phỏng thao tác**: Thu hồi quyền đọc Secrets Manager của hàm AI Proxy Lambda để mô phỏng lỗi sập logic liên tục khi gọi API.
* **Quy trình hệ thống**:
  1. Khi người dùng upload tệp tin mới, SQS nhận tin nhắn và gọi Job Starter Lambda kích hoạt Step Functions.
  2. Luồng chạy bị sập tại bước AI Proxy do lỗi phân quyền (`AccessDenied`).
  3. Tin nhắn được rollback lại SQS và Lambda retry liên tục.
  4. Sau khi số lần thử lại vượt ngưỡng tối đa (`Maximum receives = 3`), tin nhắn lỗi tự động được đưa sang Dead Letter Queue `docuflow-dev-processing-dlq`.
  5. CloudWatch Alarm `docuflow-dev-sqs-dlq-not-empty-alarm` kích hoạt SNS gửi mail cảnh báo tin nhắn kẹt.
* **Bằng chứng**: Chụp màn hình SQS Console hiển thị chỉ số tin nhắn hiển thị tại DLQ > 0.

  *(Lưu ảnh SQS DLQ nhận tin nhắn vào `static/images/5-Workshop/5.9-end-to-end-testing/dlq-messages.png`)*
  ![DLQ Messages](/images/5-Workshop/5.9-end-to-end-testing/dlq-messages.png)

### 3. Kết quả mong đợi (Expected Result)
* Luồng xử lý bất đồng bộ hoạt động chính xác cho cả 4 kịch bản kiểm thử.
* Cơ chế ghi vết và gửi cảnh báo tự động hoạt động mượt mà khi xảy ra lỗi hoặc khi có hóa đơn cần review.
* Dữ liệu đồng bộ và lưu trữ nhất quán giữa S3 và DynamoDB.
