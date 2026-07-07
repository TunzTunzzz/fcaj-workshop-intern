---
title: "Store Result and Build Review Flow"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Store Result and Build Review Flow

### 1. Goal
Implement a persistent storage strategy using an **Offloading Pattern** between Amazon DynamoDB and Amazon S3. Deploy four Lambda functions (Get, List, Update, Delete) to serve as backend APIs for the frontend UI, supporting manual review workflows (Human-in-the-loop).

---

### 2. Lab Exercises

Select a sub-page below to complete the step-by-step instructions for each component:

* **[5.7.1 Persistence & Offloading Design](5.7.1-persistence/)**: Understand Single-Table Design patterns on DynamoDB and the metadata/payload separation mechanisms.
* **[5.7.2 Data Management Lambdas](5.7.2-lambdas/)**: Deploy and configure the four core Lambda functions (Get, List, Review Update, and Delete).
* **[5.7.3 API Gateway Configuration](5.7.3-api-gateway/)**: Configure Amazon API Gateway as the frontend connection and integrate the Cognito Authorizer.
* **[5.7.4 API Testing & Reporting](5.7.4-testing/)**: Create mock database/S3 data, test API Lambda execution parameters, and generate statistics reports in AWS CloudShell.
