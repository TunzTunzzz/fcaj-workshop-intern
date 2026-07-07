---
title: "Prerequisite"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Prerequisite

### 1. Goal
Ensure readers successfully log in to the AWS Management Console, set up local tools for the Frontend, and prepare sample files to test all execution paths before starting the hands-on labs.

### 2. Tools to Prepare
For building and managing AWS resources, **the entire team agrees to use the AWS Management Console (web UI)** instead of CLI or SAM CLI commands.
You will only need to install the following software on your local machine to run the Frontend locally:
* **Node.js (v18+)**: Required for running the local frontend server.
* **Git**: Source control management.
* **VS Code** (or any preferred IDE): For code editing.

### 3. Steps
1. **Log in to AWS Console**: Sign in to the designated AWS account for the workshop via the **AWS Management Console** using the provided IAM User or AWS IAM Identity Center credentials. Make sure the region is set to **ap-southeast-1** (Singapore) in the navigation bar.

   *(Take a screenshot of the AWS Management Console showing the ap-southeast-1 region and save it to `static/images/5-Workshop/5.2-prerequisite/aws-console.png`)*
   ![AWS Console Access](/images/5-Workshop/5.2-prerequisite/aws-console.png)

2. **Verify Local Tools**: Ensure your local machine can execute these commands for running the frontend:
   ```bash
   node --version
   npm --version
   ```

   *(Take a screenshot of checking node/npm versions in terminal and save it to `static/images/5-Workshop/5.2-prerequisite/node-version.png`)*
   ![Node and NPM version](/images/5-Workshop/5.2-prerequisite/node-version.png)

3. **Prepare Sample Files**: Download or prepare at least 3 types of files to test the extraction pipelines:
   * A clear invoice image/PDF (to test the successful `EXTRACTED` path).
   * A blurry receipt or invoice missing key fields (to test the `REVIEW_REQUIRED` path).
   * An invalid file format (like `.txt` or too large) to test the `FAILED` path.

   *(Take a screenshot of your sample testing files and save it to `static/images/5-Workshop/5.2-prerequisite/sample-files.png`)*
   ![Sample Files](/images/5-Workshop/5.2-prerequisite/sample-files.png)

4. **Agree on Naming Conventions**:
   * All created resources must use the prefix: `docuflow-dev-*`.
   * Mandatory resource tags: `Project=DocuFlowAI`, `Environment=dev`, `Team=AeroOps`.

### 4. Expected Result
* Readers can log into the AWS Management Console and set the region to ap-southeast-1.
* A list of sample files is ready for E2E testing flows.
