# Global Educational Platform Architecture

## Overview
This document provides an overview of the architectural design for a globally accessible educational platform with a multi-regional deployment, high availability, and optimized costs & content delivery.

## Diagram
![educational platform diagram](/diagrams/educational-platform-diagram.png)

## Architecture Components
### 1. **Front-End**
- A Single Page Application (SPA) built with React or Svelte.
- Hosted on Amazon CloudFront for global caching and low-latency content delivery.
- AWS Route 53 with geolocation-based routing ensures users are directed to the closest regional deployment (Saudi Arabia or the USA).
- AWS Certificate Manager provides HTTPS certificates for secure connections.

### 2. **Backend Services**
#### **Monolithic PHP Application**
- Hosted on Amazon EC2 instances within an Auto Scaling group.
- Load balanced by AWS Application Load Balancer (ALB).
- Deployed in multiple Availability Zones (AZs) for redundancy.
- Backed with EBS volumes for app storage.

#### **Microservices (Containerized on AWS EKS)**
- **Microservices** Load balanced by AWS Application Load Balancer (ALB) and EKS Ingress Kubernetes resource.
- **Analytics Microservice**: Uses ClickHouse for analyzing user interactions.
- **Additional Microservices**: The system is designed to accommodate future services within the EKS cluster.

#### **Microservices (Serverless)**
- **Video Conversion/Encoding Microservice**:
  - Amazon S3 for raw and processed video storage.
  - AWS Lambda for event-driven notifications.
  - AWS MediaConvert for video processing.

### 3. **Data Storage and Multi-Region Setup**
- **Amazon RDS (Multi-AZ)**
  - Separate RDS clusters for the Saudi Arabia and USA regions.
  - Primary-standby replication for high availability.
- **Amazon S3**
  - Regional buckets for user-generated content.
  - Video content stored globally with optimized delivery.

### 4. **Media Processing & Global Delivery**
- **AWS MediaConvert** converts uploaded videos into optimized formats.
- **Amazon S3 (Intelligent Tiering)** minimizes costs by moving older files to lower-cost storage.
- **Amazon CloudFront** distributes video content globally for minimal latency.

### 5. **Security & Compliance**
- **Security Groups and Network ACLs** reducing attack surface for network & resources protection.
- **Nat Gateways** when outbount to public internet is required.
- **VPC Peering** when cross-region communication is required.
- **AWS WAF** additional layer of security by filtering out malicious traffic and protecting against common threats like SQL injection, XSS, and others.
- **AWS Shield** protect CloudFront distribution & Route 53 from DDoS attacks.
- **IAM Role-based Access Control (RBAC)** ensures least privilege access.
- **Encryption**
  - Data in transit secured with TLS (AWS Certificate Manager).
  - Data at rest encrypted using AWS KMS.
- **Logging & Monitoring**
  - AWS CloudWatch for AWS services monitoring & observation.
  - AWS CloudWatch Logs for AWS services logging.
  - Prometheus can be deployed in the cluster to collect metrics from EKS.
  - Fluentd can be deployed in the EKS cluser to collect logs from EKS.
  - PHP monolithic app & other custom developed apps can be instrumented with OpenTelemetry for better logging, monitoring, and tracing.
- **Auditing**
  - AWS CloudTrail can be used for operational and risk auditing, governance, and compliance.
- **AWS Secrets Manager**
  - Storing secrets / credentials / tokens / and other sensitive data.

### 6. **Scalability & Optimization**
- **Auto Scaling Groups** ensure EC2 instances adapt to traffic spikes.
- **Amazon EKS Cluster Scaling** dynamically scales containerized workloads.
- **Amazon CloudFront & S3 Caching** reduce latency and bandwidth costs.
- **Regional Expansion Ready**: Additional AWS regions can be integrated as needed.
- **Disaster Recovery**: Regular S3 & RDS backups, RDS automatic failover in case of zonal outage.
- **Archives**: S3 backups to be stored in Glacier or similar storage classes for cost optimization.
- **Compute Costs**: EC2 Spot instances / Reservations can be used when applicable to minimize compute costs.
- **EKS Scaling**: Enablement of EKS autoscaling can introduce significant cost optimization using Cluster Autoscaler or Karpenter.
