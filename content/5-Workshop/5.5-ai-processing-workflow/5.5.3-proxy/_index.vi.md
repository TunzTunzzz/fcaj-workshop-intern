---
title: "5.5.3 AI Proxy Lambda"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

# Hàm AI Proxy Lambda (`docuflow-dev-ai-proxy-lambda`)

Hàm này chịu trách nhiệm lấy API Key bảo mật từ AWS Secrets Manager, gửi yêu cầu tới mô hình ngôn ngữ lớn (OpenAI GPT-4o) để chuẩn hóa cấu trúc dữ liệu JSON từ các trường thô trích xuất bởi Textract, đồng thời bảo toàn điểm tin cậy (Confidence score).

---

### Các bước cấu hình trên AWS Console

1. **Khởi tạo hàm Lambda**:
   * Truy cập dịch vụ **Lambda** ➔ Nhấn nút **Create a function**.
   ![image1.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image1.png)
   * Chọn **Author from scratch**.
   ![image2.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image2.png)
   * Ở ô **Function name**: Nhập `docuflow-dev-ai-proxy-lambda` hoặc đặt tên bạn thích.
   
   ![image3.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image3.png)
   
   * **Runtime**: Chọn `Node.js 24.x`.

   ![image4.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image4.png)
   
   * Mở rộng **Additional settings** kéo xuống chọn **Architecture**: `arm64`.
   ![image5.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image5.png)
   * Kế bên ARM64 architecture chọn Execution role: Chọn **Use an existing role** và trỏ đến `docuflow-dev-security-ai-proxy-role` (hoặc role proxy mà bạn đã tạo ở IAM).
   ![image19.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image19.png)
   * Gắn Tag (không bắt buộc): Kéo xuống tab **Tags**, thêm các tag chuẩn sau để quản lý chi phí và phân quyền: 
     * **Project**: DocuFlowAI
     * **Environment**: dev
     * **ManagedBy**: SAM
     * **Module**: ai
     * **CostCenter**: Workshop
     * **Owner**: Team
   ![image20.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image20.png)
   ![image21.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image21.png)
   ![image22.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image22.png)
   * Nhấn **Save** để lưu lại các Tags vừa tạo.
   ![image10.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image10.png)
   * Kéo xuống nhấn nút **Create function** để tạo Lambda.
  ![image11.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image11.png)

2. **Cấu hình tham số vận hành**:
   * Sau khi tạo xong function chọn cái function vừa tạo rồi chuyển sang tab **Configuration** chọn **General configuration** và nhấn nút **Edit**.
   ![image23.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image23.png)
   * Sau đó để **Timeout** là **1 min 10 sec** và nhấn **Save**.
   ![image24.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image24.png)
3. **Triển khai Code**:
   * Chuyển sang lại tab **Code** bạn hãy copy toàn bộ đoạn code dưới đây, dán đè lên code cũ trong AWS Console và nhấn **Deploy**:

```javascript
import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager";
import https from 'https';

// Khởi tạo client Secrets Manager
const secretsClient = new SecretsManagerClient({ region: "ap-southeast-1" });

// Từ điển chuẩn hóa field (Màng lọc chống rác Textract)
const validSummaryFields = {
  "VENDOR_NAME": "vendorName",
  "vendorName": "vendorName",
  "INVOICE_RECEIPT_DATE": "invoiceDate",
  "invoiceDate": "invoiceDate",
  "INVOICE_RECEIPT_ID": "invoiceNumber",
  "invoiceNumber": "invoiceNumber",
  "DUE_DATE": "dueDate",
  "dueDate": "dueDate",
  "SUBTOTAL": "subtotalAmount",
  "subtotalAmount": "subtotalAmount",
  "TAX": "taxAmount",
  "taxAmount": "taxAmount",
  "TOTAL": "totalAmount",
  "totalAmount": "totalAmount",
  "CURRENCY": "currency",
  "currency": "currency"
};

export const handler = async (event) => {
  console.log("Received data from Textract Lambda:", JSON.stringify(event, null, 2));

  try {
    // ==========================================
    // 1. LẤY API KEY TỪ AWS SECRETS MANAGER
    // ==========================================
    const secretResponse = await secretsClient.send(
      new GetSecretValueCommand({ 
        SecretId: "docuflow-dev-external-ai-api-key" 
      })
    );
    
    const secret = JSON.parse(secretResponse.SecretString);
    const apiKey = secret.api_key; 

    if (!apiKey) {
      throw new Error("API Key not found in secret.");
    }

    const textractData = event.extractedData || {};

    // ==========================================
    // 2. CẬP NHẬT SYSTEM PROMPT
    // ==========================================
    const postData = JSON.stringify({
      model: "gpt-4o", 
      response_format: { type: "json_object" },
      messages: [
        {
          role: "system",
          content: `Bạn là trợ lý AI chuyên trích xuất hóa đơn. Trả về định dạng JSON thuần. Bạn phải tuân thủ CHÍNH XÁC cấu trúc sau, giữ đúng định dạng camelCase, kết thúc bằng chữ 'Amount' cho các trường tiền tệ, và tuyệt đối không thêm field lạ:
{
  "invoice": {
    "vendorName": "Tên nhà cung cấp",
    "invoiceNumber": "Số hóa đơn",
    "invoiceDate": "Ngày hóa đơn (YYYY-MM-DD)",
    "dueDate": "Ngày đến hạn (YYYY-MM-DD)",
    "currency": "Mã tiền tệ (VD: VND, USD, EUR)",
    "subtotalAmount": 100.00,
    "taxAmount": 19.00,
    "totalAmount": 119.00
  },
  "lineItems": [
    {
      "description": "Tên mặt hàng",
      "quantity": 1,
      "unitPriceAmount": 100.00,
      "taxAmount": 19.00,
      "totalAmount": 119.00
    }
  ]
}
Lưu ý: Nếu không có mặt hàng nào, hãy trả về "lineItems": []`
        },
        {
          role: "user",
          content: `Hãy chuẩn hóa toàn bộ dữ liệu sau: ${JSON.stringify(textractData)}`
        }
      ]
    });

    // ==========================================
    // 3. GỌI API EXTERNAL AI VÀ PARSE KẾT QUẢ
    // ==========================================
    const normalizedDataString = await callExternalAI(apiKey, postData);
    const openAiResult = JSON.parse(normalizedDataString);

    // ==========================================
    // 4. TỔNG HỢP CONFIDENCE VÀ LỌC FIELD RÁC
    // ==========================================
    const fieldConfidence = {};
    let totalScore = 0;
    let fieldCount = 0;

    if (textractData.summaryFields) {
      for (const [key, fieldData] of Object.entries(textractData.summaryFields)) {
        // Ánh xạ sang tên chuẩn (VD: TOTAL -> totalAmount)
        const docuflowKey = validSummaryFields[key];
        
        // Nếu là field lạ (VD: ADDRESS_BLOCK, OTHER) thì bỏ qua luôn 
        if (!docuflowKey) continue; 

        // Sử dụng Nullish Coalescing (??) để lấy fallback an toàn
        const rawScore = fieldData.confidenceScore ?? fieldData.confidence;
        
        if (rawScore !== undefined) {
          // CỰC KỲ QUAN TRỌNG: Ép về thang điểm 0-1 nếu Textract trả về 0-100 
          const finalScore = rawScore > 1 ? rawScore / 100 : rawScore;
          
          fieldConfidence[docuflowKey] = parseFloat(finalScore.toFixed(4));
          totalScore += finalScore;
          fieldCount++;
        }
      }
    }
    
    // Tính điểm trung bình tổng
    const averageScore = fieldCount > 0 ? totalScore / fieldCount : 0;
    const confidenceObj = {
      confidenceScore: parseFloat(averageScore.toFixed(4)),
      hasLowConfidence: averageScore < 0.9,
      fieldConfidence // Shorthand property
    };

    // Chuẩn hóa Line Items: Bổ sung lineItemId và confidenceScore (0-1)
    const rawLineItems = openAiResult.lineItems || [];
    const finalizedLineItems = rawLineItems.map((item, index) => {
      let itemConfidence = 0.9; // Điểm mặc định
      
      // Sử dụng Optional Chaining (?.) tránh lỗi "Cannot read properties of undefined"
      const tItem = textractData.lineItems?.[index];
      
      if (tItem) {
        const scores = Object.values(tItem)
          .map(f => f.confidenceScore ?? f.confidence)
          .filter(s => s !== undefined);
          
        if (scores.length > 0) {
          const avg = scores.reduce((a, b) => a + b, 0) / scores.length;
          // Ép về thang 0-1
          itemConfidence = avg > 1 ? avg / 100 : avg;
        }
      }

      return {
        lineItemId: `item-${String(index + 1).padStart(3, '0')}`,
        ...item,
        confidenceScore: parseFloat(itemConfidence.toFixed(4))
      };
    });

    // ==========================================
    // 5. TRẢ VỀ KẾT QUẢ ĐÃ LẮP GHÉP (BẢO TOÀN ROOT FIELDS)
    // ==========================================
    // Tách extractedData (dữ liệu thô) ra khỏi event để lấy metadata hệ thống
    const { extractedData, ...originalRootFields } = event;

    return {
      ...originalRootFields,         // Phục hồi nguyên vẹn các trường root bắt buộc 
      status: "EXTRACTED",           // Cập nhật trạng thái chuẩn
      invoice: openAiResult.invoice || {},
      lineItems: finalizedLineItems, // Mảng Line items đã đủ ID và điểm
      confidence: confidenceObj      // Object confidence chuẩn 0-1 và sạch rác
    };

  } catch (error) {
    console.error("Error in AI Proxy:", error);
    throw error; 
  }
};

// ==========================================
// HÀM HỖ TRỢ GỌI API HTTPS (GIỮ NGUYÊN)
// ==========================================
async function callExternalAI(apiKey, postData) {
  return new Promise((resolve, reject) => {
    const options = {
      hostname: "api.openai.com", 
      path: "/v1/chat/completions",
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey}`,
        'Content-Length': Buffer.byteLength(postData)
      }
    };

    const req = https.request(options, (res) => {
      let body = '';
      res.on('data', (chunk) => body += chunk);
      res.on('end', () => {
        if (res.statusCode !== 200) {
          reject(new Error(`AI API returned status ${res.statusCode}: ${body}`));
          return;
        }
        try {
          const response = JSON.parse(body);
          resolve(response.choices[0].message.content);
        } catch (e) {
          reject(new Error("Failed to parse AI response as JSON"));
        }
      });
    });

    req.on('error', (err) => reject(err));
    req.write(postData);
    req.end();
  });
}
```

4. **Kiểm thử hàm Lambda**:
   * Sau khi deploy thành công thì chuyển sang tab **Test**.
   ![image25.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image25.png)
   * Chọn **Create new event**.
   ![image26.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image26.png)
   * Sau đó chọn tiếp **Synchronous**.
   ![image27.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image27.png)
   * Ở chỗ **Event name** đặt tên mà bạn muốn (ví dụ: `Test`).
   ![image28.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image28.png)  
   * Ở chỗ **Event JSON** sao chép kết quả test đầu ra của `docuflow-dev-ai-textract-lambda` ở bài trước và dán vào.
   * Cuối cùng nhấn **Save** sau đó nhấn **Test**. Nếu kết quả hiện giống với đoạn code phía dưới là thành công:
   ![image29.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.3-proxy/image29.png)
     ```json
     {
       "status": "EXTRACTED",
       "invoice": {
         "vendorName": "East Repair Inc.",
         "invoiceNumber": "US-001",
         "invoiceDate": "2019-02-11",
         "dueDate": "2019-02-26",
         "currency": "USD",
         "subtotalAmount": 145,
         "taxAmount": 9.06,
         "totalAmount": 154.06
       },
       "lineItems": [
         {
           "lineItemId": "item-001",
           "description": "Front and rear brake cables",
           "quantity": 1,
           "unitPriceAmount": 100,
           "taxAmount": 0,
           "totalAmount": 100,
           "confidenceScore": 0.9996
         },
         {
           "lineItemId": "item-002",
           "description": "New set of pedal arms",
           "quantity": 2,
           "unitPriceAmount": 15,
           "taxAmount": 0,
           "totalAmount": 30,
           "confidenceScore": 0.9998
         },
         {
           "lineItemId": "item-003",
           "description": "Labor 3hrs",
           "quantity": 3,
           "unitPriceAmount": 5,
           "taxAmount": 0,
           "totalAmount": 15,
           "confidenceScore": 0.9999
         }
       ],
       "confidence": {
         "confidenceScore": 0.9976,
         "hasLowConfidence": false,
         "fieldConfidence": {
           "dueDate": 0.9999,
           "invoiceDate": 0.9997,
           "invoiceNumber": 0.9965,
           "subtotalAmount": 0.9999,
           "taxAmount": 0.9991,
           "totalAmount": 1,
           "vendorName": 0.988
         }
       }
     }
     ```
