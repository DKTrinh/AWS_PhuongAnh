---
title : "Implementation Steps - Tracker Maintenance"
date : 2024-01-01 
weight : 3 
chapter : false
pre : " <b> 5.3. </b> "
---
### Detailed Implementation Steps

#### Step 1: Setting up an isolated private VPC
The process of building an isolated network environment to place the entire device maintenance data processing flow into a secure zone:

Initialize a custom VPC named **Tracker-VPC** with an IP CIDR block of `10.1.0.0/16` as the primary private network range.

![VPC List](/images/5-Workshop/5.3-Implementation_Steps/tracker-vpc-list.png?classes=shadow)

> [!NOTE]
> VPC list displaying the custom VPC with CIDR 10.1.0.0/16 dedicated to the Tracker Maintenance system

Divide the subnet ranges into isolated tiers: Public Subnets (housing components exposed to the public Internet) and Private Subnets (hosting internal network interfaces of compute services to completely isolate their IPs from the Internet Gateway). The system is evenly distributed across two Availability Zones (us-east-1a and us-east-1b) to ensure redundancy.

![Subnet List](/images/5-Workshop/5.3-Implementation_Steps/tracker-subnet-list.png?classes=shadow)

> [!NOTE]
> List of Subnets distributed across Public and Private tiers of the maintenance architecture

Configure the corresponding Route Tables: The Public Subnet routes directly to the Internet Gateway; the Private Subnet establishes an internal routing range with the NAT Gateway set to None to prevent continuous monthly account credit consumption.

#### Step 2: Deploying the Serverless Backend to process ESP32 Tracker logs

Initialize a Serverless compute function named **Process_ESP32_Tracker_Telemetry** running on the Node.js environment. This function handles intensive business logic: receiving diagnostic data packets and battery statuses from ESP32 tracker devices, performing hardware fault analysis, and generating a short-lived S3 Presigned URL (with a lifespan limited to 5 minutes) to grant the device permissions to upload local log files.

![Lambda Source Code Configuration](/images/5-Workshop/5.3-Implementation_Steps/tracker-lambda-source.png?classes=shadow)

> [!NOTE]
> Source code configuration interface of the Process_ESP32_Tracker_Telemetry Lambda function integrated with API Gateway on the AWS Console

Configure the network connection for the Lambda function: Navigate to the Configuration tab -> select VPC -> Connect the Lambda function directly to the Tracker-VPC.

Accurately select the 2 Private Subnets to force Lambda to run entirely within the secure internal network partition. Assign the VPC's default Security Group to the Lambda function to manage Inbound/Outbound rules.

![Lambda VPC Configuration](/images/5-Workshop/5.3-Implementation_Steps/tracker-lambda-vpc.png?classes=shadow)

> [!NOTE]
> Configuration for allocating the Lambda function to run within the Tracker-VPC internal network environment and Private Subnets on the AWS Console

#### Step 3: Configuring system permissions on AWS IAM

When configuring Lambda to be placed in Private Subnets, the system initially throws a permission error because Lambda does not yet have the authority to create internal virtual network interfaces.

To resolve this, access the IAM service -> Roles -> Accurately select the current execution Role of the function (Process_ESP32_Tracker_Telemetry-role-...).

Click the Add permissions button -> Attach policies -> Search for and select the standard AWS managed policy: AWSLambdaVPCAccessExecutionRole.

Confirm the permission assignment to grant Lambda full privileges to create and manage Elastic Network Interfaces (ENIs) for accessing the VPC network range.

![IAM Role Assignment](/images/5-Workshop/5.3-Implementation_Steps/tracker-iam-role.png?classes=shadow)

> [!NOTE]
> Interface for assigning the AWSLambdaVPCAccessExecutionRole policy to the maintenance system Lambda function's IAM Role on the AWS Console

#### Step 4: Setting up Amazon S3 Security for Firmware and Log storage
Establish a private network connection to optimize costs and protect the device data storage repository:

Initialize an Amazon S3 Bucket named **tracker-maintenance-storage** in completely private mode (Block Public Access) and enable default Server-Side Encryption (SSE-S3) to store all over-the-air (OTA) firmware updates and hardware error logs.

![S3 Files Management](/images/5-Workshop/5.3-Implementation_Steps/tracker-s3-files.png?classes=shadow)

> [!NOTE]
> Interface for managing firmware files and error logs stored inside the tracker-maintenance-storage Bucket on the AWS Console

On the VPC Console management page, create a Gateway VPC Endpoint for S3 (service name: com.amazonaws.us-east-1.s3) and assign it directly to the Route Tables of the Private Subnets within the Tracker-VPC.

The new route entry automatically added by the system will direct all requests from the internal Lambda function to Amazon S3 through AWS's internal backbone network instead of routing out to the Internet. This mechanism cuts 100% of the network bandwidth costs for S3 and accelerates software patch transfer speeds.

Configure a strict S3 Bucket Policy to protect resources: Absolutely deny all read/write access requests from the external Internet, and only accept valid interaction requests that pass through the newly created Gateway VPC Endpoint of the system.

![S3 Block Public Access](/images/5-Workshop/5.3-Implementation_Steps/tracker-s3-security.png?classes=shadow)

> [!NOTE]
> Activation status of the Block Public Access feature securing the S3 maintenance data storage on the AWS Console

#### Step 5: Configuring the maintenance alert system with Amazon CloudWatch & SNS
Set up an automatic monitoring "eye" to track offline statuses or sensor failures of the trackers:

The business Lambda function automatically pushes all detailed diagnostic logs to the Amazon CloudWatch Log Groups service in real-time.

Set up a custom Metric Filter named **TrackerHardwareErrorFilter** on the project's Log Group to scan through the log streams and count the frequency of capitalized error codes like "HARDWARE_FAULT", "SENSOR_TIMEOUT", or "BATTERY_CRITICAL".

Initialize a CloudWatch Alarm connected directly to the filter's metric. Configure a highly sensitive alert threshold: Transition to an emergency alarm state (ALARM) as soon as the number of errors is >= 5 within a 5-minute monitoring period.

Link the Alarm with the Amazon SNS (Simple Notification Service) "siren". When the alarm state triggers, SNS will automatically compose a system maintenance request notification and fire off a real-time emergency email to the developer's administrative inbox (dokat0903000@gmail.com).

![CloudWatch Log Groups Management](/images/5-Workshop/5.3-Implementation_Steps/tracker-cloudwatch-logs.png?classes=shadow)

> [!NOTE]
> Interface for managing CloudWatch Log Groups to monitor Tracker hardware diagnostic logs on the AWS Console