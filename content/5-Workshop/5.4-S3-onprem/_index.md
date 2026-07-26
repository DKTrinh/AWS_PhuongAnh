---
title : "Test Results & Experimentation"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---
### Testing & Experimental Results

> [!TIP]
> 🎬 **Demo Video:** Watch the live demonstration video of the Tracker Maintenance system testing on YouTube: 


After finalizing the isolated network infrastructure and Serverless configuration, practical end-to-end testing was conducted to evaluate the stability of the ESP32 device maintenance log collection flow:

#### Testing the Technician Registration & Authentication Flow (AWS Cognito)

Provisioned a new account for a Maintenance Technician on the internal management application.

![Cognito Registration Flow](/images/5-Workshop/5.4-Test_Results_Experimentation/tracker-cognito-auth.png?classes=shadow)

> [!WARNING]
> Image illustrating the account creation and authorization function for Maintenance Technicians

The AWS Cognito system correctly registered the new technician and automatically dispatched temporary login credentials via email. The technician logged in for the first time, changed the password, and the account status transitioned to "Confirmed," granting access to the maintenance dashboard.

![Cognito Users Management](/images/5-Workshop/5.4-Test_Results_Experimentation/tracker-cognito-users.png?classes=shadow)

> [!WARNING]
> Management interface of the Maintenance Technician account list and authentication status on the AWS Cognito User Pool

#### Testing the Device Crash Log Upload Flow via S3 Presigned URL

Simulated an ESP32 Tracker device encountering a hardware failure. The device automatically dispatched a payload containing the error code and MAC Address to the API Gateway to trigger the `Process_ESP32_Tracker_Telemetry` AWS Lambda function.

The Lambda function, secured within the Private Subnet, executed smoothly, authenticated the device's signature, and invoked the internal S3 Gateway Endpoint. It requested Amazon S3 to issue a temporary encrypted link (Presigned URL) valid for 5 minutes, allowing the device to upload its local log file.

Upon receiving the link, the ESP32's Wi-Fi module executed an HTTP PUT request to push the `.txt` file containing the crash log directly into the Amazon S3 Bucket `tracker-maintenance-storage`.

Accessing the S3 Console verified that the device's log file was automatically categorized into the correct date folder with the exact file size, proving that the internal VPC secure connection flow operated with 100% success.

![S3 Log Upload Success](/images/5-Workshop/5.4-Test_Results_Experimentation/tracker-s3-upload.png?classes=shadow)

> [!WARNING]
> Image of the ESP32 hardware diagnostic log file securely uploaded to the S3 system

#### Testing the Hardware Error Alert System with CloudWatch & SNS

Intentionally disconnected a sensor on the ESP32 hardware to force the device to continuously send "HARDWARE_FAULT" and "SENSOR_TIMEOUT" alert strings to the Backend system.

As soon as the data was received, CloudWatch Log Groups immediately recorded the logs. The `TrackerHardwareErrorFilter` Metric Filter caught the critical error keywords and pushed the metric beyond the configured threshold (>= 5 errors within 5 minutes).

The system instantly transitioned into the ALARM state. In under 30 seconds, an urgent maintenance request notification email from Amazon SNS was directly dispatched to the administrative inbox, including the ID of the failing device. This proves that the proactive monitoring and automated maintenance alert flow operates exactly as designed.

![CloudWatch Alarm Triggered](/images/5-Workshop/5.4-Test_Results_Experimentation/tracker-cloudwatch-alarm.png?classes=shadow)

> [!WARNING]
> Emergency alert triggered on CloudWatch, dispatching a maintenance coordination email for the faulty device