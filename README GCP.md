\# GCP Log Ingestion Workflow



To ingest logs from GCP into the AWS Base Machine, I implemented a \*\*Push-to-Pull\*\* architecture:



1\. \*\*Log Sink:\*\* Created a Log Router sink in GCP to filter `resource.type="audited\_resource"` and export them to a Pub/Sub Topic.

2\. \*\*Pub/Sub:\*\* The Topic acts as a buffer. I created a \*\*Pull Subscription\*\* named `filebeat-sub`.

3\. \*\*IAM:\*\* Created a Service Account with the `Pub/Sub Subscriber` role to allow the Base Machine to pull messages.

4\. \*\*Ingestion:\*\* Filebeat on the Base Machine connects to the GCP API using the Service Account JSON key, pulls the audit logs, and indexes them into Elasticsearch.

