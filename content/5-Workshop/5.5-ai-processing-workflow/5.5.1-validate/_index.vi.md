---
title: "5.5.1 Validate Lambda"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

# Hàm Validate Lambda (`docuflow-dev-ingestion-validate-lambda`)

Hàm này chịu trách nhiệm kiểm tra định dạng và kích thước tệp tin tải lên S3 Raw Bucket để đảm bảo tệp tin hợp lệ trước khi bắt đầu bóc tách.

---

### Các bước cấu hình trên AWS Console

1. **Khởi tạo hàm Lambda**:
   * Truy cập dịch vụ **Lambda** ➔ Nhấn **Create function**.
   

   * Chọn **Author from scratch** (Mặc định).
   * **Function name**: Điền `docuflow-dev-ingestion-validate-lambda`.
   * **Runtime**: Chọn `Node.js 18.x` hoặc cao hơn.
   * **Architecture**: Chọn `arm64` để tối ưu chi phí và tăng tốc độ xử lý.
   * **Permissions**: Mở rộng **Change default execution role** ➔ Chọn **Use an existing role** ➔ Trỏ đến `docuflow-dev-ingestion-validate-lambda-role`.
   * Nhấn **Create function**.


2. **Cấu hình tham số vận hành**:
   * Chuyển sang tab **Configuration** ➔ **General configuration** ➔ Bấm **Edit**.
   

   

   

   

   

   

   * Đổi **Timeout** thành **30 seconds**. Nhấn **Save**.
   * Chọn tiếp mục **Environment variables** (Biến môi trường) ➔ Bấm **Edit**.
   * Thêm biến môi trường:
     * `DOCUFLOW_DEV_TABLE_NAME` = `docuflow-dev-documents-table`
   * Nhấn **Save**.


3. **Triển khai Code**:
   * Chuyển sang tab **Code**, copy đoạn mã Node.js v18 dưới đây và dán đè lên code cũ trong editor `index.mjs` của AWS Console.
   * Bấm nút **Deploy** để lưu lại.


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
    
    // 1. Lấy thông tin metadata file từ S3 Raw để kiểm duyệt kích thước
    const s3Meta = await s3Client.send(new HeadObjectCommand({
      Bucket: rawS3Bucket,
      Key: rawS3Key
    }));
    
    const fileSize = s3Meta.ContentLength;
    const contentType = s3Meta.ContentType;
    
    console.log(`File info: size=${fileSize}, type=${contentType}`);
    
    // Các luật kiểm duyệt: File tối đa 10MB và định dạng cho phép PDF/PNG/JPEG
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
      // Cập nhật trạng thái FAILED trong DynamoDB
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
    
    // Trả về dữ liệu gốc để chuyển tiếp sang bước tiếp theo
    return event;
  } catch (err) {
    console.error("Validation error:", err);
    throw err;
  }
};
```
