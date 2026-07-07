---
title: "5.7.2 Data Management Lambdas"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.7.2. </b> "
---

# CREATING PROCESSING FUNCTIONS (AWS LAMBDA)

What is AWS Lambda? Lambda is a service that lets you run code without provisioning or managing a traditional server. Our data module system requires four small Lambda functions to receive commands from users and read/write data to S3 and DynamoDB.

---

### STEP 3.1: Create Get Document Lambda

This function is responsible for: When a user clicks to view document details (API `GET /documents/{documentId}`), the function will fetch the detailed data and return it to them.

1. **Access the Lambda service**:
   * In the search bar at the top of the AWS Console, enter Lambda and select the **Lambda** service.
   ![image24.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image24.png)
2. **Initialize the function**:
   * Click the **Create function** button in the top right corner.
   * Select the **Author from scratch** option (Default).
   ![image25.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image25.png)

3. **Configure basic information**:
   * **Function name**: Enter the exact source code standard name: `docuflow-dev-data-get-document-lambda`. Or name it according to your requirements.
   * **Runtime**: Choose the standard programming language for the project (e.g., `Node.js 24.x`).
   ![image26.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image26.png)
   * **Architecture**: Select `arm64`.
4. **Configure Permissions**:
   * Scroll down and expand the **Additional settings** section.
   ![image27.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image27.png)
   * Under **Custom execution role**, select **Use an existing role** and choose the `docuflow-dev-data-lambda-role` created in the IAM section.
   
   ![image28.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image28.png)
   * Click **Save**.
   ![image29.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image29.png)
5. **Enter tags**:
   * Click on tags.
   * Select the necessary tags according to the setup (e.g., Project: DocuFlowAI).
   ![image30.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image30.png)
   * Click **Save**.
   ![image31.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image31.png)
6. Click the **Create function** button to create the lambda.
   ![image32.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image32.png)

Once the function is created, you will see the management interface. In the **Code source** section, you can paste the programming code below and click the **Deploy** button to save the code.
![image33.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image33.png)

**Source Code (`index.mjs`)**:
```javascript
import { DynamoDBClient, GetItemCommand } from "@aws-sdk/client-dynamodb";
import { S3Client, GetObjectCommand } from "@aws-sdk/client-s3";

const ddbClient = new DynamoDBClient({ region: "ap-southeast-1" });
const s3Client = new S3Client({ region: "ap-southeast-1" });

export const handler = async (event) => {
  console.log("Event:", JSON.stringify(event));
  try {
    const userId = event.requestContext?.authorizer?.claims?.sub || "user-001";
    const documentId = event.pathParameters?.documentId;
    const tableName = process.env.DOCUFLOW_DEV_TABLE_NAME;
    const processedBucket = process.env.DOCUFLOW_DEV_PROCESSED_BUCKET;
    
    // 1. Fetch summary metadata from DynamoDB
    const ddbResult = await ddbClient.send(new GetItemCommand({
      TableName: tableName,
      Key: {
        PK: { S: `USER#${userId}` },
        SK: { S: `DOC#${documentId}` }
      }
    }));
    
    if (!ddbResult.Item) {
      return {
        statusCode: 404,
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ error: "Document not found" })
      };
    }
    
    // 2. Download the detailed result file from the S3 Processed Bucket
    const s3Key = `processed/${userId}/${documentId}/result.json`;
    let s3Data = {};
    try {
      const s3Response = await s3Client.send(new GetObjectCommand({
        Bucket: processedBucket,
        Key: s3Key
      }));
      const bodyString = await s3Response.Body.transformToString();
      s3Data = JSON.parse(bodyString);
    } catch (s3Err) {
      console.warn("S3 result.json not found, falling back to DynamoDB:", s3Err);
    }
    
    // Reformat DynamoDB item
    const metadata = {};
    for (const [key, value] of Object.entries(ddbResult.Item)) {
      if (["PK", "SK", "GSI1PK", "GSI1SK"].includes(key)) continue;
      metadata[key] = Object.values(value)[0];
    }
    
    return {
      statusCode: 200,
      headers: {
        "Access-Control-Allow-Origin": "*",
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ ...metadata, ...s3Data })
    };
  } catch (err) {
    console.error(err);
    return {
      statusCode: 500,
      headers: { "Access-Control-Allow-Origin": "*" },
      body: JSON.stringify({ error: err.message })
    };
  }
};
```

---

### STEP 3.2: CREATE LIST DOCUMENTS LAMBDA

This function helps retrieve the list of all user documents or filter those with errors (API `GET /documents`).

1. Return to the main interface of the Lambda service (click on **Functions** in the left menu).
![image34.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image34.png)
2. Click the **Create function** button and choose **Author from scratch**.
   ![image35.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image35.png)

3. **Configure basic information**:
   * **Function name**: Enter exactly `docuflow-dev-data-list-documents-lambda`. Or name it according to your requirements.
   * **Runtime**: Choose the standard programming language for the project (`Node.js 24.x`).
   ![image36.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image36.png)
   * **Architecture**: Select `arm64`.
4. **Configure Permissions**:
   * Expand the **Additional settings** section.
   * Under **Custom execution role**, select **Use an existing role** and choose the `docuflow-dev-data-lambda-role`.
   ![image37.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image37.png)

5. **Enter tags**:
   * Click on tags, select the necessary tags according to the setup.
   ![image38.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image38.png)
   * Click **Save**.
   ![image39.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image39.png)
6. Click the **Create function** button.
   ![image40.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image40.png)
   
Once the function is created, in the Code source section, paste the programming code below and click **Deploy**.
   ![image41.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image41.png)

**Source Code (`index.mjs`)**:
```javascript
import { DynamoDBClient, QueryCommand } from "@aws-sdk/client-dynamodb";

const ddbClient = new DynamoDBClient({ region: "ap-southeast-1" });

export const handler = async (event) => {
  console.log("Event:", JSON.stringify(event));
  try {
    const userId = event.requestContext?.authorizer?.claims?.sub || "user-001";
    const status = event.queryStringParameters?.status;
    const tableName = process.env.DOCUFLOW_DEV_TABLE_NAME;
    
    let result;
    if (status) {
      // Query on Global Secondary Index (GSI) filtering by status
      result = await ddbClient.send(new QueryCommand({
        TableName: tableName,
        IndexName: "docuflow-dev-documents-status-createdAt-index",
        KeyConditionExpression: "GSI1PK = :gsi1pk",
        ExpressionAttributeValues: {
          ":gsi1pk": { S: `STATUS#${status}` }
        }
      }));
    } else {
      // Query on main table by user ID (PK)
      result = await ddbClient.send(new QueryCommand({
        TableName: tableName,
        KeyConditionExpression: "PK = :pk",
        ExpressionAttributeValues: {
          ":pk": { S: `USER#${userId}` }
        }
      }));
    }
    
    const items = result.Items.map(item => {
      const clean = {};
      for (const [key, val] of Object.entries(item)) {
        if (["PK", "SK", "GSI1PK", "GSI1SK"].includes(key)) continue;
        clean[key] = Object.values(val)[0];
      }
      return clean;
    });
    
    return {
      statusCode: 200,
      headers: {
        "Access-Control-Allow-Origin": "*",
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ items })
    };
  } catch (err) {
    console.error(err);
    return {
      statusCode: 500,
      headers: { "Access-Control-Allow-Origin": "*" },
      body: JSON.stringify({ error: err.message })
    };
  }
};
```

---

### STEP 3.3: CREATE REVIEW UPDATE LAMBDA

This function is used to save the information that the user manually edited on the interface (e.g., AI recognized the wrong amount and the user corrected it).

1. Click **Create function** ➔ **Author from scratch**.
   ![image42.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image42.png)

2. **Configure basic information**:
   * **Function name**: Enter exactly `docuflow-dev-data-review-update-lambda`.
   * **Runtime**: Choose the standard programming language for the project (`Node.js 24.x`).
   ![image43.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image43.png)
   * **Architecture**: Select `arm64`.
3. **Configure Permissions**:
   * Expand the **Additional settings** section.
   * Under **Custom execution role**, select **Use an existing role** and choose the `docuflow-dev-data-lambda-role`.
   ![image44.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image44.png)

4. **Add tags**:
   * Click on tags, select the necessary tags according to the setup.
   ![image45.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image45.png)
   * Click **Save**.
   ![image46.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image46.png)
5. Click the **Create function** button.
   ![image47.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image47.png)
6. This function is used to save the information that the user manually edited on the interface (e.g., AI recognized the wrong amount and the user corrected it)
![image48.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image48.png)

**Source Code (`index.mjs`)**:
```javascript
import { DynamoDBClient, UpdateItemCommand, GetItemCommand } from "@aws-sdk/client-dynamodb";
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";

const ddbClient = new DynamoDBClient({ region: "ap-southeast-1" });
const s3Client = new S3Client({ region: "ap-southeast-1" });

export const handler = async (event) => {
  console.log("Event:", JSON.stringify(event));
  try {
    const userId = event.requestContext?.authorizer?.claims?.sub || "user-001";
    const documentId = event.pathParameters?.documentId;
    const body = JSON.parse(event.body);
    const { corrections } = body; 
    
    const tableName = process.env.DOCUFLOW_DEV_TABLE_NAME;
    const processedBucket = process.env.DOCUFLOW_DEV_PROCESSED_BUCKET;
    
    // 1. Check if document exists
    const ddbItem = await ddbClient.send(new GetItemCommand({
      TableName: tableName,
      Key: {
        PK: { S: `USER#${userId}` },
        SK: { S: `DOC#${documentId}` }
      }
    }));
    
    if (!ddbItem.Item) {
      return {
        statusCode: 404,
        headers: { "Access-Control-Allow-Origin": "*" },
        body: JSON.stringify({ error: "Document not found" })
      };
    }
    
    // 2. Update status to APPROVED on DynamoDB
    await ddbClient.send(new UpdateItemCommand({
      TableName: tableName,
      Key: {
        PK: { S: `USER#${userId}` },
        SK: { S: `DOC#${documentId}` }
      },
      UpdateExpression: "SET #status = :status, GSI1PK = :gsi1pk, reviewedAt = :time, reviewedBy = :user",
      ExpressionAttributeNames: { "#status": "status" },
      ExpressionAttributeValues: {
        ":status": { S: "APPROVED" },
        ":gsi1pk": { S: "STATUS#APPROVED" },
        ":time": { S: new Date().toISOString() },
        ":user": { S: userId }
      }
    }));
    
    // 3. Overwrite the new modified result.json file on S3
    const s3Key = `processed/${userId}/${documentId}/result.json`;
    const finalizedResult = {
      documentId,
      userId,
      status: "APPROVED",
      corrections,
      updatedAt: new Date().toISOString()
    };
    
    await s3Client.send(new PutObjectCommand({
      Bucket: processedBucket,
      Key: s3Key,
      Body: JSON.stringify(finalizedResult),
      ContentType: "application/json"
    }));
    
    return {
      statusCode: 200,
      headers: {
        "Access-Control-Allow-Origin": "*",
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ message: "Review submitted successfully" })
    };
  } catch (err) {
    console.error(err);
    return {
      statusCode: 500,
      headers: { "Access-Control-Allow-Origin": "*" },
      body: JSON.stringify({ error: err.message })
    };
  }
};
```

---

### STEP 3.4: Create Delete Document Lambda

This function is used to delete the metadata of one or more user documents in DynamoDB and their associated files on S3 Raw & Processed.

1. Click **Create function** ➔ **Author from scratch**.
![image49.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image49.png)
2. **Configure basic information**:
   * **Function name**: Enter exactly `docuflow-dev-data-delete-lambda`.
   * **Runtime**: Choose the standard programming language for the project (`Node.js 24.x`).
   ![image50.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image50.png)
   * **Architecture**: Select `arm64`.
3. **Configure Permissions**:
   * Expand the **Additional settings** section.
   * Under **Custom execution role**, select **Use an existing role** and choose the `docuflow-dev-data-lambda-role`.
   
   ![image51.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image51.png)

4. **Add tags**:
   * Click on tags, select the necessary tags according to the setup.
    ![image52.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image52.png)
   * Click **Save**.
   ![image53.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image53.png)
5. Click the **Create function** button.
   ![image54.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image54.png)

6. This function is used to delete the data of 1 or more user documents that the user wants to delete.
   ![image55.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image55.png)

**Source Code (`index.mjs`)**:
```javascript
import { DynamoDBClient, DeleteItemCommand } from "@aws-sdk/client-dynamodb";
import { S3Client, DeleteObjectCommand } from "@aws-sdk/client-s3";

const ddbClient = new DynamoDBClient({ region: "ap-southeast-1" });
const s3Client = new S3Client({ region: "ap-southeast-1" });

export const handler = async (event) => {
  console.log("Event:", JSON.stringify(event));
  try {
    const userId = event.requestContext?.authorizer?.claims?.sub || "user-001";
    const documentId = event.pathParameters?.documentId;
    
    const tableName = process.env.DOCUFLOW_DEV_TABLE_NAME;
    const rawBucket = process.env.DOCUFLOW_DEV_RAW_BUCKET;
    const processedBucket = process.env.DOCUFLOW_DEV_PROCESSED_BUCKET;
    
    // 1. Delete record in DynamoDB
    await ddbClient.send(new DeleteItemCommand({
      TableName: tableName,
      Key: {
        PK: { S: `USER#${userId}` },
        SK: { S: `DOC#${documentId}` }
      }
    }));
    
    // 2. Delete original file in S3 Raw
    try {
      await s3Client.send(new DeleteObjectCommand({
        Bucket: rawBucket,
        Key: `raw/${userId}/${documentId}/original.pdf`
      }));
    } catch (e) { console.warn("Raw file not found in S3"); }
    
    // 3. Delete result file in S3 Processed
    try {
      await s3Client.send(new DeleteObjectCommand({
        Bucket: processedBucket,
        Key: `processed/${userId}/${documentId}/result.json`
      }));
    } catch (e) { console.warn("Processed file not found in S3"); }
    
    return {
      statusCode: 200,
      headers: {
        "Access-Control-Allow-Origin": "*",
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ message: "Document deleted successfully" })
    };
  } catch (err) {
    console.error(err);
    return {
      statusCode: 500,
      headers: { "Access-Control-Allow-Origin": "*" },
      body: JSON.stringify({ error: err.message })
    };
  }
};
```

---

### STEP 3.5: CONFIGURE ENVIRONMENT VARIABLES & TIMEOUT

Our code needs to know what the Table name and Bucket name are to connect. In addition, reading files sometimes takes time, so we need to increase the default wait time so the function doesn't get interrupted.

Perform the following steps consecutively for **all 4 Lambda functions** you just created:

1. At the details interface of each created Lambda function, switch to the **Configuration** tab.
 ![image56.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image56.png)
2. Select the **General configuration** section in the left vertical menu, click the **Edit** button.
![image57.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image57.png)
3. Change the **Timeout** from 3 sec (Default) to **10 seconds** (or 15 seconds) to ensure the function does not get interrupted midway when querying large data or reading files from S3.
![image58.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image58.png)
Click **Save**.
![image59.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image59.png)
4. Select the **Environment variables** section in the left vertical menu:
![image60.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image60.png)
   * Click the **Edit** button, then select **Add environment variable**.
   * **First variable pair**:
     * **Key**: `DOCUFLOW_DEV_TABLE_NAME`
     * **Value**: `docuflow-dev-documents-table`
   * **Second variable pair**:
     * **Key**: `DOCUFLOW_DEV_PROCESSED_BUCKET`
     * **Value**: Enter exactly the Processed S3 bucket name (e.g., `docuflow-dev-processed-<AWS_ACCOUNT_ID>-ap-southeast-1`)
   * **Third variable pair**:
     * **Key**: `DOCUFLOW_DEV_RAW_BUCKET`
     * **Value**: Enter exactly the Raw S3 bucket name (e.g., `docuflow-dev-raw-<AWS_ACCOUNT_ID>-ap-southeast-1`)

![image61.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image61.png)

5. Click **Save** to apply.
![image62.png](/images/5-Workshop/5.7-store-result-review-flow/5.7.2-lambdas/image62.png)

After completing this step for all 4 functions, your framework is ready to receive the logic processing source code and can be configured to integrate straight into the system's API Gateway.

---


