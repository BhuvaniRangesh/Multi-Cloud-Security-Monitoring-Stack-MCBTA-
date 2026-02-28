Multi-Cloud Security Monitoring Stack (MCBTA)
Base Machine & ELK SIEM Deployment

This repository documents the deployment of a centralized Security Information and Event Management (SIEM) node designed to aggregate telemetry from AWS, Azure, and GCP.
1. Project Overview

The objective of this module is to establish a Base Machine in the cloud that serves as the "Brain" of the security stack. This machine is responsible for indexing logs, managing remote agents, and providing a visualization dashboard for security analysts.
2. Infrastructure Setup
Provisioning & Access

    Platform: Amazon Web Services (AWS)

    Instance: EC2 Ubuntu-based Linux

    Authentication: RSA Key Pair (.pem)

    Configuration: * Resolved SSH identity errors by generating a fresh key pair.

        Applied secure permissions: chmod 400 your-key.pem.

3. Installation & Configuration Log
Step 1: Repository Integration

To ensure the integrity of the Elastic Stack, the official GPG keys and 8.x repositories were added.
Bash

sudo apt-get update
sudo apt install wget curl gnupg2 -y

# Add Elastic GPG Key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

# Add Elastic 8.x Repository
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt-get update

Step 2: Elasticsearch Deployment

The core search engine was installed and modified to allow external cloud ingestion.
Bash

sudo apt install elasticsearch
sudo su

# Configuration: Edit /etc/elasticsearch/elasticsearch.yml
# Action: Set network.host: 0.0.0.0 and uncomment http.port: 9200

sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl restart elasticsearch.service
sudo systemctl status elasticsearch.service

Step 3: Kibana & Filebeat Initialization

The visualization layer (Kibana) and the data shipper (Filebeat) were deployed to complete the monitoring foundation.
Bash

sudo apt install kibana filebeat

# Configuration: Edit /etc/kibana/kibana.yml
# Action: Set server.host: "0.0.0.0" and link to localhost:9200

sudo systemctl enable kibana filebeat
sudo systemctl start kibana filebeat

4. Network Security (Cloud Firewall)

The AWS Security Group was updated with the following inbound rules to permit management and data flow:
Protocol	Port	Description
TCP	22	SSH Remote Access
TCP	9200	Elasticsearch API
TCP	5601	Kibana Web Dashboard
TCP	9600	Logstash/Filebeat Ingestion
5. Verification

Deployment was verified by successfully accessing the Kibana Web UI via the Public IP on port 5601. The Elasticsearch service status was confirmed as active (running)
