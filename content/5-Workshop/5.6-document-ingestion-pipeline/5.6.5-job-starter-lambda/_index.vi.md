---
title: "5.6.5 Khởi tạo hàm Job Starter Lambda"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.6.5. </b> "
---

# Khởi tạo hàm Job Starter Lambda

#### Lab 8: Khởi tạo hàm Job Starter Lambda (`docuflow-dev-ingestion-job-starter-lambda`)
1. Truy cập **Lambda** -> nhấn **Create function** -> **Author from scratch**.
2. **Function name**: `docuflow-dev-ingestion-job-starter-lambda`.
3. **Runtime**: Chọn `Node.js 18.x` hoặc cao hơn.
4. **Role**: Chọn **Use an existing role** -> chọn role `docuflow-dev-security-job-starter-role`.
5. Bấm **Create function**.
6. Tại tab **Configuration** -> **General configuration**: Điều chỉnh Timeout thành **30 seconds**.
7. Tại tab **Configuration** -> **Environment variables**: Thêm biến:
   * `STATE_MACHINE_ARN` = `<ARN_CỦA_STEP_FUNCTIONS_STATE_MACHINE_ĐÃ_TẠO_Ở_TRÊN>`
8. Tại tab **Code**, copy đoạn code sau, dán đè vào `index.mjs` và nhấn **Deploy**:
   ```javascript
   import { SFNClient, StartExecutionCommand } from "@aws-sdk/client-sfn";

   const sfnClient = new SFNClient({ region: "ap-southeast-1" });

   export const handler = async (event) => {
     console.log("SQS Event:", JSON.stringify(event));
     try {
       const stateMachineArn = process.env.STATE_MACHINE_ARN;
       
       for (const record of event.Records) {
         // Parse body tin nhắn từ SQS (chứa event chuyển tiếp của EventBridge)
         const body = JSON.parse(record.body);
         console.log("Record Body:", JSON.stringify(body));
         
         const s3Bucket = body.detail.bucket.name;
         const s3Key = body.detail.object.key;
         
         // Tách userId và documentId từ S3 key: raw/{userId}/{documentId}/original.pdf
         const parts = s3Key.split("/");
         const userId = parts[1];
         const documentId = parts[2];
         
         const input = {
           rawS3Bucket: s3Bucket,
           rawS3Key: s3Key,
           userId: userId,
           documentId: documentId
         };
         
         console.log("Starting Step Functions execution with input:", JSON.stringify(input));
         
         await sfnClient.send(new StartExecutionCommand({
           stateMachineArn: stateMachineArn,
           name: `${documentId}-${Date.now()}`,
           input: JSON.stringify(input)
         }));
       }
       
       return { status: "SUCCESS" };
     } catch (err) {
       console.error("Error in Job Starter:", err);
       throw err; // Quăng lỗi để SQS không xóa tin nhắn và thực hiện retry
     }
   };
   ```

#### Lab 9: Cấu hình Triggers SQS cho Job Starter Lambda
1. Quay lại giao diện con Lambda `docuflow-dev-ingestion-job-starter-lambda`.
2. Tại sơ đồ **Function overview**, nhấn nút **Add trigger**.
3. **Select a source**: Chọn **SQS**.
4. **SQS queue**: Chọn hàng đợi chính `docuflow-dev-ingestion-processing-queue`.
5. **Batch size**: Điền `1` (Xử lý từng tệp tin độc lập).
6. Nhấn **Add** để hoàn tất liên kết.

### Kết quả mong đợi (Expected Result)
* S3 Event Notifications đã định tuyến chuẩn xác sang EventBridge.
* EventBridge Rule bắt đúng các tệp tin trong thư mục `raw/` và đẩy thành công vào SQS.
* Job Starter Lambda tự động kích hoạt (trigger) khi có tin nhắn SQS và ra lệnh khởi chạy Step Functions workflow skeleton thành công.
