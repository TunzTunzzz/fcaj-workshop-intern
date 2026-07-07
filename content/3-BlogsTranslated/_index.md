---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---


###  [Blog 1 - Building a scalable user search layer on top of Amazon Cognito](3.1-Blog1/)
This post introduces a practical approach to upgrading the user search experience within the AWS ecosystem, specifically by adding robust query capabilities to the Amazon Cognito service. By seamlessly integrating powerful Serverless tools such as Lambda, DynamoDB, and OpenSearch Service, this architecture not only facilitates automatic data synchronization but also delivers flexible search options (including fuzzy matching and multi-criteria filtering). Most importantly, the system guarantees near-instantaneous response times and scales autonomously based on real-time traffic.

###  [Blog 2 - Building a Smart Retry Mechanism for Serverless Queue Consumer on AWS](3.2-Blog2/)
This architectural guide offers a detailed blueprint for enhancing the reliability and fault tolerance of Serverless applications. The core focus is on the effective combination of AWS Lambda, Amazon SQS, and EventBridge Scheduler to gracefully manage transient errors and bottlenecks from downstream services. The standout feature of this solution is its ability to completely decouple the retry logic from the main business logic. This separation allows for precise scheduling of retries and integrates smoothly with Dead Letter Queues, thereby eliminating the risk of data loss.

###  [Blog 3 - How ALS GeoAnalytics LITHOLENS™ Revolutionizes Drill Core Logging Using Machine Learning on Amazon EKS](3.3-Blog3/)
This is a prime case study demonstrating the application of Machine Learning alongside the flexible computing platform of Amazon EKS to drive digital transformation in heavy industry. Specifically, through the LITHOLENS™ system, ALS GeoAnalytics has entirely replaced the traditionally manual, costly, and time-consuming process of drill core logging. Shifting to this automated model has vastly improved data accuracy. Furthermore, leveraging AWS’s Containerized and Serverless architecture has allowed the enterprise to slash operational costs and seamlessly scale this solution across other engineering domains.

###  [Blog 4 - BUILDING A MULTI-ACCOUNT PATCH COMPLIANCE DASHBOARD WITH KIRO SPECS](3.4-Blog4/)
This article delves into the Kiro Specs methodology for establishing a centralized patch compliance dashboard tailored for multi-account AWS environments. At the heart of the solution is the extraction of raw data via AWS Systems Manager Patch Manager, which is then stored in Amazon S3 and processed by Lambda functions. Ultimately, this processed data is displayed on a secure internal dashboard protected by an Application Load Balancer and Session Manager. By following the three stages of Kiro Specs (Requirements -> Design -> Tasks), the post vividly illustrates how AI can be leveraged to generate a Serverless architecture that is both highly efficient and strictly compliant with security standards.