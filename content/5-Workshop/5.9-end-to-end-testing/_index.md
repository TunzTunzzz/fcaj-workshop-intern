---
title: "End-to-End Testing"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

# End-to-End Testing

### 1. Goal
Execute a complete integration test of the system from the frontend interface down to the background processing pipelines, verifying all distinct logic paths to collect testing evidence.

### 2. Test Scenarios

#### Scenario 1: Happy Path (Clear Invoice Document Upload)
* **Simulation steps**: Log in to the Frontend app using your Cognito credentials and upload a clear PDF/image invoice document.
* **System workflow**:
  1. The Frontend requests an S3 Presigned URL from the `upload-url` Lambda function.
  2. The Frontend uploads the raw document directly to the S3 Raw Bucket.
  3. S3 triggers the ObjectCreated notification -> forwarded to EventBridge -> SQS main queue receives the message.
  4. The Job Starter Lambda function polls the SQS message and starts a Step Functions execution.
  5. The Validate Lambda function verifies file size and type.
  6. The Textract Lambda function calls the Amazon Textract `AnalyzeExpense` API to get raw fields.
  7. The AI Proxy Lambda function normalizes the fields into JSON format.
  8. The Confidence & Status Lambda function calculates a confidence score above 90% (e.g. `0.99`). The system updates DynamoDB status to `EXTRACTED` and saves `result.json` in the S3 Processed Bucket.
  9. The Frontend Dashboard refreshes and displays the extracted values.
* **Evidence**: Take a screenshot of the document details page on the Frontend showing the status `EXTRACTED` alongside the fields table.

  *(Save the Happy Path screenshot to `static/images/5-Workshop/5.9-end-to-end-testing/happy-path.png`)*
  ![Happy Path](/images/5-Workshop/5.9-end-to-end-testing/happy-path.png)

#### Scenario 2: Human-in-the-loop Flow (Blurry Invoice / Low Confidence)
* **Simulation steps**: Upload a blurry receipt or an invoice missing critical fields like the total amount (totalAmount).
* **System workflow**:
  1. The document is uploaded and processed through the pipeline steps.
  2. During the confidence check, the Confidence & Status Lambda detects that the average confidence score is < 90% or missing critical fields.
  3. The Lambda function updates the DynamoDB item status to `REVIEW_REQUIRED` and sets the review status to `PENDING`.
  4. A CloudWatch Alarm fires on status changes, publishing an email alert to the Admin via SNS/SES.
  5. The Admin opens the Reviewer Dashboard on the Frontend, examining the review form containing the low-confidence fields.
  6. The Admin corrects the values and clicks **Approve**. The Frontend sends a PATCH request to the `review-update` Lambda, changing DynamoDB status to `APPROVED` and overwriting the corrected `result.json` in S3.
* **Evidence**: Take a screenshot of the Reviewer Form on the Frontend showing corrected fields and the alert notification email received.

  *(Save the review flow screenshot to `static/images/5-Workshop/5.9-end-to-end-testing/review-required-path.png`)*
  ![Review Path](/images/5-Workshop/5.9-end-to-end-testing/review-required-path.png)

#### Scenario 3: Validation Failure (Invalid File Format)
* **Simulation steps**: Upload a plain text `.txt` file (or a file larger than 10MB).
* **System workflow**:
  1. The file is uploaded to the S3 Raw Bucket and starts a Step Functions execution.
  2. In the first state, the Validate Lambda function checks file metadata.
  3. The validation check fails (e.g. invalid file format). The Lambda updates the DynamoDB item status to `FAILED` and records the reason (e.g. `INVALID_FILE_TYPE`).
  4. The Lambda function throws an error, and the Step Functions workflow catches the exception and transitions to the **Failed state**.
  5. An SNS email notification is dispatched to notify the Admin.
* **Evidence**: Take a screenshot of the red Failed node in the Step Functions execution history and the DynamoDB item showing the status `FAILED`.

  *(Save the failed path screenshot to `static/images/5-Workshop/5.9-end-to-end-testing/failed-path.png`)*
  ![Failed Path](/images/5-Workshop/5.9-end-to-end-testing/failed-path.png)

#### Scenario 4: Dead Letter Queue (SQS DLQ Routing)
* **Simulation steps**: Temporarily remove read permissions for the AWS Secrets Manager secret from the AI Proxy Lambda role to simulate persistent backend API call errors.
* **System workflow**:
  1. Upload a new document. SQS receives the notification and triggers the Job Starter Lambda to start Step Functions.
  2. The workflow fails at the AI Proxy step due to `AccessDeniedException`.
  3. The message returns to SQS and Lambda retries processing.
  4. After exceeding the maximum retry threshold (`Maximum receives = 3`), SQS routes the poison message to the Dead Letter Queue `docuflow-dev-processing-dlq`.
  5. The CloudWatch Alarm `docuflow-dev-sqs-dlq-not-empty-alarm` fires, alerting the Admin via SNS email.
* **Evidence**: Take a screenshot of the SQS console showing messages visible in the DLQ.

  *(Save the SQS DLQ screenshot to `static/images/5-Workshop/5.9-end-to-end-testing/dlq-messages.png`)*
  ![DLQ Messages](/images/5-Workshop/5.9-end-to-end-testing/dlq-messages.png)

### 3. Expected Result
* The asynchronous processing pipeline behaves correctly across all four test scenarios.
* Automated alerting dispatches warnings on failures and review-required triggers.
* S3 and DynamoDB maintain consistent data mapping.
