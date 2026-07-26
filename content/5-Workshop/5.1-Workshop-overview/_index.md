---
title : "Introduction"
date : 2026-07-26 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

### Cloud Web Infrastructure Deployment and System Security Setup

<div style="text-align: center; margin: 20px 0;">

  ![Overall Architecture](/images/5-Workshop/5.1-Introduction/architecture.png?classes=shadow)

  <div style="font-weight: bold; margin-top: 8px; color: #555;">Figure 1. Overall architecture of the network infrastructure and AWS service integration for the Tracker Maintenance project.</div>
</div>

<br>

### Overview

This lab documents the complete process of building, configuring, and testing the cloud infrastructure for the Tracker Maintenance System project.

The configuration focuses on deploying a multi-tier web architecture on AWS, integrating Event-Driven processing, and implementing deep security measures, including:
* Establishing a Virtual Private Cloud (Amazon VPC) to clearly separate access flows between the Public Subnet (for receiving incoming requests) and the Private Subnet (for isolating and protecting the Amazon RDS database).
* Deploying the core Backend server on Amazon EC2 (FastAPI/Node.js) to handle business logic, while building an independent JWT authentication token issuance system to optimize access control.
* Optimizing the delivery of Frontend static content (React/Vue) hosted on Amazon S3 through the Amazon CloudFront content delivery network and Route 53 DNS web service.
* Building a secure file upload flow using temporary authorization mechanisms (S3 Pre-signed URLs), combined with an Event-Driven architecture using AWS Lambda and Amazon SNS to automatically process images and send notifications to technicians as soon as new data is uploaded.
* Implementing system protection features against password guessing attacks (Brute-force protection) and centralizing log collection and monitoring through Amazon CloudWatch.

Through this practical deployment, the Tracker Maintenance system has strictly applied and adhered to the design standards of the AWS Well-Architected Framework, strongly focusing on three core pillars: Security, Performance Efficiency, and Reliability.