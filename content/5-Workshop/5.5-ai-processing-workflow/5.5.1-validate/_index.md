---
title: "5.5.1 Validate Lambda"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

# Validate Lambda Function (`docuflow-dev-ingestion-validate-lambda`)

This Lambda function is responsible for checking file formatting and sizes on the S3 Raw Bucket to ensure only valid files propagate into the processing pipeline.

---

### Step-by-Step Console Configuration

1. **Create the Lambda function**:
   

   * Go to **Lambda** ➔ Click **Create function**.
   * Choose **Author from scratch** (default).
   * **Function name**: Enter `docuflow-dev-ingestion-validate-lambda`.
   * **Runtime**: Select `Node.js 18.x` or higher.
   * **Architecture**: Select `arm64` for cost efficiency.
   * **Permissions**: Expand **Change default execution role** ➔ Choose **Use an existing role** ➔ select `docuflow-dev-ingestion-validate-lambda-role`.
   * Click **Create function**.


2. **Configure General settings**:
   * Select **Configuration** tab ➔ **General configuration** ➔ Click **Edit**.
   

   

   

   

   

   

   * Change **Timeout** to **30 seconds**. Click **Save**.
   * Go to **Environment variables** ➔ Click **Edit**.
   * Add:
     * `DOCUFLOW_DEV_TABLE_NAME` = `docuflow-dev-documents-table`
   * Click **Save**.


3. **Deploy Code**:
   * In the **Code** tab, paste this Node.js v18 code over the boilerplate code in the `index.mjs` editor.
   * Click **Deploy**.


```javascript
import { S3Client, HeadObjectCommand } from "@aws-sdk/client-s3";
import { DynamoDBClient, UpdateItemCommand } from "@aws-sdk/client-dynamodb";

const s3Client = new S3Client({ region: "ap-southeast-1" });
const ddbClient = new DynamoDBClient({ region: "ap-southeast-1" });

export const handler = async (event) => {
  console.log("Validate Lambda Event:", JSON.stringify(event));
  try {
    const { rawS3Bucket, rawS3Key, userId, documentId } = event;
    const tableName = process.env.DOCUFLOW_DEV_TABLE_NAME;
    
    // 1. Fetch file metadata from S3
    const s3Meta = await s3Client.send(new HeadObjectCommand({
      Bucket: rawS3Bucket,
      Key: rawS3Key
    }));
    
    const fileSize = s3Meta.ContentLength;
    const contentType = s3Meta.ContentType;
    
    console.log(`File info: size=${fileSize}, type=${contentType}`);
    
    // File validation: max size 10MB and PDF/PNG/JPEG formats
    const maxSizeBytes = 10 * 1024 * 1024;
    const allowedTypes = ["application/pdf", "image/png", "image/jpeg"];
    
    let is_valid = true;
    let reason = "";
    
    if (fileSize > maxSizeBytes) {
      is_valid = false;
      reason = "FILE_TOO_LARGE";
    } else if (!allowedTypes.includes(contentType)) {
      is_valid = false;
      reason = "INVALID_FILE_TYPE";
    }
    
    if (!is_valid) {
      // Update DynamoDB status to FAILED
      await ddbClient.send(new UpdateItemCommand({
        TableName: tableName,
        Key: {
          PK: { S: `USER#${userId}` },
          SK: { S: `DOC#${documentId}` }
        },
        UpdateExpression: "SET #status = :status, GSI1PK = :gsi1pk, failReason = :reason",
        ExpressionAttributeNames: {
          "#status": "status"
        },
        ExpressionAttributeValues: {
          ":status": { S: "FAILED" },
          ":gsi1pk": { S: "STATUS#FAILED" },
          ":reason": { S: reason }
        }
      }));
      throw new Error(`Validation failed: ${reason}`);
    }
    
    // Pass event variables down to the next Step Functions workflow state
    return event;
  } catch (err) {
    console.error("Validation error:", err);
    throw err;
  }
};
```
