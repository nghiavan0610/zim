# Cost-Optimized Video Course Platform Architecture Analysis

## 1. System Overview

### Global Edge Layer

- **Cloudflare Network**
  - Primary Role: Provides global DNS, CDN, DDoS protection, and load balancing
  - Benefits: Reduces latency, improves security, and provides the first line of defense
  - Alternative: AWS CloudFront + Shield + WAF, but Cloudflare offers better price-performance ratio

### CI/CD Pipeline

- **Multiple Options Available:**
  - Docker, Github Actions, GitLab CI/CD, Bitbucket Pipelines
  - Optional ArgoCD for GitOps and automated rollbacks
  - Optional Harbor for private container registry
  - Benefits: Flexibility in choosing tools based on team expertise
  - Alternative: AWS ECR, but self host container registry are more cost-effective
  - Alternative: AWS CodePipeline, but chosen solutions offer better developer experience and are more cost-effective

### Network Security Layer

- **Multi-layered approach:**
  - Self-hosted NGINX WAF for cost-effective application protection
  - Optional Route 53 for DNS management
  - VPN/Tunnel options for secure access
  - Security Groups for instance-level security
  - Alternative: AWS WAF, but self-hosted NGINX WAF provides better cost control

### Infrastructure Management

- **IAM & Access Control:**
  - Minimal IAM roles following principle of least privilege
  - Optional CloudTrail for security/audit logging of all AWS API calls
  - Alternative: Third-party IAM solutions, but native AWS IAM is sufficient and cost-effective
  - Alternative: Keycloak/OpenLDAP: Provides identity management for your applications and infrastructure

### VPC Architecture

- **Cost-optimized network design:**
  - VPC & Public + Private subnets for security
  - NAT Gateway for outbound traffic
  - Security Groups without NACLs for simplicity
  - Alternative: AWS Transit Gateway setup, but current design is sufficient for initial scale

## 2. Core Components

### Compute Infrastructure

- **Kubernetes-based deployment:**
  - Self-managed Kubernetes with Rancher
  - Flexible Spot + Spot Fleet, On-Demand instances for cost optimization
  - Multi-cluster setup (Production, Staging, Dev)
  - Alternative: EKS, but self-managed K8s offers better cost control
  - Alternative: k8s Management - k9s, lens but Rancher is good for enterprise environments, multi-cluster management, and role-based access control

### Microservices Architecture

- **Key Services:**
  1. API Gateway Service: Entry point for all API requests
  2. User Service: Internal Built Authentication and user management
  3. Websocket: Real-time data via WebSocket
  4. Video Streaming: Bunny or AWS Elemental MediaConvert, MediaLive, MediaPackage
  - Alternative: AWS Cognito for Authentication
  - Alternative: Monolithic architecture, but microservices offer better scalability

### Data Services

- **Distributed Data Storage:**
  1. NoSQL Cluster (MongoDB/ScyllaDB)
     - Use: High-throughput operations, flexible schema
     - Alternative: DynamoDB could be a better choice despite higher costs because:
       - It reduces operational overhead (backups, patching, high availability)
       - Offers better reliability guarantees
       - Provides read replicas and easier scaling
       - Simplifies disaster recovery
  2. Aurora RDS (PostgreSQL)
     - Use: ACID-compliant transactions
     - Alternative: Self-host Postgres, MySQL, SQLite, but RDS could be a better choice despite higher costs because:
       - It reduces operational overhead (backups, patching, high availability)
       - Offers better reliability guarantees
       - Provides read replicas and easier scaling
       - Simplifies disaster recovery
  3. Redis Caching
     - Use: High-speed data access, session management
     - Alternative: Memcached, but Redis offers more features
  4. Object Storage (R2)
     - Use: Static content, backups
     - Alternative: S3, MinIO
     - Could use Cloudflare R2 for better cost because R2 have no egress fees (pay only for storage & operations), with S3 charges for storage + egress bandwidth
  5. Message Broker (Kafka/RabbitMQ)
     - Use: Async communication, event streaming
     - Alternative: AWS MSK, but self-hosted offers better cost control
     - Alternative: Redis Queue
       - Use Redis Queue if you need a simple, fast, in-memory queue for short-lived tasks or real-time processing
       - Use Kafka if you need high-throughput event streaming, logs, analytics, or distributed systems that process massive amounts of events
       - Use RabbitMQ if you need a reliable message broker for background jobs, task queues, or request-response patterns

## 3. Monitoring & Observability

### Comprehensive Monitoring Stack:

1. **Metrics Stack:**

   - Grafana for visualization
   - Prometheus for metrics collection
   - Thanos for long-term storage
   - Alternative: CloudWatch, but current stack offers better features/cost ratio

2. **Logging/Tracing:**

   - OpenTelemetry for instrumentation
   - Jaeger for distributed tracing
   - Loki for log aggregation
   - Alternative: AWS X-Ray, but chosen stack provides better visibility

3. **Error Tracking:**
   - Sentry for error monitoring
   - Telegram, Slack for alerts
   - Alternative: CloudWatch Alerts, but current setup offers better developer experience

## 4. Cost Estimates

### Current Setup (1000 Concurrent Users, 65000 Daily Users)

#### Compute Costs (Monthly)

- **Kubernetes Nodes (EC2)**
  - 12 x t3.xlarge Spot instances (4 vCPU, 16GB RAM) for applications
    - Average Spot price: ~$0.05/hour
    - Cost: 12 × $0.05 × 24 × 30 = ~$432
  - 3 x t3.2xlarge On-Demand instances for critical components
    - On-Demand price: ~$0.34/hour
    - Cost: 3 × $0.34 × 24 × 30 = ~$245
  - EBS costs:
    - Root volumes: 15 nodes × 20GB = 300GB
    - NoSQL Database: 2 nodes × 200GB = 400GB
    - Total EBS: 700GB × $0.08/GB = ~$56/month
  - Total Compute: ~$733/month

#### Storage Costs (Monthly)

- **RDS Aurora PostgreSQL** (Single AZ)

  - db.r6g.2xlarge: ~$500/month
  - 1TB storage: ~$100/month
  - I/O costs:
    - Estimated 2M IOPS/day
    - ~$0.20 per 1M I/O
    - Cost: ~$12/month
  - Total RDS: ~$612/month

- **Object Storage (Cloudflare R2)**
  - Storage: $0.015/GB/month
  - Operations: $4.50/million Class A operations
  - Estimated 1TB storage: ~$15/month
  - Estimated operations: ~$20/month
  - Total R2: ~$35/month

#### Network Costs (Monthly)

- **Cloudflare Enterprise**

  - Starting from $200/month
  - DDoS protection included
  - CDN included

- **Data Transfer**
  - Internet egress: Minimal (covered by R2 and Cloudflare)
  - Single AZ, so no inter-AZ costs

#### Total Monthly Cost: ~$1,580

## 5. Security

### Protecting Video Content from Unauthorized Access

- **Token-Based Authentication**: Internal built User Service Authentication/Authorization/RBAC with JWT, Guard that must be included in requests to access video content.

- **Secure Video Delivery**: Use **Bunny.net** for video streaming, which provides secure delivery options. Bunny supports signed URLs, allowing you to create temporary access links for your video content. This ensures that only users with valid tokens can access the streams. Or Consider using **AWS Elemental MediaPackage**.

- **Encryption**: Encrypt video files stored in **Amazon S3**. This ensures that even if someone gains access to the storage, they cannot play the videos without the decryption keys. Consider: use **AWS Key Management Service (KMS)** to manage encryption keys securely.

- **Access Control**: Set up **AWS IAM** policies to restrict access to S3 buckets containing video files. Only authorized services (like MediaPackage) and authenticated users should have access. Use signed URLs or signed cookies to control access to video content, which can expire after a certain time.

- **Dynamic Watermarking**: Consider implementing dynamic watermarking for videos to deter unauthorized sharing by embedding user-specific information into the video stream.

### Network Security

- **Multi-Layered Security Approach**: Use a self-hosted NGINX WAF or **AWS WAF** to protect against common web exploits and attacks. Leverage services like **Cloudflare** or **AWS Shield** for DDoS protection.

- **Secure Access**: Implement VPN or secure tunnels for administrative access to your infrastructure, ensuring that sensitive operations are conducted over secure channels. Use AWS Security Groups to control inbound and outbound traffic to your instances.

### Infrastructure Management

- **IAM & Access Control**: Implement minimal IAM roles for users and services, ensuring that they only have access to the resources necessary for their function. Enable **AWS CloudTrail** for logging all API calls, providing an audit trail for security and compliance.

- **VPC Architecture**: Design your VPC with public and private subnets to isolate sensitive resources and control access. Use a NAT Gateway for outbound internet traffic from private subnets.

### Monitoring and Incident Response

- **Monitoring Solutions**: Use **AWS CloudWatch** to monitor system performance, set alarms for unusual activity, and track resource utilization. Implement logging solutions using **OpenTelemetry**, **Jaeger**, and **Loki** for distributed tracing and log aggregation.

- **Incident Response Plan**: Develop an incident response plan that outlines steps to take in the event of a security breach, including communication protocols and recovery procedures. Conduct regular security assessments and penetration testing to identify vulnerabilities and improve your security posture.
