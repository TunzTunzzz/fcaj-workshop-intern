---
title: "Dọn dẹp tài nguyên"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

# Dọn dẹp tài nguyên

### 1. Mục tiêu (Goal)
Thực hiện dọn dẹp và xóa bỏ toàn bộ các tài nguyên AWS đã tạo sau khi kết thúc buổi workshop, nhằm tránh phát sinh chi phí phát sinh ngoài ý muốn và chứng minh kiểm soát ngân sách dự án.

### 2. Các bước thực hiện (Steps)

#### Bước 1: Dọn rác trong các S3 Buckets và tiến hành xóa bucket
AWS quy định không cho phép xóa S3 bucket nếu bên trong vẫn còn chứa tệp tin. Do đó, chúng ta cần làm rỗng (Empty) trước khi xóa:
1. Vào dịch vụ **S3**. Lần lượt thực hiện cho 3 buckets:
   * `docuflow-dev-raw-<MÃ_ACCOUNT>-ap-southeast-1`
   * `docuflow-dev-processed-<MÃ_ACCOUNT>-ap-southeast-1`
   * `docuflow-dev-cloudtrail-logs-<mã-ngẫu-nhiên>`
2. Với từng bucket, tích chọn vào tên của nó -> Chọn **Empty** -> Gõ cụm từ `permanently delete` để xác nhận xóa toàn bộ tệp tin bên trong.
3. Sau khi bucket đã trống, chọn lại bucket -> Chọn **Delete** -> Nhập tên của bucket để xóa hoàn toàn bucket khỏi hệ thống.

#### Bước 2: Xóa bảng cơ sở dữ liệu DynamoDB
1. Truy cập **DynamoDB** -> Chọn mục **Tables**.
2. Tích chọn bảng dữ liệu chính của dự án: `docuflow-dev-documents-table`.
3. Nhấn nút **Delete** ở góc trên cùng -> Gõ chữ `confirm` vào ô xác nhận và nhấn **Delete** để xóa bảng.

#### Bước 3: Xóa các hàm xử lý Backend Lambda
1. Truy cập **Lambda** -> Tại danh sách **Functions**, gõ cụm từ `docuflow-dev-` vào ô tìm kiếm để lọc ra tất cả các hàm của dự án.
2. Tích chọn tất cả các hàm này cùng một lúc (bao gồm cụm API Data, AI Lambdas, Job Starter, Validate).
3. Nhấn nút **Actions** -> Chọn **Delete** -> Gõ `confirm` và nhấn **Delete**.

#### Bước 4: Xóa API Gateway kết nối
1. Truy cập **API Gateway**.
2. Chọn API của dự án: `docuflow-dev-api`.
3. Nhấn vào **Actions** ở góc trên -> Chọn **Delete API** -> xác nhận đồng ý xóa.

#### Bước 5: Xóa các đường truyền cảnh báo CloudWatch Alarms
1. Truy cập **CloudWatch** -> Ở menu bên trái, chọn **Alarms** -> **All alarms**.
2. Tích chọn 4 bộ Alarms đã tạo:
   * `docuflow-dev-workflow-failed-alarm`
   * `docuflow-dev-sqs-dlq-not-empty-alarm`
   * `docuflow-dev-lambda-ai-proxy-error-alarm`
   * `docuflow-dev-low-confidence-spike-alarm`
3. Nhấn nút **Actions** -> Chọn **Delete** -> nhấn **Delete** để xác nhận.

#### Bước 6: Xóa CloudWatch Log Groups phát sinh
1. Vẫn ở CloudWatch, chọn **Logs** -> **Log groups**.
2. Tích chọn lần lượt 6 nhóm Log Groups của các hàm Lambda đã cấu hình:
   * `/aws/lambda/docuflow-dev-api-upload-url-lambda`
   * `/aws/lambda/docuflow-dev-ingestion-job-starter-lambda`
   * `/aws/lambda/docuflow-dev-ingestion-validate-lambda`
   * `/aws/lambda/docuflow-dev-ai-textract-lambda`
   * `/aws/lambda/docuflow-dev-ai-proxy-lambda`
   * `/aws/lambda/docuflow-dev-ai-confidence-status-lambda`
3. Nhấn nút **Actions** -> Chọn **Delete log group(s)** -> xác nhận xóa.

#### Bước 7: Xóa các kênh thông báo SNS Topic & SES Email Identity
1. **SNS Topic**: Truy cập **Simple Notification Service (SNS)** -> chọn **Topics** -> Tích chọn topic `docuflow-dev-notification-system-alerts-topic` -> Nhấn **Delete** -> Gõ `delete me` để xóa.
2. **SES Email Identity**: Truy cập **Simple Email Service (SES)** -> chọn **Verified identities** -> Tích chọn địa chỉ email nhận tin của bạn -> Nhấn **Delete** -> xác nhận xóa.

#### Bước 8: Xóa luồng giám sát lịch sử CloudTrail
1. Truy cập **CloudTrail** -> Chọn mục **Trails** ở menu bên trái.
2. Bấm vào Trail của nhóm: `docuflow-dev-audit-trail`.
3. Nhấn nút **Stop logging** (Dừng ghi log) -> sau đó bấm **Delete** để xóa trail.

#### Bước 9: Xóa hệ thống kiểm soát ngân sách AWS Budgets
1. Gõ **Budgets** trên thanh Search -> Chọn **AWS Budgets**.
2. Tích chọn 3 bộ ngân sách đã tạo:
   * `DocuFlow-Monthly-Actual-Cost`
   * `DocuFlow-Credit-Safety`
   * `DocuFlow-FreeTier-Watch`
3. Nhấn nút **Actions** -> Chọn **Delete** -> xác nhận xóa bỏ.

#### Bước 10: Xóa chìa khóa API ngoài trong Secrets Manager
1. Truy cập **Secrets Manager** -> chọn secret `docuflow-dev-external-ai-api-key`.
2. Nhấn nút **Actions** -> Chọn **Delete secret**.
3. Tại ô chọn số ngày chờ xóa, chọn số ngày tối thiểu là **7 ngày** để hệ thống khóa bí mật lại ngay lập tức và không tính phí duy trì.

#### Bước 11: Xóa các IAM Roles & Policies đã tạo
1. Truy cập **IAM** -> Chọn mục **Roles**.
2. Tìm kiếm và chọn lần lượt 9 bộ Roles đã tạo thủ công ở bước 3:
   * `docuflow-dev-security-upload-url-role`
   * `docuflow-dev-security-job-starter-role`
   * `docuflow-dev-workflow-stepfunctions-role`
   * `docuflow-dev-ai-textract-lambda-role`
   * `docuflow-dev-security-ai-proxy-role`
   * `docuflow-dev-data-lambda-role`
   * `docuflow-dev-notification-lambda-role`
   * `docuflow-dev-ingestion-validate-lambda-role`
   * `docuflow-dev-ai-confidence-status-lambda-role`
3. Nhấn **Delete** và gõ tên Role để xác nhận gỡ bỏ.
4. Chuyển sang mục **Policies** -> Tìm kiếm và xóa tất cả các policy tùy chỉnh do bạn tự viết (như `docuflow-dev-ingestion-s3-raw-access-policy`, `docuflow-dev-ai-secret-read-policy`, v.v.).

#### Bước 12: Lên lịch xóa khóa mã hóa gốc KMS Key (Bắt buộc làm cuối cùng)
1. Truy cập **Key Management Service (KMS)** -> chọn **Customer managed keys**.
2. Tích chọn khóa CMK: `docuflow-dev-main-key`.
3. Nhấn nút **Key actions** -> Chọn **Schedule key deletion**.
4. Tại ô **Waiting period (in days)**, gõ số ngày chờ tối thiểu là **7** (AWS bắt buộc thời gian chờ từ 7-30 ngày để tránh xóa nhầm).
5. Bấm nút xác nhận màu cam để hoàn tất. Khóa sẽ chuyển sang trạng thái `Pending deletion` và dừng tính phí duy trì.

### 3. Kết quả mong đợi (Expected Result)
* Tất cả tài nguyên chịu phí của AWS được xóa sạch sẽ.
* Hoàn thành xác thực không còn chi phí phát sinh bất ngờ.

### 4. Minh chứng thực hành (Evidence)
* **Ảnh kiểm tra S3 Raw & Processed đã trống rỗng (empty) và bị xóa:**
  *(Lưu ảnh vào `static/images/5-Workshop/5.10-clean-up/empty-buckets.png`)*
  ![Empty Buckets](/images/5-Workshop/5.10-clean-up/empty-buckets.png)

* **Ảnh Console hoặc Terminal output xác nhận xóa các tài nguyên:**
  *(Lưu ảnh vào `static/images/5-Workshop/5.10-clean-up/cleanup-resources.png`)*
  ![Cleanup Resources](/images/5-Workshop/5.10-clean-up/cleanup-resources.png)

* **Ảnh theo dõi chi phí hóa đơn (Billing/Budgets) sau dọn dẹp:**
  *(Lưu ảnh vào `static/images/5-Workshop/5.10-clean-up/billing-check.png`)*
  ![Billing Check](/images/5-Workshop/5.10-clean-up/billing-check.png)
