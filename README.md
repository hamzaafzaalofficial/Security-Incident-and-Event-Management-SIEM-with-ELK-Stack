# Security-Incident-and-Event-Management-SIEM-with-ELK-Stack

---

## Objectives:

1. Deploy Elasticsearch, Logstash, and Kibana (ELK Stack) on AWS.
2. Ingest AWS CloudTrail logs into Elasticsearch using Logstash
3. Visualize CloudTrail logs in Kibana.
4. Set up detection rules and alerts to monitor and identify potential security threats.

Note: 
For visual aids check out here: https://www.linkedin.com/pulse/security-incident-event-management-siem-elk-stack-hamza-afzal-dutte/


## Steps:

## Step1: 

Setup an s3 bucket named "cloud-trail-bucket-786". 
Simply create a S3 bucket.

## Step2:

Setup the cloud trail logs to deliver to s3 bucket created above. 
Go to cloud trail console and create a new trail or modify an existing one name it cloudtrail-logs.
Specify the S3 bucket where you want the logs to deliver. Enable the trail and verify logs delivery.
Here cloud trail is configured to send logs to s3 bucket "cloud-trail-bucket-786"

## Step3:
Create an IAM user and attach policy of "Full S3 Read" policy to this user.  Download this user's access keys and id. 
This will be used in later steps. 

## Step4: 
Launch a cloud instance of ec2 or on-prem instance and setup 
Elasticsearch, Kibana, (Logstash or any other of your choice) 

*** Refer this article for complete setup of ELK stack on cloud instance: https://www.linkedin.com/pulse/nlp-driven-automated-compliance-reporting-elk-stack-hamza-afzal-bi0me/?trackingId=jd8O9qWHRl6eC2aADWsbsw%3D%3D


Configure Logstash to ingest cloudtrail-logs & output to elasticsearch endpoint

-> Change your working directory

```bash

cd /etc/logstash/conf.d 

vi cloudtrail.conf

input {
  s3 {
    bucket => "cloud-trail-bucket-786"
    region => "us-east-1"
    prefix => "AWSLogs/780621109903/CloudTrail/"
    type => "cloudtrail"
    codec => json
    interval => 60
    access_key_id => "QFE3KHS7O"
    secret_access_key => "7FNPunVapKTyza5ngxgYzhLn8V3"
    sincedb_path => "/var/lib/logstash/sincedb"
    additional_settings => {
      force_path_style => false
      follow_redirects => true
    }
  }
}

filter {
  json {
    source => "message"
  }

  date {
    match => [ "eventTime", "ISO8601" ]
    target => "@timestamp"
  }

  mutate {
    remove_field => ["@version", "path", "host"]
    rename => {
      "sourceIPAddress" => "source_ip"
      "userIdentity" => "user"
      "eventName" => "action"
    }
  }

  geoip {
    source => "source_ip"
    target => "geoip"
  }
}

output {
  elasticsearch {
    hosts => ["http://167.71.239.192:9200"]
    index => "cloudtrail-logs-%{+YYYY.MM.dd}"
  }

}

-> Once the above code is customized according to your credentials, run 

sudo systemctl restart logstash

```

## Step5: 
Access the Kibana dashboard and navigate to management section. Use cloudtrail-logs as the pattern to match the indices created by logstash.  
The entry of cloudtrail-logs ensures logs are indeed ingested by logstash to Elasticsearch
Explore the logs and setup dashboards accordingly.

## Step6: 
Enable pre-built detection Rules. Go to security tabs in Kibana and enable pre detection rules. 

## Step7: 
Create new alerts by defining trigger conditions and actions when a certain pattern is detected. 

## Summary: 

This article guide covers the deployment of an ELK stack on AWS, configuring it to ingest AWS CloudTrail logs, and setting up threat detection and alerting mechanisms. This setup helps you monitor and secure your AWS environment by analyzing CloudTrail logs in real-time.

---
