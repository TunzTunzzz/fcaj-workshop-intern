---
title: "5.5.4 Confidence Status Lambda"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

# Confidence & Status Lambda Function (`docuflow-dev-ai-confidence-status-lambda`)

This function is responsible for checking data integrity (Schema completeness), evaluating the extraction confidence (Confidence Score), and determining the final status of the document (`EXTRACTED` or `REVIEW_REQUIRED`).

---

### Step-by-Step Console Configuration

1. **Create the Lambda function**:
   * Go to **Lambda** ➔ Click **Create function**.
   
   ![image1.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image1.png)
   
   * Choose **Author from scratch** (default).
   ![image2.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image2.png)
   * **Function name**: Enter `docuflow-dev-ai-confidence-status-lambda`.
   
   ![image3.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image3.png)
   
   * **Runtime**: Select `Node.js 24.x`.
   ![image4.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image4.png)
   * Expand **Additional settings** and under **Architecture** select `arm64`.
   ![image30.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image30.png)
   * Under **Permissions** ➔ **Change default execution role**, choose **Use an existing role** and select `docuflow-dev-ai-confidence-status-lambda-role`.
   ![image31.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image31.png)
   * **Required - Add Tags**: Scroll down to the **Tags** tab and add the following standard tags for cost allocation and governance:
     * **Project**: DocuFlowAI
     * **Environment**: dev
     * **ManagedBy**: SAM
     * **Module**: ai
     * **CostCenter**: Workshop
     * **Owner**: Team
   ![image32.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image32.png)
   ![image33.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image33.png)
   * Click **Save** to save the tags.
   ![image34.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image34.png)
   * Click **Create function**.
![image11.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image11.png)

2. **Configure General settings**:
   * Go to the **Configuration** tab ➔ **General configuration** ➔ Click **Edit**.
   ![image35.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image35.png)
   * Change **Timeout** to **30 seconds** and click **Save**.
   ![image36.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image36.png)

3. **Deploy Code**:
   * In the **Code** tab, paste the following Node.js code over the boilerplate code in the `index.mjs` editor.
   * Click **Deploy**.

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

4. **Test the Lambda function**:
   * Switch to the **Test** tab.
   ![image37.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image37.png)
   * Select **Create new event**.
   ![image38.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image38.png)
   * Select **Synchronous**.
   ![image39.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image39.png)
   * Enter `Test` in the **Event name** field.
   ![image40.png](/images/5-Workshop/5.5-ai-processing-workflow/5.5.4-confidence/image40.png)
   * In the **Event JSON** block, paste the test output result from the previous `docuflow-dev-ai-proxy-lambda` execution.
   * Click **Save** and then click **Test**. If the execution succeeds and returns output similar to the following JSON, the Lambda is configured correctly:
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
