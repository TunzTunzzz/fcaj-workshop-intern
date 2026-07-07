---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
## Building a scalable user search layer on top of Amazon Cognito

**Posted by:** Vu Duy Tai

**AWS Study Group Posting Date:** June 5, 2026

If your team only needs simple search over standard Amazon Cognito attributes, the ListUsers API is sufficient. However, when the system has to handle advanced requirements such as custom attribute search, fuzzy matching, complex filtering, and sub-second response times, building a dedicated search layer is the appropriate choice.
This article discusses how to build a scalable and advanced user search layer on top of Amazon Cognito, using a combination of AWS services including AWS Lambda, Amazon DynamoDB, and Amazon OpenSearch Service, presented as concisely as possible on AWS.

![Architecture Diagram](/images/3-BlogsTranslated/3.1-Blog1/blog-1.png)

Here are the highlights of the solution:

### 1. Unlocking Advanced Search Capabilities
The solution uses Amazon OpenSearch Serverless to provide search features that Cognito's native API cannot do:
* Fuzzy search: Allows finding users even with partial information (partial email, name) or typos.
* Complex filtering: Executes complex queries, combining multiple attributes simultaneously such as: email, phone number, groups, and registration date.

### 2. Dual Data Ingestion Architecture (No "Blind Spots")
This is the smartest design of the solution to ensure data is always synchronized in real-time without running manual tasks:
* Capturing changes from the user side: Uses Cognito Lambda Triggers (Post-confirmation and Pre-token generation) to automatically save data as soon as users register or log in.
* Capturing changes from the Admin side: Administrator actions (e.g., manual user creation, password changes) are automatically collected through AWS CloudTrail and Amazon EventBridge, ensuring no changes are missed.

### 3. Using DynamoDB Streams as a Safe "Buffer"
Instead of pushing data directly from Cognito to OpenSearch (which can easily cause overload or data loss), the solution uses Amazon DynamoDB as intermediate storage.
Every state change triggers DynamoDB Streams, which in turn calls Lambda to update OpenSearch. This mechanism helps the system run extremely smoothly and with high fault-tolerance.

### 4. Speed and Performance (Sub-second Performance)
Completely separating the Data Write flow (running in the background via Events) and the Data Read flow (via a standalone Search Lambda function) ensures the system never gets bottlenecked.
Search results are returned through a paginated RESTful API with a response time of under 1 second, regardless of whether the database has thousands or millions of users.

### 5. Serverless Infrastructure & Automated Deployment
The entire solution is built on fully Serverless services, automatically scaling according to actual traffic to optimize costs.
The author of the article on AWS also provides the full source code (Infrastructure as Code using AWS CDK) including both Backend and a React frontend, allowing anyone to deploy this complete system to their AWS account in under 20 minutes. (Link: https://github.com/aws-samples/sample-user-search-layer-for-cognito.)

Original article link: [https://aws.amazon.com/blogs/architecture/building-a-scalable-user-search-layer-on-top-of-amazon-cognito/](https://aws.amazon.com/blogs/architecture/building-a-scalable-user-search-layer-on-top-of-amazon-cognito/)