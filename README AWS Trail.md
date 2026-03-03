AWS CloudTrail Ingestion Pipeline
Overview
This module implements a real-time serverless pipeline to ingest AWS administrative logs into a centralized Elastic Stack (ELK) instance. It utilizes an event-driven architecture to minimize latency between API calls and security visualization.

Prerequisites
IAM User: Created with AmazonSQSReadOnlyAccess and AmazonS3ReadOnlyAccess permissions.

Elastic Stack: Version 8.x (deployed on EC2).

Filebeat: Installed on the Base Machine with the aws module enabled.

1. Infrastructure Configuration (Terraform/Manual)
CloudTrail: Configured to log all Management Events (Read/Write) across the global account.

S3 Bucket: Acts as the persistent storage layer for compressed .json.gz log files.

SQS Queue (MCBTA): Acts as the notification buffer.

S3 Event Notification: Configured on the bucket to trigger s3:ObjectCreated:* events, sending metadata directly to the SQS ARN.

2. Security Policy (SQS Access)
To ensure only the designated S3 bucket can publish to our queue, the following Resource-Based Policy was applied:

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "SQS:SendMessage",
      "Resource": "arn:aws:sqs:us-east-2:677920913760:MCBTA",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:::aws-cloudtrail-logs-677920913760-5a746840"
        }
      }
    }
  ]
}
3. Filebeat Configuration
The aws.yml module was configured to poll the SQS queue using the following parameters:

YAML
- module: aws
  cloudtrail:
    enabled: true
    var.queue_url: https://sqs.us-east-2.amazonaws.com/677920913760/MCBTA
    var.access_key_id: ${AWS_ACCESS_KEY}
    var.secret_access_key: ${AWS_SECRET_KEY}
4. Key Achievements
Reduced Latency: Shifted from "Pull" (polling S3) to "Push" (SQS notifications), reducing log ingestion time from minutes to seconds.

Secure Transit: Implemented ca_trusted_fingerprint verification to ensure encrypted delivery to the Elasticsearch cluster.

Dashboards: Integrated the default Filebeat AWS Dashboard to visualize global API activity and geographic login heatmaps.