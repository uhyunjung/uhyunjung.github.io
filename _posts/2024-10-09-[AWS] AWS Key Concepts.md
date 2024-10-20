---
title: '[AWS] AWS Key Concepts'
date: 2024-10-09 18:10:00 +0900
categories: [인프라/클라우드, AWS]
tags: [AWS]
math: true
mermaid: true
---

## Module 1 (Introduction)
■ Six advantages of Cloud Computing

■ The Well Architected Framework(WAF)

■ AWS Global Infrastructure
  - Data Center
  - Availability Zone
  - Region
  - Edge Location

## Module 2 (The Simiplest Architecture)
■ S3
  - Bucket, Object
  - Access Control
    - ACL
    - Bucket Policy
    - CORS(Cross-Origin Resource Sharing)
  - Versioning
  - Multipart Upload
  - Transfer Acceleration
  - Snowball, Snowmobile

■ Glacier  
  - Vault, Archive
  - Retrieval options
    - Expedited
    - Standard
    - Bulk

■ Storage Class + S3 Analytics + S3 Lifecycle  
  - S3 Standard
  - S3 Standrad IA
  - S3 One Zone IA
  - S3 Intelligent Tiering
  - Glacier / Deep Archive

■ Choosing a Region

## Module 3 (Adding a Compute Layer)
■ EC2
  - AMI
  - User Data
  - Instance Metadata
  - Storing Data
    - EBS(Elastic Block Storage)
      - Volume Type (SSD vs HDD)
      - EBS-Optimized
        - GP2(General Purpose SSD 2)
        - GP3(General Purpose SSD 3)
        - PIOPS(Provisioned IOPS SSD)
          - custom - SSD DISK SIZE / IO Speed
      - EFS / FSx
  - Instance Size & Type
  - Pricing
    - On Demand
    - Reserved
    - Spot
    - Dedicated Instance
    - Dedicated Hosts
  - Tag
  - Placement Group
    - Cluster
    - Spread
    - Partition
  - Limit increase
    - Default limit of 20 instances per region(Case open can be increase)

## Module 4 (Adding a Database Layer)
■ RDS(Relational Database Service)
  - Amazon Aurora
  - PostgreSQL
  - MSSQL
  - MySQL
  - Maria DB
  - Oracle

■ DynamoDB
  - Global Table
  - Consistency Option
    - Eventually
    - Strongly

■ DMS(Data Migration Service)
  - Snoball Edge
  - Schema Conversion Toll (SCT)

## Module 5 (Networking - Part I)
■ VPC
  - Multi-VPC
  - Multi-Account

■ VPC Limits
  - dfault vpc 5 per Region(Case open can be increased)

■ CIDR

■ Subnet
  - Public
    - Internet Gateway
  - Private
    - NAT Gateway(vs. NAT Instance)

■ Routing Table

■ ENI(Elastic Network Interface)

■ EIP(Elastic IP)

■ Security
  - Security Group
    - Security Group Chaining
  - Network ACL

## Module 6 (Networking - Part II)
■ VGW(Virtual Private Gateway)
  - VPN(Virtual Private Network)
  - CGW(Customer Gateway)
  - Direct Connect(DX)

■ VPC Peering(VPC <-> VPC)

■ Transit Gateway

■ VPC Endpoint
  - Interface
  - Gateway

■ ELB
  - Options
    - Application(ALB)
    - Network(NLB)
    - Classic(CLB)

■ High Availability
  - ELB
  - Route53
    - Traffic Flow
    - Latency Based Routing
    - Geo DNS(Geographic Location)
    - Private DNS
    - Health Checks and Monitoring
    - DNS Failover
    - Weighted Round Robin(WRR)

## Module 7 (IAM)
■ Account Root User

■ IAM
  - User / Federated User(AD, LDAP)
  - Group
  - Policy
    - Resource-Based / Identity-Based
    - Manage / Inline
  - Role
    - STS(Security Token Service)
    - Identity Broker
    - SAML

■ Cognito(mobile/app User Pool Federated Identities)
  - User Pool

■ Landing Zone

■ Multi Accounts
  - Cross-account access
  - Organization
    - Consolidated Billing
    - Root, OU, Policy

## Module 8 (Elasticity, HA and Monitoring)
■ Elasticity
  - Time-Bases, Volume-Based, Predictive-Based

■ Monitoring
  - Cost Explorer
  - CloudWatch
    - Metrics
    - Logs
    - Alarm
    - Event
    - Rule
    - Target
  - CloudTrail
  - VPC Flow Logs

■ Auto Scaling
  - Scheduled/Dynamic/Predictive
  - Auto Scaling Group
    - Min/Max/Desired

■ Scaling Database
  - RDS
    - Read Replica
    - Sharding
  - DynamoDB
    - Auto Scaling/On Demand
    - Adaptive Capacity

## Module 9 (Automation)
■ CloudFormation
  - Infrastructure as Code(IaC)
  - Change set

■ Quick Start

■ System Manager
  - Run Command
  - Maintenance Windows
  - Patch Management
  - State Manager
  - Session Manager
  - Inventory

■ OpsWorks

■ Beanstalk

## Module 10 (Caching)
■ CloudFront
  - Expire contents
    - TTL
    - Change object name
    - Invalidate Object

## Module 11 (Decoupled Architecture)
■ SQS(Simple Queue Service)
  - Type
    - Standard vs FIFO
  - Features
    - Dead letter queue
    - Visibility timeout
    - Long polling(vs Short polling)

■ SNS(Simple Notification Service)
  - Lambda
  - SQS
  - HTTP / HTTPS
  - Email
  - SMS

## Module 12 (Microservices and Serverless)
■ Microservices
  - Container vs Virtual Machines
    - ECR(Elastic Container Registry)
    - ECS(Elastic Container Service)
    - EKS(Elastic Kubernetes Service)

## Module 13 (RTO/RPO and Backup Recovery)
■ Availability Concepts
  - High Availability
  - Backup
  - Disaster Recovery

■ RTO, RPO

■ AWS Services for DR
  - Storage
    - S3(Cross-region Replication)
    - Glacier
    - EBS(Snapshot, Copy)
    - Snoball
    - DataSync
  - Compute
    - Custom AMI
    - Custom Container Image
  - Networking
    - Route53(Load Balancing, Failover)
    - ELB(Load Balancing, FailoveR)
    - VPC
    - Direct Connect(Backup Replication)