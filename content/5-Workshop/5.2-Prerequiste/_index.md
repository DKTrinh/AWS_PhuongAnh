---
title: "Prerequisite"
date: 2026-07-26
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### Prerequisite

#### Accounts and Access (AWS IAM & CLI)

The system requires an active AWS account with IAM administrator privileges. To comply with the Principle of Least Privilege, the development team does not use the main Root account. Instead, a secondary developer account named **Tracker-Developer** is created to grant permissions and obtain security keys for connecting from the local workstation through the following steps:

- **Step 1 (Configuration on AWS Console):** Log in to the AWS Management Console using the administrator account and navigate to the **IAM (Identity and Access Management)** service → **Users** → select the user **Tracker-Developer**. Go to the **Security credentials** tab, locate the **Access keys** section, and select **Create access key**. Choose **Command Line Interface (CLI)** as the use case, agree to the terms, and confirm to generate the key pair: **Access Key ID** and **Secret Access Key**. Download the `.csv` file containing this security key information.

- **Step 2 (Configuration on Local Workstation):** Open Terminal or PowerShell on your personal computer and run the following command:

```bash
aws configure
````

Enter the **Access Key ID** and **Secret Access Key** generated in Step 1. Set the **Default region name** to `ap-southeast-1` (Singapore — the ideal deployment region to optimize network latency for users in Vietnam) and the **Default output format** to `json`. This configuration will be automatically saved in the user directory (`~/.aws/` on Linux/macOS or `%USERPROFILE%\.aws\` on Windows).

> [!NOTE]
> The **IAM Security credentials** tab, with the **Access keys** section, is used to generate CLI connection keys.

#### Local Workstation Environment and Source Code

Ensure the workstation has successfully installed **Node.js** (version 18 or higher) and **Python** (version 3.10 or higher) for backend development (FastAPI/Node.js) and frontend bundling (React/Vue).

Install **Git** for source code management. Pay special attention to verifying the `.gitignore` file configuration before committing to avoid accidentally pushing sensitive information such as API keys to the remote repository.

Prepare an `.env` file in the local environment containing essential environment variables such as the database connection string (`DB_URL`) and the token encryption secret (`JWT_SECRET`).

#### Infrastructure Region Selection

Select the AWS network infrastructure region in Singapore (`ap-southeast-1`) as the default deployment region. This ensures the lowest latency, providing a smooth access experience for the Tracker Maintenance system while fully supporting all the core AWS services that make up the architecture.


