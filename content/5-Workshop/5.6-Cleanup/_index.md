---
title : "Difficulties & Development Direction"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---
### Challenges & Future Development

#### Challenges Encountered

* **Hidden network permission error (ENI)**: During the initial phase of deploying the `Process_ESP32_Tracker_Telemetry` Lambda function into the Private VPC subnet, the system denied network access. After consulting the AWS Knowledge Center regarding the `CreateNetworkInterface` error, it was discovered that the function lacked permissions to create internal virtual network interfaces. This was resolved by attaching the managed `AWSLambdaVPCAccessExecutionRole` policy to its IAM Role.
* **Security policy conflict (Bucket Policy vs. Endpoint Policy)**: When securing the `tracker-maintenance-storage` bucket, blocking all public access too early—before accurately specifying the VPC Endpoint ARN—resulted in the internal Lambda function itself being denied permission to generate S3 Presigned URLs (Access Denied). The author had to trace errors through CloudWatch logs and utilize the AWS CLI to debug and rectify the policy configuration sequence.
* **Inconsistent sensor data handling**: Telemetry payloads transmitted from the ESP32 boards were occasionally malformed or interrupted due to signal interference or critically low battery, initially causing the Backend system to crash during JSON parsing. The author had to implement robust data validation mechanisms and strict `try-catch` blocks within the Lambda function.

#### Future Development Directions

* **Infrastructure as Code (IaC) Deployment**: Transition all manual Web Console configurations (VPC, Subnets, Lambda, Endpoint, IAM, S3) to centrally managed code using AWS CloudFormation or Terraform. This will make it effortless to replicate the environment to support thousands of IoT devices with a single deployment command.
* **Integrating MQTT protocol with AWS IoT Core**: To significantly optimize the ESP32's battery life and ensure stable connectivity in weak network conditions, the architecture is planned to migrate from standard HTTP REST API calls (via API Gateway) to the lightweight MQTT protocol managed by the AWS IoT Core service.
* **Building a Fleet Management Dashboard**: Synchronize discrete metrics from CloudWatch Alarms and system logs into a unified monitoring interface. This dashboard will provide a visual map of device locations, real-time battery levels, and hardware error alerts across the entire tracker fleet, thereby streamlining the fieldwork coordination for the maintenance team.