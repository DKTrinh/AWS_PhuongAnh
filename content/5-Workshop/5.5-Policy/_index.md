---
title : "Clean up"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---
### Resource Cleanup and Decommissioning
Upon successfully completing the practical testing scenarios for the ESP32 tracker diagnostics and maintenance flow, all experimental cloud infrastructure resources were decommissioned to optimize the budget and eliminate the risk of unexpected charges on the AWS account:

* **Clear S3 Storage Space**: Permanently delete all experimental hardware crash logs and OTA firmware patch files within the `tracker-maintenance-storage` bucket to free up storage capacity.
* **Deactivate VPC Endpoints**: Remove the dedicated S3 Gateway VPC Endpoint and delete the internal routing entries from the Route Tables of the Private Subnets within the `Tracker-VPC` to restore the original network configuration.
* **Dismantle Monitoring & Alert Systems**: Delete the `TrackerHardwareErrorFilter` metric filter and remove the sensor failure warning Alarms on the Amazon CloudWatch service. Concurrently, delete the Topic on Amazon SNS to terminate the automated maintenance coordination email flow.