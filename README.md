# Educational-Platform
Globally accessible educational platform

# Front-End:
## 1. Overview
The front-end of the global educational platform is a Single Page Application (SPA), hosted on Amazon S3 and distributed globally using Amazon CloudFront. The platform is designed for minimal latency and high availability for users across regions such as Saudi Arabia and the United States. The Amazon Route 53 service is used for DNS management and routing traffic efficiently.

- SPA (React or Svelte).
- S3: Stores static website files.
- CloudFront: Distributes content globally with caching at edge locations.
- Route 53: Manages custom DNS for the website domain name.
- ACM: Secures communication over HTTPS.
- IAM: Access & security of resources.

![frontend diagram](/diagrams/frontend.png)

## 2. Front-End Deployment Components
### 2.1. S3 for Static Website Hosting
- S3 Storage: Stores the SPA and related static files (HTML, CSS, JavaScript, images).
- Scalable: S3 automatically scales to handle growing content and user traffic.
- Public Access: S3 is configured with public read permissions for static assets, which are cached via CloudFront.
- Storage Class: Using Intelligent-Tiering / Glacier could be used for less frequently accessed content achieving cost optimization.
- DR: Backing up the bucket for disaster recovery.
- KMS Encryption: Object encryption to satisfy compliance requirements, protecting user data at-rest.
### 2.2. CloudFront for Global Content Delivery
- Global Caching: CloudFront caches content at edge locations, reducing latency by serving assets from the nearest region (e.g., Bahrain for Saudi users, U.S. East for U.S. users).
- Fast Delivery: Static assets like images, CSS, and JavaScript are delivered quickly via CloudFront’s CDN.
- Cache Control: Custom caching policies are applied to optimize delivery speeds while ensuring freshness of content.
### 2.3. Route 53 for DNS Management
- DNS Routing: Route 53 maps the domain name (e.g., www.domain-name.com) to the CloudFront distribution.
- Latency-Based Routing: Automatically routes users to the nearest CloudFront edge location to minimize load times and improve performance.
### 2.4. ACM for HTTPS Encryption
- SSL/TLS: SSL certificates managed by AWS Certificate Manager (ACM) ensure secure HTTPS communication between users and CloudFront, protecting user data in-transit.

## 3. Key Features
### 3.1. Global Content Delivery and Performance Optimization
- CloudFront Caching: CloudFront’s global content delivery network (CDN) caches static assets at edge locations close to users, reducing latency and improving load times.
- Latency-Based Routing: Route 53 employs latency-based routing, directing users to the nearest CloudFront edge location, ensuring fast and responsive content delivery.
- Cache Control: Custom caching policies are applied via CloudFront, optimizing delivery speed while balancing content freshness. This reduces the load on origin servers and ensures fast access to frequently requested assets.
### 3.2. Scalability and High Availability
- Scalable Hosting with S3: The front-end is hosted on Amazon S3, which automatically scales to accommodate growing user traffic without the need for manual intervention. S3 ensures that large numbers of users can access the website simultaneously with minimal performance degradation.
- Global Availability via CloudFront: CloudFront ensures that content is accessible with minimal latency to users across the globe, providing high availability in both the US and Saudi Arabia regions.
### 3.3. Security
- SSL/TLS Encryption: AWS Certificate Manager (ACM) provides SSL/TLS certificates to encrypt communication between users and CloudFront, ensuring that user data is protected in-transit.
- Secure Access Control: IAM roles and policies manage access to resources, ensuring that only authorized services and users can interact with the front-end assets hosted on S3.
### 3.4. Cost Efficiency
- Intelligent-Tiering Storage: S3 Intelligent-Tiering storage is used to optimize costs by automatically moving less frequently accessed content to cheaper storage classes, while frequently accessed content remains readily available.
- Edge Caching: CloudFront’s edge caching further reduces costs by offloading traffic from S3, while delivering content from the nearest edge location to end-users.
### 3.5. Disaster Recovery and Data Protection
- S3 Data Backup: The static assets hosted on Amazon S3 are backed up, ensuring disaster recovery capabilities in case of unexpected failures.
- KMS Encryption: Data stored in S3 is encrypted using AWS Key Management Service (KMS), ensuring compliance and security for data at rest.


# Back-End: PHP App
## 1. Overview
The back-end of the global educational platform is a monolithic PHP application hosted on Amazon EC2 instances within an Auto Scaling Group (ASG), deployed across two regions: Saudi Arabia and the United States. This architecture is designed for high availability, scalability, and low latency. Amazon Route 53 is used for geo-based DNS routing, ensuring that users are directed to the nearest region's load balancer, while Elastic Load Balancers (ALB) distribute incoming traffic efficiently across EC2 instances.
Leveraging Virtual Private Cloud (VPC) to ensure network isolation, with security groups controlling traffic flow and subnets defining the network boundaries.

- Monolithic PHP Application: Hosted on EC2 instances.
- ASG: Ensures scalability and resilience of the application by automatically adjusting the number of EC2 instances.
- Elastic Load Balancer (ALB): Distributes incoming traffic across healthy EC2 instances.
- Route 53: Routes traffic based on the user's region (geo-based DNS routing).
- VPC: Provides network isolation and security.
- Security Groups: Controls inbound/outbound traffic to EC2 instances and ALB.
- ACM: Ensures secure HTTPS communication.
- IAM: Secures access to AWS resources.
- Shield & WAF: For web app protection.

![backend diagram](/diagrams/backend-php-app.png)
  
## 2. Back-End: PHP App Deployment Components
### 2.1. EC2 Instances and Auto Scaling Group (ASG)
- Storage: An EBS volume is attached to each EC2 instance for app local storage.
- Auto Scaling Group (ASG): The ASG automatically adjusts the number of EC2 instances based on demand (e.g., CPU utilization, request count).
- Health Checks: The ASG replaces unhealthy instances automatically, ensuring minimal downtime and high availability.
- Multi-AZ Deployment: EC2 instances in the ASG are distributed across multiple Availability Zones (AZs) within the region to improve fault tolerance and prevent single points of failure.

### 2.2. Elastic Load Balancer (ALB)
- SSL Termination: The ALB handles SSL termination, offloading encryption and decryption processes from EC2 instances. It uses certificates from AWS ACM for HTTPS.
- Health Checks: ALBs continuously check the health of EC2 instances, routing traffic only to healthy instances. If an instance becomes unhealthy, traffic is rerouted to healthy instances without any disruption to users.
- Cross-Region Load Balancing: Route 53's geo-based routing policy will ensure users are routed to their closest region.

### 2.3. Amazon Route 53 for Geo-Based Traffic Routing
- Geo DNS Routing: Route 53 uses geo-based routing policies to direct traffic to the closest available region.
- Users from Saudi Arabia will be routed to the Saudi region load balancer.
- Users from the U.S. will be routed to the U.S. region load balancer.

### 2.4. Virtual Private Cloud (VPC)
VPC Setup: Both regions (Saudi Arabia and U.S.) will have their own VPC to isolate network traffic. Each VPC will have:

- Private Subnets: For EC2 instances, ensuring that internal traffic is kept secure.
- Public Subnets: For the ALB, which needs to be publicly accessible to handle incoming traffic.
- Network Isolation: VPCs ensure that resources like EC2 instances and databases are isolated from external networks unless explicitly allowed.
- Peering (Optional): If there’s a need for secure communication between regions (e.g., replication or sharing data between the regions), VPC peering can be implemented to connect VPCs.

### 2.5. Security Groups & ACLs for Firewall features
ALB Security Group: Allows inbound traffic on port 80 (HTTP) and 443 (HTTPS) from the internet.
EC2 Instance Security Group: Allows inbound traffic from the ALB and outbound traffic to the internet (if required for updates or external API calls).
Network ACLs (optional): Can be used for an additional layer of security. Controlling both inbound and outbound traffic at the subnet level.

### 2.6. ACM for HTTPS Encryption
- SSL Certificates: AWS ACM is used to manage SSL certificates, ensuring secure HTTPS communication between users and the backend application via the ALB.
- SSL Termination: SSL termination is performed at the ALB, which decrypts traffic before forwarding it to EC2 instances.

### 2.7. DDoS Protection and WAF
- AWS Shield: AWS Shield is enabled to protect the CloudFront distribution & Route 53 from DDoS attacks, providing additional layers of protection.
- AWS WAF: AWS Web Application Firewall can be integrated to provide an additional layer of security by filtering out malicious traffic and protecting the platform from common threats like SQL injection, XSS, and others.

## 3. Key Features
### 3.1. Performance and Scalability
- Auto-Scaling: The ASG ensures that the application can scale up or down based on demand, providing cost-effective performance.
- High Availability: By deploying the application in multiple Availability Zones and regions, the architecture ensures fault tolerance and high availability.
- Global Traffic Optimization: Route 53 ensures that users are routed to the closest available load balancer, minimizing latency and ensuring optimal performance.
### 3.2. Security
- Encryption: All communication is encrypted using SSL/TLS via AWS ACM, protecting data in-transit.
- Server Encryption: Data stored on Amazon EBS volumes is encrypted, ensuring security for sensitive data at rest.
- IAM: IAM roles and security policies ensure that access to AWS resources is tightly controlled.
- Security Groups: Controls traffic flow to EC2 instances and ALBs, ensuring only authorized traffic reaches the application.
- DDoS Protection and WAF: Add layers of application protection to the platform.
### 3.3. Cost Optimization
- Elastic Scaling: The Auto Scaling Group helps minimize costs by adjusting the number of EC2 instances based on traffic needs.
- Multi-Region Deployment: By deploying in two regions, the application can balance traffic and optimize costs.
