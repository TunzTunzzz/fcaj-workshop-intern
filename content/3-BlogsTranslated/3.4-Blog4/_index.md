---
title: "Blog 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.4. </b> "
---
## BUILDING A MULTI-ACCOUNT PATCH COMPLIANCE DASHBOARD WITH KIRO SPECS

**Posted by:** Hoang Trong Tra 

**AWS Study Group Posting Date:** June 18, 2026

When a system only has a few EC2 instances or a couple of AWS accounts, checking which machines have been patched and which ones are missing patches can be done manually. But as the environment scales out to multiple accounts, multiple Regions, and various workloads, tracking patch compliance begins to become much more difficult to control.
This AWS Blog caught my attention because it doesn't just talk about creating a dashboard; it also shows how to build that dashboard following a very systematic process using Kiro Specs.

The core idea of the solution is: patch compliance data from AWS Systems Manager Patch Manager will be exported to Amazon S3 via Resource Data Sync. Then, a Lambda function running periodically will read this raw data, aggregate it into JSON cache files, and the dashboard simply reads the pre-processed data to display faster.

![Architecture Diagram](/images/3-BlogsTranslated/3.4-Blog4/architecture.jpg)

### I noticed a few notable points:

* **First**, this dashboard is not exposed to the public Internet. Users access it via AWS Systems Manager Session Manager port forwarding to an internal Application Load Balancer. This approach makes the system safer since it doesn't require opening a public endpoint for a dashboard containing internal compliance information.
* **Second**, the solution clearly separates the source data and the data serving the dashboard. The Resource Data Sync bucket is just a place to hold the raw data. The dashboard uses a separate S3 bucket to store frontend assets and the aggregated cache. As a result, the dashboard doesn't have to read thousands of raw files every time a user opens the page.
* **Third**, the serverless architecture is quite reasonable. Lambda handles the frontend, API, and cache aggregation; EventBridge triggers the Lambda to update the cache every 30 minutes; ALB routes internal requests to Lambda. This method reduces server operations while fitting the needs of a dashboard that doesn't always have continuous traffic.
* **Fourth**, the dashboard is designed in two layers. The first layer shows the big picture such as total instances, compliance rate, compliant/non-compliant machines count, categorized by platform and severity. When a deeper look is needed, users can drill down into each account to know exactly which instance is missing patches and which specific patches are missing.

The part I found best is how the article utilizes Kiro Specs. Instead of just asking AI to write code immediately, Kiro follows the process Requirements -> Design -> Tasks. This means having to clearly define what to build first, then designing the architecture and data flow, before dividing it into deployment tasks.
This approach helps AI guess less. Especially with security-related systems like a compliance dashboard, if not specified clearly from the start, AI might create an inappropriate design, like opening public endpoints or processing data sub-optimally.
The article also uses steering files to retain context for Kiro, for instance architecture.md, data-schemas.md, compliance-logic.md, frontend-specs.md, and security.md. As I understand it, this is like writing down the "rules of the game" for the project so that AI always sticks to them when generating requirements, design, or code.

What I learned after reading this article is: a well-operated dashboard does not only lie in a beautiful interface or clear charts. The more critical part is where the data comes from, how it is processed, whether access permissions are safe, and if the architecture is easily scalable when the number of accounts increases.

**Original article:** [https://aws.amazon.com/.../build-a-multi-account-patch.../](https://aws.amazon.com/.../build-a-multi-account-patch.../)

*If you are managing multiple AWS accounts, what would you prioritize first: aggregating compliance data, securing dashboard access, or automated alerts when an instance misses a patch?*
