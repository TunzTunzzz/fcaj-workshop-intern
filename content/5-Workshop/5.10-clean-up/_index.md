---
title: "Clean Up"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

# Clean Up

### 1. Goal
Clean up and delete all deployed AWS resources after the workshop to avoid unwanted charges and verify solid project cost control.

### 2. Steps

#### Step 1: Empty and Delete Amazon S3 Buckets
AWS does not allow S3 buckets to be deleted if they contain objects. We must empty them first:
1. In the **S3 Console**, complete the following for all three buckets:
   * `docuflow-dev-raw-<AWS_ACCOUNT_ID>-ap-southeast-1`
   * `docuflow-dev-processed-<AWS_ACCOUNT_ID>-ap-southeast-1`
   * `docuflow-dev-cloudtrail-logs-<RANDOM_STRING>`
2. For each bucket, select the checkbox next to its name -> Click **Empty** -> Type `permanently delete` to confirm.
3. Once empty, select the bucket -> Click **Delete** -> Type the name of the bucket to delete it permanently.

#### Step 2: Delete DynamoDB Documents Table
1. Go to **DynamoDB** -> select **Tables** from the left sidebar.
2. Select the main project table: `docuflow-dev-documents-table`.
3. Click the **Delete** button in the top menu -> Type `confirm` in the confirmation dialog and click **Delete**.

#### Step 3: Delete Backend Lambda Functions
1. Go to **Lambda** -> In the **Functions** tab, search for `docuflow-dev-` to list all project functions.
2. Select all of these functions at once (includes API Data, AI Lambdas, Job Starter, and Validate).
3. Click **Actions** -> Select **Delete** -> Type `confirm` and click **Delete**.

#### Step 4: Delete the Connected API Gateway
1. Go to the **API Gateway Console**.
2. Select the project API: `docuflow-dev-api`.
3. Click the **Actions** dropdown in the top menu -> Select **Delete API** -> Confirm the action.

#### Step 5: Delete CloudWatch Alarms
1. Go to **CloudWatch** -> Choose **Alarms** -> **All alarms** in the left sidebar.
2. Select the four alarms created for the project:
   * `docuflow-dev-workflow-failed-alarm`
   * `docuflow-dev-sqs-dlq-not-empty-alarm`
   * `docuflow-dev-lambda-ai-proxy-error-alarm`
   * `docuflow-dev-low-confidence-spike-alarm`
3. Click **Actions** -> Select **Delete** -> click **Delete** to confirm.

#### Step 6: Delete CloudWatch Log Groups
1. Still in CloudWatch, choose **Logs** -> **Log groups**.
2. Select the six log groups associated with your Lambda functions:
   * `/aws/lambda/docuflow-dev-api-upload-url-lambda`
   * `/aws/lambda/docuflow-dev-ingestion-job-starter-lambda`
   * `/aws/lambda/docuflow-dev-ingestion-validate-lambda`
   * `/aws/lambda/docuflow-dev-ai-textract-lambda`
   * `/aws/lambda/docuflow-dev-ai-proxy-lambda`
   * `/aws/lambda/docuflow-dev-ai-confidence-status-lambda`
3. Click **Actions** -> Select **Delete log group(s)** -> Confirm the action.

#### Step 7: Delete SNS Topics & SES Verified Identities
1. **SNS Topic**: Go to **Simple Notification Service (SNS)** -> Select **Topics** -> select `docuflow-dev-notification-system-alerts-topic` -> Click **Delete** -> Type `delete me` to confirm.
2. **SES Email Identity**: Go to **Simple Email Service (SES)** -> Select **Verified identities** -> select your verified email address -> Click **Delete** -> Confirm the deletion.

#### Step 8: Stop & Delete CloudTrail Audit Logs
1. Go to **CloudTrail** -> Select **Trails** on the left menu.
2. Click on the project trail: `docuflow-dev-audit-trail`.
3. Click **Stop logging** -> then click **Delete** to remove the trail.

#### Step 9: Delete AWS Budgets cost alerts
1. Search for **Budgets** in the AWS console -> Select **AWS Budgets**.
2. Select the three budgets:
   * `DocuFlow-Monthly-Actual-Cost`
   * `DocuFlow-Credit-Safety`
   * `DocuFlow-FreeTier-Watch`
3. Click **Actions** -> Select **Delete** -> Confirm the deletion.

#### Step 10: Delete external AI credentials in Secrets Manager
1. Go to **Secrets Manager** -> select the secret: `docuflow-dev-external-ai-api-key`.
2. Click **Actions** -> Select **Delete secret**.
3. Set the waiting period to the minimum of **7 days** to disable the secret immediately and avoid future billing.

#### Step 11: Remove IAM Roles & Policies
1. Go to **IAM** -> Select **Roles** in the left sidebar.
2. Search and select the nine roles created in Step 3:
   * `docuflow-dev-security-upload-url-role`
   * `docuflow-dev-security-job-starter-role`
   * `docuflow-dev-workflow-stepfunctions-role`
   * `docuflow-dev-ai-textract-lambda-role`
   * `docuflow-dev-security-ai-proxy-role`
   * `docuflow-dev-data-lambda-role`
   * `docuflow-dev-notification-lambda-role`
   * `docuflow-dev-ingestion-validate-lambda-role`
   * `docuflow-dev-ai-confidence-status-lambda-role`
3. Click **Delete** and type the role names to confirm.
4. Go to **Policies** -> Search and delete all custom policies (e.g. `docuflow-dev-ingestion-s3-raw-access-policy`, `docuflow-dev-ai-secret-read-policy`, etc.).

#### Step 12: Schedule KMS Key Deletion (Must be performed last)
1. Go to **Key Management Service (KMS)** -> select **Customer managed keys**.
2. Select the CMK key: `docuflow-dev-main-key`.
3. Click **Key actions** -> Select **Schedule key deletion**.
4. Set the waiting period to **7 days** (the minimum allowed waiting period by AWS).
5. Click the orange confirmation button. The key state will transition to `Pending deletion` and maintenance fees will freeze.

### 3. Expected Result
* All billable AWS resources are deleted.
* Verification steps ensure no lingering charges accumulate post-workshop.

### 4. Evidence
* **S3 Buckets verified empty and deleted:**
  *(Save the screenshot to `static/images/5-Workshop/5.10-clean-up/empty-buckets.png`)*
  ![Empty Buckets](/images/5-Workshop/5.10-clean-up/empty-buckets.png)

* **AWS Console confirmation of deleted stack components:**
  *(Save the screenshot to `static/images/5-Workshop/5.10-clean-up/cleanup-resources.png`)*
  ![Cleanup Resources](/images/5-Workshop/5.10-clean-up/cleanup-resources.png)

* **AWS Billing Dashboard verified zero cost accumulation:**
  *(Save the screenshot to `static/images/5-Workshop/5.10-clean-up/billing-check.png`)*
  ![Billing Check](/images/5-Workshop/5.10-clean-up/billing-check.png)
