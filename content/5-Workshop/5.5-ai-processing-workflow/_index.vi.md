---
title: "Dựng Workflow xử lý AI"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Dựng Workflow xử lý AI

### 1. Mục tiêu (Goal)
Thiết lập luồng xử lý AI bất đồng bộ sử dụng **AWS Step Functions** để điều phối tuần tự 4 hàm Lambda:
1. **Kiểm tra tệp tin hợp lệ (Validate)**
2. **Gọi dịch vụ OCR bóc tách trường thô (Amazon Textract)**
3. **Chuẩn hóa cấu trúc dữ liệu JSON bằng mô hình ngôn ngữ lớn (AI Proxy)**
4. **Tự động tính toán điểm tin cậy để định tuyến trạng thái tài liệu (Confidence & Status)**

---

### 2. Các nội dung thực hành chi tiết

Vui lòng bấm chọn từng mục dưới đây để thực hiện chi tiết từng bước:

* **[5.5.1 Validate Lambda](5.6.1-validate/)**: Khởi tạo và lập trình hàm Lambda kiểm tra định dạng và kích thước tệp tin đầu vào.
* **[5.5.2 Textract Lambda](5.6.2-textract/)**: Cấu hình hàm Lambda gọi dịch vụ OCR bóc tách trường thô bằng Amazon Textract.
* **[5.5.3 AI Proxy Lambda](5.6.3-proxy/)**: Cấu hình hàm Lambda chuẩn hóa dữ liệu bóc tách bằng mô hình ngôn ngữ lớn OpenAI GPT-4o.
* **[5.5.4 Confidence Status Lambda](5.6.4-confidence/)**: Cấu hình hàm Lambda phân định độ tin cậy và tự động gán trạng thái tài liệu.
* **[5.5.5 Step Functions Workflow](5.6.5-stepfunctions/)**: Xây dựng State Machine liên kết tuần tự và xử lý lỗi cho toàn bộ 4 hàm Lambda trên.
