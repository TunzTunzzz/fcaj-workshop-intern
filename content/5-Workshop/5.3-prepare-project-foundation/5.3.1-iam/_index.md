---
title: "5.3.1 Configure IAM Roles"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

# Configure IAM Policies and Roles for DocuFlow AI System

To adhere to the Principle of Least Privilege, we will design and create specialized IAM Policies and Roles for each Lambda and Step Functions service within the system.

---

### - Role 1: `docuflow-dev-security-upload-url-role` (Lambda to generate upload URL)
**Purpose**: Allows API Gateway to invoke Lambda to generate a Presigned URL so the Frontend can upload files to the S3 Raw bucket and record initial information into the database.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (Select the **JSON** tab).
  2. Paste the following JSON Policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowPutObjectToRawBucket",
                 "Effect": "Allow",
                 "Action": "s3:PutObject",
                 "Resource": "arn:aws:s3:::docuflow-dev-raw-*"
             },
             {
                 "Sid": "AllowWriteToDynamoDB",
                 "Effect": "Allow",
                 "Action": [
                     "dynamodb:PutItem",
                     "dynamodb:UpdateItem"
                 ],
                 "Resource": "arn:aws:dynamodb:*:*:table/docuflow-dev-documents-table"
             }
         ]
     }
     ```
     ![image1.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image1.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-ingestion-s3-raw-access-policy`.


  4. Enter Description: `Allows Lambda function to generate S3 presigned URLs for raw document upload and initialize metadata in DynamoDB table.`
     ![image2.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image2.png)
  5. Click **Create policy**.
  6. Go to **Roles** ➔ Choose **Create role** (Select Trusted Entity Type: **AWS service** ➔ Service or use case: **Lambda**).
  ![image3.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image3.png)
  7. Click **Next**.
  8. Attach Policy: Check `docuflow-dev-ingestion-s3-raw-access-policy` and the default system policy `AWSLambdaBasicExecutionRole` (for logging).
  ![image4.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image4.png)
  ![image5.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image5.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-security-upload-url-role`.
  ![image6.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image6.png)
  10. Click **Create role**.
  ![image7.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image7.png)
---

### - Role 2: `docuflow-dev-security-job-starter-role` (Lambda to read SQS and trigger Workflow)
**Purpose**: Allows Lambda to receive messages from the SQS queue to get the file ID, then trigger the automated processing workflow in Step Functions.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (Select the **JSON** tab).
  2. Paste the following JSON Policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowStartStepFunctions",
                 "Effect": "Allow",
                 "Action": "states:StartExecution",
                 "Resource": "arn:aws:states:*:*:stateMachine:docuflow-dev-*"
             },
             {
                 "Sid": "AllowManageSQSMessages",
                 "Effect": "Allow",
                 "Action": [
                     "sqs:ReceiveMessage",
                     "sqs:DeleteMessage",
                     "sqs:GetQueueAttributes"
                 ],
                 "Resource": "arn:aws:sqs:*:*:docuflow-dev-*"
             }
         ]
     }
     ```
     ![image8.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image8.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-workflow-trigger-policy`.
  4. Enter Description: `Grants permissions to process messages from SQS queue and trigger Step Functions workflow executions.`
     ![image9.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image9.png)
  5. Click **Create policy**.
    ![image10.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image10.png)
  6. Go to **Roles** ➔ Choose **Create role** (Select Trusted Entity Type: **AWS service** ➔ Service or use case: **Lambda**).
  ![image11.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image11.png)
  7. Click **Next**.
  8. Attach Policy: Check `docuflow-dev-workflow-trigger-policy` and `AWSLambdaBasicExecutionRole`.
  ![image12.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image12.png)
    ![image13.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image13.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-security-job-starter-role`.
  ![image14.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image14.png)

  10. Click **Create role**.
    ![image15.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image15.png)
---

### - Role 3: `docuflow-dev-workflow-stepfunctions-role` (Dedicated permissions for Step Functions Orchestrator)
**Purpose**: Grants Step Functions permissions to sequentially invoke processing Lambda functions and log execution progress.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (Select the **JSON** tab).
  2. Paste the following JSON Policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowStepFunctionToCallLambda",
                 "Effect": "Allow",
                 "Action": "lambda:InvokeFunction",
                 "Resource": "arn:aws:lambda:*:*:function:docuflow-dev-*"
             },
             {
                 "Sid": "AllowStepFunctionsLogging",
                 "Effect": "Allow",
                 "Action": [
                     "logs:CreateLogDelivery",
                     "logs:PutLogEvents",
                     "logs:GetLogDelivery",
                     "logs:UpdateLogDelivery",
                     "logs:DeleteLogDelivery",
                     "logs:ListLogDeliveries"
                 ],
                 "Resource": "*"
             }
         ]
     }
     ```
     ![image16.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image16.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-workflow-orchestration-policy`.
  4. Enter Description: `Allows Step Functions to orchestrate workflow by invoking core processing Lambdas and logging deliveries.`
     ![image17.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image17.png)
  5. Click **Create policy**.
  ![image18.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image18.png)
  6. Go to **Roles** ➔ Choose **Create role** (Select Trusted Entity Type: **Custom trust policy**).
  
  7. Paste the following Trust Policy defining the Step Functions service:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [{ "Effect": "Allow", "Principal": { "Service": "states.amazonaws.com" }, "Action": "sts:AssumeRole" }]
     }
     ```
     ![image19.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image19.png)
  8. Click **Next**. Attach Policy: Check `docuflow-dev-workflow-orchestration-policy`.
  ![image20.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image20.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-workflow-stepfunctions-role`.
  10. Enter Description: `Service role for Step Functions state machine to orchestrate the AI document pipeline.`
  ![image21.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image21.png)
  11. Click **Create role**.
  ![image22.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image22.png)

---

### - Role 4: `docuflow-dev-ai-textract-lambda-role` (Lambda permissions for Amazon Textract text extraction)
**Purpose**: Allows Lambda to retrieve files from the S3 Raw bucket, perform extraction using Textract, and save the raw results to the S3 Processed Bucket.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (JSON).
  2. Paste the following JSON Policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowReadRawBucket",
                 "Effect": "Allow",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::docuflow-dev-raw-*"
             },
             {
                 "Sid": "AllowWriteProcessedBucket",
                 "Effect": "Allow",
                 "Action": "s3:PutObject",
                 "Resource": "arn:aws:s3:::docuflow-dev-processed-*"
             },
             {
                 "Sid": "AllowCallTextractAI",
                 "Effect": "Allow",
                 "Action": "textract:AnalyzeExpense",
                 "Resource": "*"
             }
         ]
     }
     ```
     ![image23.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image23.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-ai-textract-invoke-policy`.
  4. Enter Description: `Allows Lambda to retrieve raw documents from S3, invoke Amazon Textract AnalyzeExpense API, and save results.`
     ![image24.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image24.png)
  5. Click **Create policy**.
    ![image25.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image25.png)
  6. Go to **Roles** ➔ Choose **Create role** (Trusted Entity Type: **AWS service** ➔ Service: **Lambda**).
  ![image26.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image26.png)
  7. Click **Next**.
  8. Attach Policy: Check `docuflow-dev-ai-textract-invoke-policy` along with `AWSLambdaBasicExecutionRole`, `AmazonS3ExpressFullAccess`, `AmazonTextractFullAccess`.
  ![image27.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image27.png)
    ![image28.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image28.png)
    ![image29.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image29.png)
    ![image30.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image30.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-ai-textract-lambda-role`.
  10. Enter Description: `Execution role for extraction lambda to read raw invoices, run Textract OCR, and write thô data.`
     ![image31.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image31.png)
  11. Click **Create role**.
  

---

### - Role 5: `docuflow-dev-security-ai-proxy-role` (Lambda permissions to call External AI outside AWS)
**Purpose**: The ultimate security checkpoint. Only this AI Proxy function is allowed to read the Token (API Key) stored in Secrets Manager to connect to external AI models.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (JSON).
  2. Paste the following JSON Policy (change the ARN accordingly if necessary):
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowAIProxyToGetSecret",
                 "Effect": "Allow",
                 "Action": "secretsmanager:GetSecretValue",
                 "Resource": "arn:aws:secretsmanager:ap-southeast-1:*:secret:docuflow-dev-external-ai-api-key-*"
             },
             {
                 "Sid": "AllowAIProxyToUseKMS",
                 "Effect": "Allow",
                 "Action": [
                     "kms:Decrypt",
                     "kms:DescribeKey"
                 ],
                 "Resource": "arn:aws:kms:*:*:alias/docuflow-dev-main-key"
             }
         ]
     }
     ```
     ![image32.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image32.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-ai-secret-read-policy`.
  4. Enter Description: `Strictly restricts access to External AI API credentials stored within AWS Secrets Manager.`
  ![image33.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image33.png)
  5. Click **Create policy**.
   ![image34.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image34.png)
  6. Go to **Roles** ➔ Choose **Create role** (Trusted Entity Type: **AWS service** ➔ Service: **Lambda**).
    ![image35.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image35.png)
  7. Click **Next**.
  8. Attach Policy: Check `docuflow-dev-ai-secret-read-policy` and `AWSLambdaBasicExecutionRole`.
  ![image36.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image36.png)
      ![image37.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image37.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-security-ai-proxy-role`.
     ![image38.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image38.png)
  10. Click **Create role**.
    ![image39.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image39.png)

---

### - Role 6: `docuflow-dev-data-lambda-role` (Lambda CRUD permissions for UI Display Data)
**Purpose**: Allows API Lambda functions to perform CRUD operations on DynamoDB and load files from S3 buckets to display on the Frontend.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (JSON).
  2. Paste the following JSON Policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowReadWriteDynamoDBOnly",
                 "Effect": "Allow",
                 "Action": [
                     "dynamodb:GetItem",
                     "dynamodb:Query",
                     "dynamodb:UpdateItem",
                     "dynamodb:Scan",
                     "dynamodb:DeleteItem"
                 ],
                 "Resource": [
                     "arn:aws:dynamodb:*:*:table/docuflow-dev-documents-table",
                     "arn:aws:dynamodb:*:*:table/docuflow-dev-documents-table/index/*"
                 ]
             },
             {
                 "Sid": "AllowReadS3ProcessedBucket",
                 "Effect": "Allow",
                 "Action": [
                     "s3:GetObject",
                     "s3:DeleteObject",
                     "s3:ListBucket"
                 ],
                 "Resource": [
                     "arn:aws:s3:::docuflow-dev-processed-*/*",
                     "arn:aws:s3:::docuflow-dev-raw-*/*",
                     "arn:aws:s3:::docuflow-dev-processed-*",
                     "arn:aws:s3:::docuflow-dev-raw-*"
                 ]
             }
         ]
     }
     ```
  ![image40.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image40.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-data-dynamodb-access-policy`.
  4. Enter Description: `Allows standard query, scanning, and metadata update operations against the core DynamoDB documents table.`
     ![image41.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image41.png)
  5. Click **Create policy**.
  ![image42.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image42.png)
  6. Go to **Roles** ➔ Choose **Create role** (Trusted Entity Type: **AWS service** ➔ Service: **Lambda**).
  ![image43.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image43.png)
  7. Click **Next**.
  8. Attach Policy: Check `docuflow-dev-data-dynamodb-access-policy` and `AWSLambdaBasicExecutionRole`.
  ![image44.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image44.png)
  ![image45.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image45.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-data-lambda-role`.
10. Enter Description: `Execution role for metadata/data lambda to fetch and modify documents inside DynamoDB.`
  ![image46.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image46.png)
  11. Click **Create role**.
  ![image47.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image47.png)

---

### - Role 7: `docuflow-dev-notification-lambda-role` (Lambda permissions to send Alert notifications)
**Purpose**: Pushes system error notifications or abnormal invoice alerts to an SNS Topic to trigger the automated email flow for the team.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (JSON).
  2. Paste the following JSON Policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowPublishToSNSTopic",
                 "Effect": "Allow",
                 "Action": "sns:Publish",
                 "Resource": "arn:aws:sns:*:*:docuflow-dev-notification-system-alerts-topic"
             }
         ]
     }
     ```
     ![image48.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image48.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-notification-sns-publish-policy`.
  4. Enter Description: `Enables system alerting components to publish failure logs and anomaly alerts to the target SNS Topic.`
     ![image49.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image49.png)
  5. Click **Create policy**.
  ![image50.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image50.png)
  6. Go to **Roles** ➔ Choose **Create role** (Trusted Entity Type: **AWS service** ➔ Service: **Lambda**).
  ![image51.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image51.png)
  7. Click **Next**.
  8. Attach Policy: Check `docuflow-dev-notification-sns-publish-policy` and `AWSLambdaBasicExecutionRole`.
  ![image52.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image52.png)
  ![image53.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image53.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-notification-lambda-role`.
  10. Enter Description: `Execution role for notification component to broadcast system alert messages via SNS.`
  ![image54.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image54.png)
  11. Click **Create role**.
  ![image55.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image55.png)

---

### - Role 8: `docuflow-dev-ingestion-validate-lambda-role` (Lambda permissions for input data validation)
**Purpose**: Responsible for validating file formats and sizes from the S3 Raw bucket, and recording the initial PROCESSING state in DynamoDB.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (JSON).
  2. Paste the following JSON Policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowReadS3RawMetadata",
                 "Effect": "Allow",
                 "Action": [
                     "s3:GetObject",
                     "s3:GetObjectAttributes"
                 ],
                 "Resource": "arn:aws:s3:::docuflow-dev-raw-*"
             },
             {
                 "Sid": "AllowUpdateStatusToDynamoDB",
                 "Effect": "Allow",
                 "Action": [
                     "dynamodb:UpdateItem",
                     "dynamodb:GetItem"
                 ],
                 "Resource": "arn:aws:dynamodb:*:*:table/docuflow-dev-documents-table"
             }
         ]
     }
     ```
     ![image56.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image56.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-ingestion-validation-policy`.
  4. Enter Description: `Allows Validate Lambda to check file metadata in S3 Raw and update initial processing state in DynamoDB.`
     ![image57.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image57.png)
  5. Click **Create policy**.
  ![image58.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image58.png)
  6. Go to **Roles** ➔ Choose **Create role** (Trusted Entity Type: **AWS service** ➔ Service: **Lambda**).
  ![image59.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image59.png)
  7. Click **Next**.
  8. Attach Policy: Check `docuflow-dev-ingestion-validation-policy` and `AWSLambdaBasicExecutionRole`.
  ![image60.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image60.png)
  ![image61.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image61.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-ingestion-validate-lambda-role`.
  10. Enter Description: `Execution role for validation lambda to inspect uploaded documents and update state.`
  ![image62.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image62.png)
  11. Click **Create role**.
  ![image63.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image63.png)

---

### - Role 9: `docuflow-dev-ai-confidence-status-lambda-role` (Lambda permissions for Status Logic & Confidence Evaluation)
**Purpose**: After the AI Proxy returns the results, the Confidence + Status Lambda evaluates the score, categorizes the status as EXTRACTED or REVIEW_REQUIRED, saves the final result (`result.json`) to the S3 Processed Bucket, and updates the DynamoDB table.
* **Steps**:
  1. Go to the **IAM** service ➔ **Policies** ➔ Choose **Create policy** (JSON).
  2. Paste the following JSON Policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowWriteToProcessedBucket",
                 "Effect": "Allow",
                 "Action": "s3:PutObject",
                 "Resource": "arn:aws:s3:::docuflow-dev-processed-*"
             },
             {
                 "Sid": "AllowWriteFinalStatusToDynamoDB",
                 "Effect": "Allow",
                 "Action": [
                     "dynamodb:UpdateItem",
                     "dynamodb:PutItem"
                 ],
                 "Resource": "arn:aws:dynamodb:*:*:table/docuflow-dev-documents-table"
             }
         ]
     }
     ```
     ![image64.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image64.png)
  3. Click **Next**. Set Policy Name: `docuflow-dev-ai-confidence-status-policy`.
  4. Enter Description: `Allows Lambda to calculate confidence scores, save final result JSON to S3 Processed, and update metadata in DynamoDB.`
     ![image65.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image65.png)
  5. Click **Create policy**.
  ![image66.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image66.png)
  6. Go to **Roles** ➔ Choose **Create role** (Trusted Entity Type: **AWS service** ➔ Service: **Lambda**).
  ![image67.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image67.png)
  7. Click **Next**.
  8. Attach Policy: Check `docuflow-dev-ai-confidence-status-policy` and `AWSLambdaBasicExecutionRole`.
  ![image68.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image68.png)
  ![image69.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image69.png)
  9. Click **Next**. Set Role Name: `docuflow-dev-ai-confidence-status-lambda-role`.
  10. Enter Description: `Execution role for confidence and status lambda to evaluate AI result and update final state.`
  ![image70.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image70.png)
  11. Click **Create role**.
  ![image71.png](/images/5-Workshop/5.3-prepare-project-foundation/5.3.1-iam/image71.png)
