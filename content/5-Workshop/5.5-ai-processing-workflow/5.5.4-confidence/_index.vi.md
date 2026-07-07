---
title: "5.5.4 Confidence Status Lambda"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

# Hàm Confidence & Status Lambda (`docuflow-dev-ai-confidence-status-lambda`)

Hàm này chịu trách nhiệm kiểm tra tính toàn vẹn dữ liệu (Schema completeness), đánh giá điểm tin cậy bóc tách (Confidence Score), và đưa ra quyết định trạng thái cuối cùng của tài liệu (`EXTRACTED` hoặc `REVIEW_REQUIRED`).

---

### Các bước cấu hình trên AWS Console

1. **Khởi tạo hàm Lambda**:
   * Truy cập dịch vụ **Lambda** ➔ Nhấn nút **Create function**.
   
   ![image1.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image1.png)
   
   * Chọn **Author from scratch** (Mặc định).
   ![image2.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image2.png)
   * Ở ô **Function name**: Nhập `docuflow-dev-ai-confidence-status-lambda` hoặc đặt tên bạn thích.
   
   ![image3.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image3.png)
   
   * **Runtime**: Chọn `Node.js 24.x`.
   ![image4.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image4.png)
   * Mở rộng **Additional settings** kéo xuống chọn **Architecture**: `arm64`.
   ![image30.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image30.png)
   * Kế bên ARM64 architecture chọn **Execution role**: Chọn **Use an existing role** và trỏ đến `docuflow-dev-ai-confidence-status-lambda-role` (hoặc role bạn đã tạo ở IAM).
   ![image31.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image31.png)
   * **Bắt buộc - Gắn Tag**: Kéo xuống tab **Tags**, thêm các tag chuẩn sau để quản lý chi phí và phân quyền: 
     * **Project**: DocuFlowAI
     * **Environment**: dev
     * **ManagedBy**: SAM
     * **Module**: ai
     * **CostCenter**: Workshop
     * **Owner**: Team
   ![image32.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image32.png)
   ![image33.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image33.png)
   * Nhấn **Save** để lưu lại các Tags vừa thêm.
   ![image34.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image34.png)
   * Nhấn nút **Create function** để tạo Lambda.
![image11.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image11.png)

2. **Cấu hình tham số vận hành**:
   * Sau khi tạo xong function chọn cái function vừa tạo rồi chuyển sang tab **Configuration** chọn **General configuration** và nhấn nút **Edit**.
   ![image35.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image35.png)
   * Ở chỗ **Timeout** để là **30 sec** sau đó nhấn **Save** để lưu lại.
   ![image36.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image36.png)

3. **Triển khai Code**:
   * Chuyển sang lại tab **Code** bạn hãy copy toàn bộ đoạn code dưới đây, dán đè lên code cũ trong AWS Console và nhấn **Deploy**:

```javascript
export const handler = async (event) => {
  console.log("Received event:", JSON.stringify(event, null, 2));

  // ==========================================
  // 1. TÌM ĐÚNG VỊ TRÍ DỮ LIỆU (Bao phủ mọi kịch bản của Step Functions)
  // ==========================================
  // Sử dụng Toán tử Nullish Coalescing (??) để fallback an toàn
  const invoice = 
    event.invoice ?? 
    event.normalizedResult?.invoice ?? 
    event.normalizedResult?.Payload?.invoice ?? 
    event.normalizedResult?.payload?.invoice ?? 
    {};

  const confidence = 
    event.confidence ?? 
    event.normalizedResult?.confidence ?? 
    event.normalizedResult?.Payload?.confidence ?? 
    event.normalizedResult?.payload?.confidence ?? 
    {};
  
  // Khởi tạo danh sách mã lý do cần kiểm duyệt
  const reviewReasonCodes = [];

  // ==========================================
  // 2. KIỂM TRA LỖI VÀ TÍNH ĐẦY ĐỦ
  // ==========================================
  if (!invoice.vendorName) reviewReasonCodes.push('MISSING_VENDOR_NAME');
  if (!invoice.invoiceDate) reviewReasonCodes.push('MISSING_INVOICE_DATE');
  
  // Sử dụng == null để bắt đồng thời cả null và undefined
  if (invoice.totalAmount == null) {
    reviewReasonCodes.push('MISSING_TOTAL_AMOUNT');
  } else if (typeof invoice.totalAmount !== 'number') {
    reviewReasonCodes.push('INVALID_AMOUNT_FORMAT');
  }
  
  if (!invoice.currency) reviewReasonCodes.push('MISSING_CURRENCY');

  // ==========================================
  // 3. TÍNH TOÁN VÀ CHUẨN HÓA CONFIDENCE SCORE
  // ==========================================
  let rawScore = confidence.confidenceScore;

  if (typeof rawScore !== 'number') {
    const fieldScores = Object.values(confidence.fieldConfidence || {});
    // Rút gọn khối if-else bằng toán tử ba ngôi (Ternary operator)
    rawScore = fieldScores.length > 0 
      ? fieldScores.reduce((a, b) => a + b, 0) / fieldScores.length 
      : 0;
  }

  // Đảm bảo giá trị nằm trong khoảng từ 0 đến 1
  let finalConfidenceScore = rawScore > 1 ? rawScore / 100 : rawScore;
  finalConfidenceScore = Math.max(0, Math.min(1, finalConfidenceScore));

  // Gán LOW_CONFIDENCE nếu điểm dưới 0.9 hoặc cờ hasLowConfidence đã bật
  if (finalConfidenceScore < 0.9 || confidence.hasLowConfidence) {
    if (!reviewReasonCodes.includes('LOW_CONFIDENCE')) {
      reviewReasonCodes.push('LOW_CONFIDENCE');
    }
  }

  // ==========================================
  // 4. QUYẾT ĐỊNH STATUS CUỐI CÙNG
  // ==========================================
  const isReviewRequired = reviewReasonCodes.length > 0;
  const status = isReviewRequired ? 'REVIEW_REQUIRED' : 'EXTRACTED';

  // ==========================================
  // 5. TRẢ VỀ KẾT QUẢ ĐÃ LẮP GHÉP
  // ==========================================
  return {
    ...event, // BẢO TOÀN TOÀN BỘ CÁC TRƯỜNG GỐC TỪ BƯỚC TRƯỚC
    status,   // Shorthand property (thay vì status: status)
    confidence: {
      confidenceScore: parseFloat(finalConfidenceScore.toFixed(4)),
      hasLowConfidence: reviewReasonCodes.includes('LOW_CONFIDENCE'),
      fieldConfidence: confidence.fieldConfidence || {}
    },
    review: {
      reviewStatus: isReviewRequired ? 'PENDING' : 'NOT_REQUIRED', 
      reviewReasonCodes, // Shorthand property
      reviewedBy: null,
      reviewedAt: null,
      corrections: []
    }
  };
};
```

4. **Kiểm thử hàm Lambda**:
   * Sau khi deploy thành công thì chuyển sang tab **Test**.
   ![image37.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image37.png)
   * Chọn **Create new event**.
   ![image38.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image38.png)
   * Sau đó chọn tiếp **Synchronous**.
   ![image39.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image39.png)
   * Ở chỗ **Event name** đặt tên mà bạn muốn (ví dụ: `Test`).
   ![image40.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image40.png)
   * Ở chỗ **Event JSON** sao chép kết quả test của `docuflow-dev-ai-proxy-lambda` sau đó xóa code cũ và bỏ vô.
   * Cuối cùng nhấn **Save** sau đó nhấn **Test**. Nếu kết quả hiện giống với đoạn code phía dưới là thành công:
   ![image41.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image41.png)
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
       },
       "review": {
         "reviewStatus": "NOT_REQUIRED",
         "reviewReasonCodes": [],
         "reviewedBy": null,
         "reviewedAt": null,
         "corrections": []
       }
     }
     ```
