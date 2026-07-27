---
title: "Proposal"
date: 2026-07-26
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Tracker Maintenance – Maintenance Management System on AWS
### A secure operational solution with a Multi-tier Cloud Web architecture and JWT Authentication

### 1. Executive Summary
Tracker Maintenance is a maintenance task management system built on a modern Multi-tier architecture on Amazon Web Services (AWS). From a technical perspective, the user interface (Frontend) is developed using React/Vue, while the core processing system (Backend) is built with Node.js/FastAPI, integrated with a secure relational database. The system is completely autonomous in managing identity through a JWT Auth mechanism. 

### 2. Objectives
With Tracker Maintenance, the core objective focuses on optimizing management workflows and maximizing system security:
- Build a stable Cloud infrastructure, clearly separating Front-end and Back-end access flows.
- Replace dependent authentication solutions (like Cognito) with a more flexible and autonomous internal JWT system.
- Implement proactive defense mechanisms against password-guessing attacks (Brute-force protection).
- Optimize server load by utilizing direct file uploads to S3 (Pre-signed URL) and automated event processing.

### 3. Problem Statement
- **Current Situation:** Traditional tracking systems are often vulnerable to brute-force attacks, suffer from exposed API Key configurations, and frequently encounter bottlenecks when processing large file uploads through the main server.
- **Solution:** Tracker Maintenance leverages an Amazon VPC virtual private network to protect the database. The system transitions to JWT authentication, strictly secures configurations using `.env` files, and applies an Event-Driven architecture with S3 and Lambda for file processing.
- **Benefits:** Delivers a highly available system with robust security from the network layer to the application layer, while providing a seamless experience for end-users.

### 4. System Architecture
The entire infrastructure is deployed on AWS within an internal Amazon VPC network, clearly divided into a Public Subnet (hosting the EC2 server) and a Private Subnet (hosting the RDS database).

**Technologies used:**
- **Frontend:** React/Vue (Static hosting and CDN distribution).
- **Backend:** Node.js / FastAPI.
- **Database:** Amazon RDS (PostgreSQL/MySQL).

**Core AWS Services:**
- **Amazon EC2:** The main server running the Backend application, handling JWT authentication logic and Brute-force control.
- **Amazon S3:** Static storage for the Frontend and a repository for files/images uploaded by users.
- **Amazon CloudFront & Route 53:** Global content delivery network (CDN) and high-speed DNS resolution.
- **AWS Lambda & Amazon SNS:** Automated Event-Driven processing flow. When a new file arrives on S3, Lambda is triggered to process it, and SNS sends a notification to technicians.
- **Amazon CloudWatch:** Centralized monitoring and logging service to detect system errors.

![System Architecture](/images/2-Proposal/architecture.png?classes=shadow)

**Core Data Flows:**
- **Secure Authentication Flow:** The user sends a login request. The EC2 server checks the Brute-force logic (blocking if there are too many failed attempts). If valid, the Backend issues a secure JWT Token for the user to access business APIs.
- **Optimized File Upload Flow:** The Frontend calls an API to EC2 to request permission. EC2 returns a temporary S3 Pre-signed URL. The Frontend uses this URL to upload the file directly to S3, bypassing heavy transmission through EC2.
- **Event Notification Flow:** As soon as the file is successfully uploaded to S3 (Event Trigger), AWS Lambda automatically initiates processing logic and triggers Amazon SNS to push real-time notifications.

### 5. Technical Implementation
The development team divides the technical tasks to ensure project progress:
- **Front-end & Back-end Development:** WhooDuck1810 and phuonganh284 collaborate to develop the React/Vue interface, set up the `.env` security environment, and write API logic on Node.js/FastAPI.
- **Authentication Security:** Remove AWS Cognito integration, program, and fully transition to a JWT authentication mechanism combined with Anti-login protection features.
- **AWS Infrastructure Deployment:** Set up the VPC network architecture, configure S3 Pre-signed URLs, and the Lambda - SNS event flow.

### 6. Implementation Roadmap
The implementation roadmap for the Tracker Maintenance project takes place over 8 weeks:
- **Weeks 1-2:** Research Web architecture overview on AWS. Initialize the repository and source code structure.
- **Weeks 3-4:** Build the basic application framework. Set up secure environment variables (`.env`), and clean up exposed API keys from Git history.
- **Weeks 5-6:** Deploy the application to AWS infrastructure (EC2, S3, CloudFront), integrate the RDS database, and configure initial Amplify/Cognito steps.
- **Weeks 7-8:** Optimize authentication (replace Cognito with JWT). Implement anti-brute-force features, optimize UI/UX, and finalize system documentation.

### 7. Cost Estimation
The combined architecture of EC2 and AWS managed services helps optimize costs:
- **Amazon EC2 & RDS:** Utilizing small server instances (t2.micro/t3.micro) falls within the Free Tier limits for the development environment.
- **Amazon S3 & CloudFront:** Static content delivery and storage costs are extremely low, mostly covered by the monthly free tier.
- **AWS Lambda & SNS:** Charged based on the number of function invocations and messages, extremely economical for the event processing flow.

### 8. Risk Assessment
- **Risk of Sensitive Information Exposure:** Completely resolved through the strict synchronization of `.env` files and proper `.gitignore` configuration.
- **Password Guessing (Brute-force) Attacks:** Prevented by Anti-login protection logic programmed directly in the Backend.
- **Database Intrusion:** Very low risk thanks to the VPC network design that completely isolates Amazon RDS in the Private Subnet, preventing direct access from the Internet.

### 9. Expected Results
Successfully deploy a secure, stable, and performance-optimized Tracker Maintenance system. The project serves as a testament to the seamless integration between traditional Multi-tier Web architecture (EC2, RDS) and modern Cloud features (S3 Pre-signed URL, Event-Driven Lambda), resulting in a powerful maintenance management tool that is fully autonomous in terms of authentication.
