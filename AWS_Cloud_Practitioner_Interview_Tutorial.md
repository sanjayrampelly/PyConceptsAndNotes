# AWS Cloud Practitioner Certification - Quick Interview Recap

## 1. Cloud Computing Fundamentals

**What is Cloud Computing?**
Cloud computing is the on-demand delivery of IT resources over the internet with pay-as-you-go pricing.

**Key Characteristics:**
- On-demand self-service
- Broad network access
- Resource pooling
- Rapid elasticity
- Measured service

**Example:** Instead of buying physical servers for $10,000, you rent AWS EC2 instances for $50/month and scale up/down as needed.

**Interview Q&A:**

**Q: What are the main cloud deployment models?**
**A:** 
- **Public Cloud:** Resources owned by third-party provider (AWS, Azure, GCP). Example: Netflix using AWS infrastructure.
- **Private Cloud:** Dedicated infrastructure for single organization. Example: Bank's internal data center with cloud capabilities.
- **Hybrid Cloud:** Combination of public and private. Example: Company keeping sensitive data on-premises while using AWS for web applications.

**Q: What are the advantages of cloud computing?**
**A:**
1. **Trade capital expense for variable expense** - Pay only for what you use instead of investing in data centers
2. **Massive economies of scale** - Lower pay-as-you-go prices due to AWS's scale
3. **Stop guessing capacity** - Scale up/down based on actual demand
4. **Increase speed and agility** - Resources available in minutes
5. **Stop spending money on data centers** - Focus on customers, not infrastructure
6. **Go global in minutes** - Deploy applications in multiple regions worldwide

---

## 2. AWS Global Infrastructure

**Components:**
- **Regions:** Geographic areas (e.g., us-east-1, eu-west-1) - 33+ regions
- **Availability Zones (AZs):** Isolated data centers within a region (2-6 per region)
- **Edge Locations:** CDN endpoints for CloudFront (400+ locations)

**Example:** A company in New York uses us-east-1 region with 3 AZs for high availability. If one AZ fails, the application continues running in other AZs.

**Interview Q&A:**

**Q: How do you choose an AWS Region?**
**A:** Consider four factors:
1. **Compliance/Data Governance:** Some countries require data to stay within borders (e.g., GDPR in EU)
2. **Proximity to users:** Lower latency - choose region closest to customers
3. **Available services:** Not all services available in all regions
4. **Pricing:** Costs vary by region (us-east-1 typically cheapest)

**Q: What is an Availability Zone?**
**A:** An AZ is one or more discrete data centers with redundant power, networking, and connectivity. AZs are physically separated (miles apart) but connected with high-bandwidth, low-latency networking. This design protects against single points of failure.

---

## 3. IAM (Identity and Access Management)

**Core Concepts:**
- **Users:** Individual people or applications
- **Groups:** Collections of users
- **Roles:** Permissions for AWS services
- **Policies:** JSON documents defining permissions

**Example:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```

**Interview Q&A:**

**Q: What is the principle of least privilege?**
**A:** Grant only the minimum permissions needed to perform a task. For example, if a developer only needs to read S3 objects, don't give them delete permissions. Start with minimal permissions and add more as needed.

**Q: What is the difference between IAM Users and IAM Roles?**
**A:**
- **IAM Users:** Long-term credentials for specific people/applications. Example: Developer John has username/password.
- **IAM Roles:** Temporary credentials assumed by users, applications, or services. Example: EC2 instance assumes a role to access S3 without storing credentials.

**Q: What is MFA and why is it important?**
**A:** Multi-Factor Authentication adds an extra security layer requiring both password and physical device (phone, hardware token). Critical for root account and privileged users. Example: Root account with password + Google Authenticator code.

**Q: What is the AWS root account?**
**A:** The root account is created when you first set up AWS. It has complete access to all resources. Best practices:
- Don't use for everyday tasks
- Enable MFA
- Create IAM users for daily work
- Delete root access keys

---

## 4. EC2 (Elastic Compute Cloud)

**Instance Types:**
- **General Purpose (t3, m5):** Balanced compute, memory, networking
- **Compute Optimized (c5):** High-performance processors
- **Memory Optimized (r5):** Large datasets in memory
- **Storage Optimized (i3):** High sequential read/write
- **Accelerated Computing (p3):** GPU instances

**Example:** E-commerce site uses t3.medium for web servers, r5.large for database, and c5.xlarge for data processing.

**Interview Q&A:**

**Q: What are the EC2 pricing models?**
**A:**
1. **On-Demand:** Pay per hour/second, no commitment. Use for: short-term, unpredictable workloads. Example: Testing environment.
2. **Reserved Instances:** 1 or 3-year commitment, up to 75% discount. Use for: steady-state applications. Example: Production database.
3. **Spot Instances:** Bid for unused capacity, up to 90% discount, can be terminated. Use for: fault-tolerant workloads. Example: Batch processing.
4. **Dedicated Hosts:** Physical server dedicated to you. Use for: compliance requirements or licensing.
5. **Savings Plans:** Flexible pricing model, commitment to consistent usage.

**Q: What is EC2 Auto Scaling?**
**A:** Automatically adjusts the number of EC2 instances based on demand. Example: E-commerce site scales from 2 instances to 10 during Black Friday, then back to 2. Benefits: cost optimization and high availability.

**Q: What is an AMI?**
**A:** Amazon Machine Image - a template containing OS, applications, and configurations to launch EC2 instances. Example: Create custom AMI with pre-installed web server and application, then launch 10 identical instances.

---

## 5. Storage Services

### S3 (Simple Storage Service)

**Storage Classes:**
- **S3 Standard:** Frequent access, 99.99% availability
- **S3 Intelligent-Tiering:** Automatic cost optimization
- **S3 Standard-IA:** Infrequent access, lower cost
- **S3 One Zone-IA:** Single AZ, even lower cost
- **S3 Glacier:** Archival, retrieval in minutes to hours
- **S3 Glacier Deep Archive:** Lowest cost, 12-hour retrieval

**Example:** Store website images in S3 Standard, monthly reports in S3-IA, 7-year compliance logs in Glacier Deep Archive.

**Interview Q&A:**

**Q: What is S3 and what are its use cases?**
**A:** S3 is object storage for any type of data. Use cases:
- Backup and disaster recovery
- Static website hosting
- Data lakes for analytics
- Application hosting (images, videos)
- Archive compliance data

**Q: How does S3 ensure durability and availability?**
**A:** 
- **Durability:** 99.999999999% (11 nines) - data replicated across multiple devices and facilities
- **Availability:** 99.99% - designed to sustain loss of 2 facilities simultaneously

**Q: What is S3 versioning?**
**A:** Keeps multiple versions of an object. If you accidentally delete or overwrite, you can restore previous versions. Example: Document v1, v2, v3 all stored. Delete v3, easily restore v2.

### EBS (Elastic Block Store)

**Interview Q&A:**

**Q: What is EBS and how is it different from S3?**
**A:**
- **EBS:** Block storage attached to EC2 instances, like a hard drive. Use for: databases, file systems. Tied to single AZ.
- **S3:** Object storage accessed via API. Use for: static content, backups. Regional service.
Example: Database runs on EBS volume, database backups stored in S3.

**Q: What is an EBS snapshot?**
**A:** Point-in-time backup of EBS volume stored in S3. Incremental - only changed blocks saved. Use for: disaster recovery, creating new volumes, migrating across AZs.

---

## 6. Database Services

### RDS (Relational Database Service)

**Supported Engines:** MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Amazon Aurora

**Example:** E-commerce site uses RDS MySQL with Multi-AZ deployment for automatic failover.

**Interview Q&A:**

**Q: What are the benefits of RDS over running database on EC2?**
**A:**
- Automated backups and patching
- Multi-AZ for high availability
- Read replicas for scalability
- Automatic failover
- Managed service - AWS handles infrastructure
Example: RDS automatically patches MySQL, creates daily backups, and fails over to standby in different AZ if primary fails.

**Q: What is RDS Multi-AZ?**
**A:** Synchronous replication to standby instance in different AZ. If primary fails, automatic failover to standby (1-2 minutes). Use for: production databases requiring high availability. Not for scaling reads - use Read Replicas.

### DynamoDB

**Interview Q&A:**

**Q: What is DynamoDB?**
**A:** Fully managed NoSQL database with single-digit millisecond latency. Key features:
- Serverless - no servers to manage
- Automatic scaling
- Built-in security and backup
Example: Mobile app storing user profiles, gaming leaderboards, IoT sensor data.

**Q: When would you choose DynamoDB over RDS?**
**A:**
- **DynamoDB:** Need massive scale, flexible schema, key-value access, serverless. Example: Social media app with millions of users.
- **RDS:** Need complex queries, joins, transactions, SQL. Example: Financial application with complex reporting.

---

## 7. Networking Services

### VPC (Virtual Private Cloud)

**Components:**
- **Subnets:** Public (internet-facing) and Private (internal)
- **Internet Gateway:** Connects VPC to internet
- **NAT Gateway:** Allows private subnet to access internet
- **Security Groups:** Virtual firewall for instances
- **Network ACLs:** Subnet-level firewall

**Example:** Web servers in public subnet, databases in private subnet. Security group allows port 443 from internet to web servers, port 3306 from web servers to database.

**Interview Q&A:**

**Q: What is the difference between Security Groups and NACLs?**
**A:**
- **Security Groups:** 
  - Instance level
  - Stateful (return traffic automatically allowed)
  - Only allow rules
  - Evaluate all rules
- **Network ACLs:**
  - Subnet level
  - Stateless (must explicitly allow return traffic)
  - Allow and deny rules
  - Process rules in order

**Q: What is a VPC?**
**A:** Logically isolated section of AWS cloud where you launch resources. You control IP ranges, subnets, route tables, and gateways. Example: Create VPC with 10.0.0.0/16, public subnet 10.0.1.0/24, private subnet 10.0.2.0/24.

### Route 53

**Interview Q&A:**

**Q: What is Route 53?**
**A:** AWS's DNS service. Features:
- Domain registration
- DNS routing
- Health checks
- Multiple routing policies (simple, weighted, latency, failover, geolocation)
Example: Route users to nearest region using latency-based routing.

---

## 8. Content Delivery & Edge Services

### CloudFront

**Interview Q&A:**

**Q: What is CloudFront and why use it?**
**A:** Content Delivery Network (CDN) that caches content at edge locations worldwide. Benefits:
- Reduced latency - content closer to users
- Reduced load on origin servers
- DDoS protection
- HTTPS support
Example: Website hosted in us-east-1, users in Tokyo get content from Tokyo edge location (20ms vs 200ms).

---

## 9. Monitoring & Management

### CloudWatch

**Interview Q&A:**

**Q: What is CloudWatch?**
**A:** Monitoring service for AWS resources and applications. Features:
- Metrics (CPU, network, disk)
- Logs (application and system logs)
- Alarms (trigger actions based on thresholds)
- Dashboards (visualize metrics)
Example: Create alarm when EC2 CPU > 80% for 5 minutes, trigger Auto Scaling to add instances.

### CloudTrail

**Interview Q&A:**

**Q: What is CloudTrail?**
**A:** Records AWS API calls for auditing and compliance. Tracks:
- Who made the call
- When it was made
- What action was performed
- Which resources were affected
Example: Investigate who deleted S3 bucket last Tuesday at 3 PM.

**Q: What's the difference between CloudWatch and CloudTrail?**
**A:**
- **CloudWatch:** Performance monitoring (CPU, memory, application metrics)
- **CloudTrail:** Audit trail of API calls (who did what, when)

---

## 10. Security & Compliance

### AWS Shared Responsibility Model

**Interview Q&A:**

**Q: Explain the AWS Shared Responsibility Model.**
**A:**
- **AWS Responsibility (Security OF the cloud):**
  - Physical security of data centers
  - Hardware and infrastructure
  - Network infrastructure
  - Virtualization layer

- **Customer Responsibility (Security IN the cloud):**
  - Data encryption
  - IAM and access management
  - OS patching (for EC2)
  - Application security
  - Network configuration

Example: AWS secures the physical server, you secure your application code and data.

**Q: How does responsibility differ for different service types?**
**A:**
- **IaaS (EC2):** Customer manages OS, applications, data
- **PaaS (RDS):** AWS manages OS and database engine, customer manages data
- **SaaS (S3):** AWS manages everything except data and access policies

### AWS Compliance Programs

**Interview Q&A:**

**Q: What compliance certifications does AWS have?**
**A:** AWS complies with:
- PCI DSS (payment card data)
- HIPAA (healthcare data)
- SOC 1, 2, 3 (security controls)
- ISO 27001 (information security)
- GDPR (EU data protection)
- FedRAMP (US government)

---

## 11. Pricing & Billing

### AWS Pricing Models

**Interview Q&A:**

**Q: What are the three fundamental pricing characteristics?**
**A:**
1. **Compute:** Pay for compute time (per hour/second)
2. **Storage:** Pay for data stored (per GB)
3. **Data Transfer:** Pay for data OUT of AWS (data IN is free)

**Q: What is AWS Free Tier?**
**A:** Three types:
- **Always Free:** DynamoDB 25 GB, Lambda 1M requests/month
- **12 Months Free:** 750 hours EC2 t2.micro, 5 GB S3 Standard
- **Trials:** Short-term free trials for specific services

**Q: What tools help manage AWS costs?**
**A:**
- **AWS Budgets:** Set custom cost/usage budgets with alerts
- **Cost Explorer:** Visualize and analyze costs over time
- **AWS Cost and Usage Report:** Detailed billing data
- **AWS Pricing Calculator:** Estimate costs before deployment

---

## 12. Support Plans

**Interview Q&A:**

**Q: What are the AWS Support Plans?**
**A:**
1. **Basic:** Free, 24/7 customer service, documentation, forums
2. **Developer:** $29/month, business hours email support, 12-24 hour response
3. **Business:** $100/month, 24/7 phone/chat, 1-hour response for urgent issues, full Trusted Advisor
4. **Enterprise:** $15,000/month, 15-minute response for critical issues, Technical Account Manager (TAM), Concierge Support

---

## 13. Additional Core Services

### Lambda

**Interview Q&A:**

**Q: What is AWS Lambda?**
**A:** Serverless compute service - run code without managing servers. Features:
- Pay only for compute time used
- Automatic scaling
- Event-driven
- Supports multiple languages
Example: Process image uploads to S3 - Lambda function automatically triggers, creates thumbnail, stores in another bucket.

### Elastic Beanstalk

**Interview Q&A:**

**Q: What is Elastic Beanstalk?**
**A:** Platform as a Service (PaaS) for deploying applications. You upload code, Beanstalk handles:
- Capacity provisioning
- Load balancing
- Auto scaling
- Application health monitoring
Example: Deploy Node.js app - Beanstalk creates EC2 instances, load balancer, Auto Scaling group automatically.

### SNS (Simple Notification Service)

**Interview Q&A:**

**Q: What is SNS?**
**A:** Pub/sub messaging service for sending notifications. Use cases:
- Send emails/SMS
- Push notifications to mobile
- Trigger Lambda functions
- Fan-out messages to multiple subscribers
Example: Order placed → SNS topic → Email to customer, SMS to warehouse, Lambda to update inventory.

### SQS (Simple Queue Service)

**Interview Q&A:**

**Q: What is SQS and when would you use it?**
**A:** Fully managed message queue for decoupling applications. Benefits:
- Asynchronous processing
- Fault tolerance
- Scalability
Example: Web app receives 1000 orders/second → Places in SQS queue → Worker processes at own pace (100/second). If worker fails, messages remain in queue.

**Q: What's the difference between SNS and SQS?**
**A:**
- **SNS:** Push-based, pub/sub, multiple subscribers receive same message
- **SQS:** Pull-based, queue, one consumer processes each message

---

## 14. Migration & Transfer

**Interview Q&A:**

**Q: What is AWS Snow Family?**
**A:** Physical devices for data migration:
- **Snowcone:** 8 TB, portable
- **Snowball:** 80 TB, rugged
- **Snowmobile:** 100 PB, shipping container
Use when: Network transfer would take weeks/months or cost too much.
Example: Migrate 50 TB data - would take 100 days over 50 Mbps connection, Snowball takes 1 week.

**Q: What is AWS Database Migration Service (DMS)?**
**A:** Migrate databases to AWS with minimal downtime. Supports:
- Homogeneous (Oracle to Oracle)
- Heterogeneous (Oracle to Aurora)
- Continuous replication
Example: Migrate on-premises MySQL to RDS MySQL while keeping source database operational.

---

## 15. Well-Architected Framework

**Interview Q&A:**

**Q: What are the six pillars of the Well-Architected Framework?**
**A:**
1. **Operational Excellence:** Run and monitor systems
   - Example: Automate deployments, use CloudWatch for monitoring

2. **Security:** Protect information and systems
   - Example: Enable MFA, encrypt data at rest and in transit

3. **Reliability:** Recover from failures, meet demand
   - Example: Multi-AZ deployment, automated backups

4. **Performance Efficiency:** Use resources efficiently
   - Example: Choose right instance types, use caching

5. **Cost Optimization:** Avoid unnecessary costs
   - Example: Use Reserved Instances, right-size resources

6. **Sustainability:** Minimize environmental impact
   - Example: Use efficient instance types, optimize storage

---

## Key Exam Tips

1. **Understand the Shared Responsibility Model** - Know what AWS manages vs. what you manage
2. **Know when to use each service** - S3 vs EBS, RDS vs DynamoDB, etc.
3. **Memorize key service use cases** - CloudFront for CDN, Route 53 for DNS
4. **Understand pricing models** - On-Demand vs Reserved vs Spot
5. **Know security best practices** - MFA, least privilege, encryption
6. **Understand high availability** - Multi-AZ, Auto Scaling, Load Balancing
7. **Know the difference between regions, AZs, and edge locations**
8. **Understand basic networking** - VPC, subnets, security groups

---

## Common Scenario-Based Questions

**Q: A company needs to host a static website. What AWS services would you recommend?**
**A:** 
- Store website files in S3 (enable static website hosting)
- Use CloudFront for global content delivery
- Use Route 53 for DNS
- Benefits: Highly available, scalable, cost-effective, no servers to manage

**Q: An application needs to process uploaded images. How would you design this?**
**A:**
- Users upload images to S3
- S3 event triggers Lambda function
- Lambda processes image (resize, compress)
- Store processed image in another S3 bucket
- Benefits: Serverless, automatic scaling, pay per use

**Q: A database needs high availability. What would you recommend?**
**A:**
- Use RDS with Multi-AZ deployment
- Automatic failover to standby in different AZ
- Automated backups to S3
- Read replicas for read scaling
- Benefits: 99.95% availability SLA, automatic recovery

---

Good luck with your interview! Focus on understanding concepts and real-world use cases rather than memorizing facts.
