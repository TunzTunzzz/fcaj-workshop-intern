---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# DocuFlow AI 
### 1. Executive Summary
DocuFlow AI is an asynchronous serverless platform designed for automated invoice/receipt data extraction and normalization. Users securely authenticate via Amazon Cognito and upload documents (PDF, PNG, JPG) to an S3 Raw Bucket. Upon upload completion, S3 triggers an event via Amazon EventBridge, queueing the message in SQS. A Job Starter Lambda processes the queue messages and starts an AWS Step Functions Standard Workflow execution. The state machine coordinates a chain of Lambda functions: Validate (file inspection), Amazon Textract (raw OCR extraction via AnalyzeExpense API), AI Proxy (GPT-4o semantic normalization using Secrets Manager API keys), and Confidence Status (scoring and final classification). Normalization results are saved into S3 Processed and DynamoDB. If a document receives a low confidence score, it is flagged as `REVIEW_REQUIRED` and administrators are notified via Amazon SNS/SES.

### 2. Problem Statement
* **Inefficient Manual Ingestion**: Businesses frequently input invoice fields manually, leading to high human error rates, operational bottlenecks, and administrative delays as document volumes scale.
* **High Costs and Rigid Architecture**: Traditional OCR packages charge expensive enterprise licensing fees, require compute-intensive dedicated servers, and fail to map unstructured raw data to a standardized schema (e.g., dates, vendor names, currencies, line items).
* **The Solution**: DocuFlow AI resolves these issues by using a cost-efficient AWS Serverless model (pay-for-value) combined with Amazon Textract's powerful document OCR and an external Large Language Model's semantic parsing. Security is tightly managed using AWS Secrets Manager and IAM Least Privilege boundaries.

### 3. Solution Architecture
The system architecture uses an asynchronous event-driven design to ensure high scalability and zero request bottlenecks:

1. **Frontend Client**: Deployed on AWS Amplify/Vite. The client authenticates via Cognito, requests a Presigned URL from API Gateway, and uploads documents directly to the S3 Raw Bucket.
2. **Event Routing**: S3 Raw fires an ObjectCreated event to EventBridge, which is filtered and queued in SQS (backed by a Dead Letter Queue for fault tolerance).
3. **Orchestration Workflow**: Job Starter Lambda triggers the AWS Step Functions State Machine.
4. **AI Processing Pipeline**:
   * **Validate Lambda**: Checks document boundaries, file formats, and sizes.
   * **Textract Lambda**: Invokes Amazon Textract AnalyzeExpense API to extract unstructured text blocks.
   * **AI Proxy Lambda**: Fetches API credentials from Secrets Manager and queries GPT-4o to normalize fields into a unified schema.
   * **Confidence Status Lambda**: Evaluates completeness, writes the final `result.json` to S3 Processed, and stores metadata in the DynamoDB documents table under `EXTRACTED`, `REVIEW_REQUIRED`, or `FAILED` states.
5. **Observability & Alerts**: CloudWatch logs structured telemetry, X-Ray traces active requests, and SNS/SES emails alerts for system failures or documents requiring review.

### 4. AWS Services Used
* **Amazon Cognito**: Handles user authentication, registration, and secure access.
* **Amazon API Gateway**: Secures REST endpoints for Presigned URLs and queries.
* **Amazon S3**: Stores original raw invoices (Raw Bucket) and processed results (Processed Bucket).
* **Amazon EventBridge**: Triggers asynchronous processing pipelines based on file uploads.
* **Amazon SQS & DLQ**: Buffer message queues to throttle load and handle poison messages.
* **AWS Lambda**: Executes decoupled microservice business logic.
* **AWS Step Functions**: Orchestrates the sequential Lambda executions via a state machine.
* **Amazon Textract**: Processes layout analysis and expense OCR.
* **AWS Secrets Manager**: Secures External LLM API credentials.
* **Amazon DynamoDB**: Stores document metadata using single-table partitioning.
* **Amazon SNS & SES**: Alerts developers and reviewers of system states and review queues.
* **Amazon CloudWatch & AWS X-Ray**: Provide logs, alarms, metrics, and trace maps.
* **AWS Budgets**: Limits cost overrun risk.
* **AWS CloudTrail**: Logs administrative API calls for infrastructure compliance audits.
![image](/images/2-Proposal/image.png)