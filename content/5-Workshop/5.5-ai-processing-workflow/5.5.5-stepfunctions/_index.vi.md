---
title: "5.5.5 Step Functions Workflow"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5.5. </b> "
---

# Thiết lập AWS Step Functions State Machine

Chúng ta sẽ sử dụng dịch vụ AWS Step Functions để thiết kế luồng xử lý tự động (orchestration flow) kết nối tuần tự 4 hàm Lambda vừa tạo ở các bước trên.

---

### Các bước thực hiện trên AWS Console

1. **Truy cập dịch vụ Step Functions**:
   * Tại thanh tìm kiếm trên cùng của AWS Console, gõ **Step Functions** ➔ Chọn dịch vụ **Step Functions**.
   * Ở menu dọc bên trái, bấm chọn **State machines** ➔ Nhấn nút **Create state machine**.

2. **Cấu hình mã định nghĩa**:
   * Chọn **Write in JSON** (ASL Editor - Amazon States Language) để dán code định nghĩa luồng.
   * Xóa đoạn JSON mặc định và dán đoạn ASL JSON chuẩn dưới đây vào (Lưu ý: Thay thế các phần `603199863187` bằng **AWS Account ID** thực tế của bạn):

```json
{
  "Comment": "DocuFlow AI Invoice Processing Workflow",
  "StartAt": "ValidateDocument",
  "States": {
    "ValidateDocument": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-southeast-1:603199863187:function:docuflow-dev-ingestion-validate-lambda",
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "Next": "FailedWorkflow"
        }
      ],
      "Next": "ExtractTextract"
    },
    "ExtractTextract": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-southeast-1:603199863187:function:docuflow-dev-ai-textract-lambda",
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "Next": "FailedWorkflow"
        }
      ],
      "Next": "AIProxyNormalization"
    },
    "AIProxyNormalization": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-southeast-1:603199863187:function:docuflow-dev-ai-proxy-lambda",
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "Next": "FailedWorkflow"
        }
      ],
      "Next": "CalculateConfidenceAndStatus"
    },
    "CalculateConfidenceAndStatus": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-southeast-1:603199863187:function:docuflow-dev-ai-confidence-status-lambda",
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "Next": "FailedWorkflow"
        }
      ],
      "Next": "FinishedWorkflow"
    },
    "FinishedWorkflow": {
      "Type": "Succeed"
    },
    "FailedWorkflow": {
      "Type": "Fail",
      "Cause": "Invoice processing workflow failed",
      "Error": "WorkflowFailed"
    }
  }
}
```

3. **Cấu hình Quyền hạn (Permissions) và Hoàn tất**:
   * Nhấn **Next**.
   * **State machine name**: Đặt tên chuẩn `docuflow-dev-invoice-processing-workflow`.
   * **Permissions**: Chọn **Choose an existing role** ➔ Trỏ đến `docuflow-dev-workflow-stepfunctions-role` đã tạo ở bước IAM.
   * Nhấn **Create state machine**.

4. **Đồng bộ biến môi trường cho Job Starter Lambda**:
   * Sao chép **ARN** của State Machine vừa tạo (Ví dụ: `arn:aws:states:ap-southeast-1:603199863187:stateMachine:docuflow-dev-invoice-processing-workflow`).
   * Quay lại cấu hình hàm Lambda `docuflow-dev-ingestion-job-starter-lambda`.
   * Vào tab **Configuration** ➔ **Environment variables** ➔ Bấm **Edit** ➔ Cập nhật giá trị biến `STATE_MACHINE_ARN` bằng chuỗi ARN của State Machine bạn vừa sao chép. Nhấn **Save**.

---

### Minh chứng thực hành (Evidence)
*(Lưu ảnh sơ đồ State Machine Graph thiết kế thành công vào `static/images/5-Workshop/5.6-ai-processing-workflow/5.6.5-stepfunctions/stepfunctions-graph.png`)*
![Step Functions Graph](/images/5-Workshop/5.6-ai-processing-workflow/5.6.5-stepfunctions/stepfunctions-graph.png)
