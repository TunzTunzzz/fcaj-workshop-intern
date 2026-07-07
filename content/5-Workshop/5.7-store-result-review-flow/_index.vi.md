---
title: "Lưu trữ kết quả & Xây dựng luồng phê duyệt"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Lưu trữ kết quả & Xây dựng luồng phê duyệt

### 1. Mục tiêu (Goal)
Thiết lập cơ chế lưu trữ bền vững (Data Persistence) theo mô hình "Offloading" (phân tách dữ liệu) giữa DynamoDB và S3, triển khai cụm 4 hàm Lambda xử lý nghiệp vụ Data (Get, List, Update, Delete) làm API phục vụ giao diện người dùng, đồng thời hỗ trợ luồng phê duyệt thủ công (Human-in-the-loop).

---

### 2. Các nội dung thực hành chi tiết

Vui lòng bấm chọn từng mục dưới đây để thực hiện chi tiết từng bước:

* **[5.7.1 Mô hình dữ liệu & Offloading](5.7.1-persistence/)**: Tìm hiểu thiết kế Single-Table Design trên DynamoDB và cơ chế phân tách Metadata / Payload chi tiết để tối ưu hiệu năng.
* **[5.7.2 Các hàm Lambda quản lý dữ liệu](5.7.2-lambdas/)**: Khởi tạo và triển khai mã nguồn Node.js SDK v3 cho 4 hàm Lambda (Get, List, Review Update, Delete).
* **[5.7.3 Cấu hình API Gateway](5.7.3-api-gateway/)**: Thiết lập Amazon API Gateway làm cổng kết nối cho các Lambda và cấu hình xác thực Cognito Authorizer.
* **[5.7.4 Thử nghiệm & Báo cáo](5.7.4-testing/)**: Hướng dẫn tạo dữ liệu mẫu (Mock data), kiểm thử các hàm Lambda API và chạy script báo cáo chi phí/số liệu thông qua AWS CloudShell.
