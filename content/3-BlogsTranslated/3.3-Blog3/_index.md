---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---
## How ALS GeoAnalytics LITHOLENS™ Revolutionizes Drill Core Logging Using Machine Learning on Amazon EKS

**Author:** Pham Tung Duong

**AWS Study Group Posting Date:** June 18, 2026

In the mining industry, accurate geological analysis is a crucial factor for improving mine design and development. Traditionally, this process required experts to manually inspect drill core samples on-site—often in remote and harsh areas—which was very time-consuming and labor-intensive.
To address this issue, ALS GeoAnalytics developed the LITHOLENS™ platform. This is a system applying Machine Learning (ML) and computer vision to automate the core logging process, helping to enhance data consistency, improve operational efficiency, and reduce costs as well as greenhouse gas emissions.

### Challenges in the Traditional Method
Building a 3D resource model for a new mine requires drilling thousands of holes to inspect the structure and composition of samples. This process faces multiple barriers such as:
* Difficult site access: Geologists must travel long distances to inspect physical sample boxes.
* Subjective assessments: Different experts often provide inconsistent geological logs.
* Forgotten historical data: Images from previous campaigns often lack standardization tools for meaningful analysis.
* Sample degradation: Physical samples can be lost or degraded over time, making it difficult to validate old data.

### Machine Learning at Geological Scale
LITHOLENS™ utilizes a comprehensive suite of ML and Deep Learning (DL) models to turn raw images into actionable insights:
* Color Extraction and Clustering: Uses algorithms like K-Means or GMM to identify mineralogical variations through color pixels.
* Percentage Reporting: Segments images into sections (e.g., 20cm intervals) to analyze the spatial distribution of lithological patterns.
* Specialized Deep Learning Models:
  * RoQE Net: An advanced neural network helping extract geotechnical parameters like Rock Quality Designation (RQD).
  * VeinNet and CobbleNet: Designed to identify complex geological features like mineral veins and lithological structures with high accuracy.

### Solution Architecture on AWS
ALS GeoAnalytics built a hybrid architecture combining containerized workloads and serverless components:

![Architecture Diagram](/images/3-BlogsTranslated/3.3-Blog3/architecture_blog3.png)

* Amazon EKS (Elastic Kubernetes Service): Handles heavy ML/DL tasks requiring GPU compute power (G6 instance types).
* AWS Lambda: Handles API operations and workload orchestration, helping reduce operational costs compared to maintaining always-on API servers.
* Amazon S3 and Amazon RDS: Used to store input data, intermediate results, and manage structured data.

Performance Highlights: By using pre-configured machine images (AMIs), container startup time has been reduced from several minutes to under 30 seconds. Additionally, EKS clusters can automatically scale down to zero when there are no jobs in the queue, maximizing cost optimization.

### Business Impact and Future Vision
To date, LITHOLENS™ has been successfully deployed across 10 mining companies with over 40 active projects. The system not only helps standardize the analysis process but also enables real-time monitoring and reporting, helping managers make faster and more accurate decisions.
The success of LITHOLENS™ on Amazon EKS doesn't stop at the mining industry. ALS GeoAnalytics is looking to expand this platform to sectors such as oil and gas, civil engineering, and even space exploration.

Original article link: [https://aws.amazon.com/vi/blogs/architecture/how-als-geoanalytics-litholens-revolutionizes-core-logging-through-machine-learning-with-amazon-eks/](https://aws.amazon.com/vi/blogs/architecture/how-als-geoanalytics-litholens-revolutionizes-core-logging-through-machine-learning-with-amazon-eks/)