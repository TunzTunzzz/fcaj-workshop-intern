---
title: "5.5.5 Step Functions Workflow"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5.5. </b> "
---

# Deploy AWS Step Functions State Machine

We will use AWS Step Functions to create the automated orchestration workflow that executes our four Lambda functions in sequence.

---

### Step-by-Step Console Configuration

1. **Access the Step Functions Dashboard**:
   * Search for **Step Functions** in the AWS Console search bar ➔ Select **Step Functions**.
   * In the left menu, choose **State machines** ➔ click **Create state machine**.

2. **Configure Definition Code**:
   * Select **Write in JSON** (ASL Editor - Amazon States Language) to paste the workflow structure.
   * Clear the default JSON and paste this ASL JSON (Note: Replace `603199863187` with your actual **AWS Account ID**):

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

3. **Configure Permissions and Complete Deployment**:
   * Click **Next**.
   * **State machine name**: Enter `docuflow-dev-invoice-processing-workflow`.
   * **Permissions**: Choose **Choose an existing role** ➔ select `docuflow-dev-workflow-stepfunctions-role` created in IAM steps.
   * Click **Create state machine**.

4. **Sync the State Machine ARN with your Job Starter Lambda**:
   * Copy the **ARN** of your new State Machine (e.g. `arn:aws:states:ap-southeast-1:603199863187:stateMachine:docuflow-dev-invoice-processing-workflow`).
   * Return to your `docuflow-dev-ingestion-job-starter-lambda` configuration.
   * Open **Configuration** ➔ **Environment variables** ➔ Click **Edit** ➔ update `STATE_MACHINE_ARN` with your copied State Machine ARN. Click **Save**.

---

### Evidence
*(Save the State Machine graph screenshot to `static/images/5-Workshop/5.6-ai-processing-workflow/5.6.5-stepfunctions/stepfunctions-graph.png`)*
![Step Functions Graph](/images/5-Workshop/5.6-ai-processing-workflow/5.6.5-stepfunctions/stepfunctions-graph.png)
