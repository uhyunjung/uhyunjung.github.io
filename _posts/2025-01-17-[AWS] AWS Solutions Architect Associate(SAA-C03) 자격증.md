---
title: '[AWS] AWS Solutions Architect Associate(SAA-C03) 자격증'
date: 2025-01-17 18:10:00 +0900
categories: [인프라/클라우드, AWS]
tags: [AWS]
math: true
mermaid: true
---

## Module 1 (Introduction)
■ Cloud Computing Deploy Model
  - **Cloud**
  - On-Premise(Private Cloud)
  - Hybrid

■ Six Advantages of Cloud Computing
  - **Trade capital expense (CAPEX) for operational expense(OPEX)** : 선행 비용(고정 비용)을 가변 비용으로 대체
    - eliminates many of the costs of building and maintaining on-premises data centers
    - Elimination of expenses for running and maintaining data centers
    - The AWS Cloud pricing model differ from the traditional on-premises storage pricing model
  - **Benefit from massive economies of scale** : 거대한 규모의 경제로 얻는 이점
    - Ability to offer lower variable costs as a result of high purchase volumes
    - Lower variable costs over fixed costs
    - High economies of scale
  - **Stop guessing capacity** : 용량 추정 불필요
    - Compute capacity that is adjusted on demand
    - No longer having to guess what capacity will be required
    - Reduced or eliminated tasks for hardware troubleshooting, capacity planning, and procurement
    - The ability to use the **pay-as-you-go model**
      - There are **no upfront cost commitments**
      - There are no infrastructure operating costs X -> EC2에 해당, AWS에 해당하지 않음
    - Elasticity
    - Auto Scaling
  - **Increase global reach, speed and agility** : 속도 및 민첩성 개선
    - Migration
    - A company wants to know more about the benefits offered by cloud computing. The company wants to understand the operational advantage of agility. -> The ability to provision and deprovision resources quickly with minimal effort
    - Ability to quickly change required capacity
  - **Stop spending money running and maintaining data centers** : 데이터 센터 운영 및 유지 관리에 비용 투자 불필요
    - Elimination of expenses for running and maintaining data centers
  - **Ability to Deploy globally in minutes** : 몇 분 안에 전 세계에 배포
    - Migration
    - The ability to deploy to AWS on a global scale

■ 클라우드로 해결한 문제점
  - **Flexibility**
    - AWS enables capacity to be adjusted on demand
  - **Cost-Effectiveness**
    - Increased workforce productivity
    - Faster product launches (Business Agility)
    - No Infrastructure Operating Costs
  - **`Elasticity`**
    - Resource Elasticity : An AWS value proposition that describes a user’s ability to scale infrastructure based on demand
    - Time-Bases, Volume-Based, Predictive-Based
    - The ability to rightsize resources as demand shifts
    - How easily resources can be procured when they are needed
    - Helps users eliminate underutilized CPU capacity
  - **High Availability and Fault Tolerance**
    - Operational resilience (운영 회복력)
    - An architecture's ability to withstand failures with minimal downtime
    - Is shown by an architecture's ability to withstand failures with minimal downtime
  - **`Agility`**
    - The speed at which AWS resources are implemented
    - The ability to experiment quickly

■ **The Well Architected Framework(WAF)**
  - **Operational Excellence(운영 우수성)** : **Anticipate failure, 프로세스 개선**
  - **Security(보안)** : **암호화, 데이터 무결성 검사**
  - **Reliability(안정성/신뢰성)** : **기능 수행, 복구, 가용성(High Availability)**
  - **Performance Efficiency(성능 효율성)** : **Serverless, 고급 기술**
  - **`Cost Efficiency = Cost Optimization(비용 최적화)`**
  - Sustainability(지속 가능성)

■ Architecture Design Principle

■ **Shared Responsibility Model**
  - **`AWS Responsibility`**
    - Physical and Environmental Controls
      - AWS will take care of physical and environmental controls for you and **you inherit this right.**
    - Update the Firmware of the Network Infrastructure
    - Physical security of DynamoDB
    - Encryption of data at rest in DynamoDB
    - Confirming that the hardware is working in the data center
    - Patching of DynamoDB
    - Patching the operating system
    - Upgrade the firmware of the network infrastructure
    - Patching and fixing flaws within the infrastructure
    - Shutting down Lambda functions when they are no longer in use
    - Provision hosts
    - Secure the operating system
    - Maintain the security of the AWS Cloud
    - Implement physical and environmental controls
    - Operating system installations
    - Maintenance of physical and environmental controls
    - Applying updates to the Nitro Hypervisor
    - Maintain the physical security of edge locations
    - Patch the host operating system of an Amazon EC2 instance
    - Perform automated backups of Amazon RDS instances
  - **`Customer(Company) Responsibility`**
    - Access to DynamoDB tables
    - Classify company assets in the AWS Cloud
    - Manage connections to the database
    - Managing the code within the Lambda function
    - Manage database access permissions
    - Management of the guest operating systems
    - Perform client-side data encryption
    - Enabling client-side encryption for objects that are stored in Amazon S3
    - Application data security
    - Configure the AWS provided security group firewall
    - Configure firewalls and networks
    - Configure IAM credentials
    - Configuring IAM (Identity and Access Management) security policies to comply with the principle of least privilege
    - Patching their guest OS and applications
    - Patching the guest operating system on an Amazon EC2 instance is a customer responsibility
    - Updating and Patching Amazon WorkSpaces virtual Windows desktop
    - Updating the guest operating system on Amazon EC2 instances
    - Patch the guest operating system of Amazon EC2 instances
    - Creating versions of Lambda functions
    - Operating system installations
  - **`A shared responsibility between AWS and its customers`**
    - Patch Management
    - Configuration Management
    - Cloud Awareness and Training
    - Resource configuration management
    - Employee Awareness and Training

■ **Cloud Adoption Framework(CAF)**
  - Migration 조언
  - **Perspective**
    - Business Capabilities
      - **Business** : Strategic Partnership, Data Monetization
        - AWS availability and security provide the ability to improve service level agreements (SLAs) while reducing risk and unplanned downtime
        - Companies that migrate to the AWS Cloud reduce IT costs related to infrastructure, freeing budget for reinvestment in other areas
      - **`People`** : Cloud fluency, Culture Evolution
      - Governance : Benefits Management
        - A company has refined its workload to use specific AWS services to improve efficiency and reduce cost. This example show **Architecture optimization** for cost governance.
    - Techincal Capabilities
      - Platform : Platform Architecture, **Data Architecture**, Platform Engineering, Data Engineering, Provisioning & Orchestration, Modern App Development, CI/CD
      - Security : Security Governance, Security Assurance, Identity and Access Management, Threat Detection, Vulnerability Management, Data Protection, Application Security, **Incident Response, Infrastructure Protection**
      - Operations : Event management
        - Capabilities for configuration management and patch management
  - Cloud Transformation Journey
    - Envision phase
      - Focuses on demonstrating how cloud will help accelerate your business outcomes. It does so by identifying and prioritizing transformation opportunities across each of the four transformation domains in line with your strategic business objectives
    - Align phase
      - Identifying its capability gaps by using the AWS Cloud Adoption Framework (AWS CAF) perspectives. -> Align
    - Launch phase
      - focuses on delivering pilot initiatives in production and on demonstrating incremental business value
    - Scale phase
      - focuses on expanding production pilots and business value to desired scale and ensuring that the business benefits associated with your cloud investments are realized and sustained

■ AWS Global Infrastructure
  - Data Center
  - **`Availability Zone(AZ)`**
    - 논리적 개념
    - 1개 이상의 개별 데이터 센터(보통 3개 ~ 6개)
    - An environment that consists of one or more data centers
    - Made up of one or more discrete data centers that have redundant power, networking, and connectivity
  - **Region**
    - 물리적 위치
    - Compliance, Latency, AZ, Costs 고려
    - A component of the AWS Global Infrastructure
    - Region 종속 Service : EC2, Elastic Beanstalk, Lambda, Rekognition
    - Global Service : IAM, Route53, CloudFront, WAF
  - **`Global Edge Location`** : 서비스별 작업 수행하는 데이터 센터 **ex) CloudFront, Global Accelerator**

## Module 2 (Adding a Compute Layer)
■ **EC2**
  - AMI
  - EC2 Image Builder
    - Assists with the creation, testing, and management of custom Amazon EC2 images
  - User Data
  - Instance Metadata
  - Storing Data
    - **`EBS(Elastic Block Storage)`** : Root Volume
      - EC2 1대에 여러 EBS Attach 가능
      - Data Encryption 지원
      - Volume Type (SSD vs HDD)
      - EBS-Optimized
        - GP2(General Purpose SSD 2)
        - GP3(General Purpose SSD 3)
        - PIOPS(Provisioned IOPS SSD)
          - custom - SSD DISK SIZE / IO Speed
    - EFS / FSx
      - 네트워크 파일 시스템
      - A company wants a fully managed Windows file server for its Windows-based applications. -> FSx
      - ex) A company is hosting a web application on AWS using a single Amazon EC2 instance that stores user-uploaded documents in an Amazon EBS volume. For better scalability and availability, the company duplicated the architecture and created a second EC2 instance and EBS volume in another Availability Zone, placing both behind an Application Load Balancer. After completing this change, users reported that, each time they refreshed the website, they could see one subset of their documents or the other, but never all of the documents at the same time. What should a solutions architect propose to ensure users see all of their documents at once? -> **Copy the data from both EBS volumes to Amazon EFS. Modify the application to save new documents to Amazon EFS** <-> EBS doesn't support cross az only reside in one AZ.
        - 현재 아키텍처의 문제점은 각 EC2 인스턴스가 독립적인 EBS 볼륨을 가지고 있어 데이터 일관성이 보장되지 않는다는 점입니다. 이로 인해 사용자가 다른 인스턴스로 요청을 보내면 아직 동기화되지 않은 데이터만 볼 수 있게 됩니다.
        - The problem with the current architecture is that each EC2 instance has an independent EBS volume, which doesn't guarantee data consistency. This means that when a user sends a request to another instance, they only see data that hasn't yet been synchronized. <-> Copy the data so that all documents are contained on both EBS volumes solves data duplication, but it does not solve the real-time synchronization problem because each instance still manages data independently. Configure the Application Load Balancer to direct users to the server with the documents. bypasses the basic functionality of load balancing and forces direct connections to specific servers, reducing availability and scalability. Configure the Application Load Balancer to send requests to both servers, returning each document from the correct server helps distribute requests, but does not solve the problem of data inconsistency on each server.
      - ex) LA solutions architect is designing a high performance computing (HPC) workload on Amazon EC2. The EC2 instances need to communicate to each other frequently and require network performance with low latency and high throughput. Which EC2 configuration meets these requirements? -> **Launch the EC2 instances in a cluster placement group in one Availability Zone.**
      - ex) A company is creating a new application that will store a large amount of data. The data will be analyzed hourly and modified by several Amazon EC2 Linux instances that are deployed across multiple Availability Zones. The application team believes the amount of space needed will continue to grow for the next 6 months. Which set of actions should a solutions architect take to support these needs? -> **Store the data in an Amazon Elastic File System (Amazon EFS) file system. Mount the file system on the application instances.**
    - EC2 instance store
      - Is ephemeral and is deleted when an Amazon EC2 instance is stopped or terminated
      - ex) A solutions architect is deploying a distributed database on multiple Amazon EC2 instances. The database stores all data on multiple instances so it can withstand the loss of an instance. The database requires block storage with latency and throughput to support several million transactions per second per server. Which storage solution should the solutions architect use? -> **Amazon EC2 instance store**
  - Instance Size & Type
    - M5, C5, R5, T2 & T3, G3 & P3, I2, D2
  - Scale Up(수직 확장 : 인프라 추가 없이), Scale Out(수평 확장 : 인프라 추가)
  - **Pricing**
    - **`On-Demand Instance`** : 필요에 따라 인스턴스를 생성하고 제거
      - **(Cannot be interrupted(Spot Instances), No long-term commitment(Reserved Instances), or upfront payment(Dedicated Hosts))**
    - **`Reserved Instance(RI)`** : 결제 할인 옵션, 일정 사용량 악정 X, 예측 가능
    - **Spot Instance** : 공급/수요에 따라 조정, 짧은 시간, CPU 사용량에 따라 확장
    - **Dedicated Instance**
      - Dedicated Hosts : 코어당 소프트웨어 라이선스
    - **Savings Plan** : 1년 또는 3년 기간 동안 일정한 컴퓨팅 사용량 약정
    - **Pay-as-you-go** : 종량제 가격 측정, 인프라 선행 투자 X
  - Tag
  - Placement Group
    - Cluster
    - Spread
    - Partition
  - Limit increase
    - Default limit of 20 instances per region(Case open can be increase)
  - Connect
    - EC2 Instance Connect
    - Systems Manager Session Manager
  <br/>
  - **The advantages of using Amazon EC2 instances** to host applications in the AWS Cloud instead of on premises
    - EC2 integrates with Amazon VPC, AWS CloudTrail, and AWS Identity and Access Management (IAM)
    - EC2 has a flexible, pay-as-you-go pricing model

■ **LightSail**
  - **가상 프라이빗 서버(Virtual Private Server)**

■ **`Elastic BeanStalk`**
  - **간단하게 서버 배포**
  - 프로비저닝 필요 없이 빠르게 애플리케이션을 AWS에 업로드
  - 용량 조정(프로비저닝), 로드 밸런싱, Auto Scaling, 애플리케이션 상태 모니터링

■ **Lambda** : **Serverless** (코드가 서버에서 실행되지만, 프로비저닝하거나 관리 필요 없음)
  - 비용 청구 기준 : 실행시간, 요청 수
  - 회사의 책임 : 코드 내부의 보안, 코드 작성 및 업데이트
  - **The responsibility of a company that is using AWS Lambda**
    - Security inside of code
    - Writing and updating of code
  - ex) A solutions architect is designing a solution to access a catalog of images and provide users with the ability to submit requests to customize images. Image customization parameters will be in any request sent to an AWS API Gateway API. The customized image will be generated on demand, and users will receive a link they can click to view or download their customized image. The solution must be highly available for viewing and customizing images. What is the MOST cost-effective solution to meet these requirements? -> **Use AWS Lambda to manipulate the original image to the requested customizations. Store the original and manipulated images in Amazon S3. Configure an Amazon CloudFront distribution with the S3 bucket as the origin.**

■ **`Fargate`** : **Container(ECS, EKS)**를 위한 Serverless 컴퓨팅 엔진
  - A **serverless** compute engine for containers

■ Batch

## Module 3 (The Simiplest Architecture)
■ **`S3`**
  - **Severless**
  - Data Encryption 제공
  - Provides highly durable object storage
  - Used to host static websites
  <br/>
  - Bucket, Object
  - Tag
    - ACL
    - Bucket Policy
    - CORS(Cross-Origin Resource Sharing)
  - **`Versioning`**
  - Multipart Upload
  - **Transfer Acceleration**
    - ex) A company collects data for temperature, humidity, and atmospheric pressure in cities across multiple continents. The average volume of data that the company collects from each site daily is 500 GB. Each site has a high-speed Internet connection. The company wants to aggregate the data from all these global sites as quickly as possible in a single Amazon S3 bucket. The solution must minimize operational complexity. -> **Turn on S3 Transfer Acceleration on the destination S3 bucket. Use multipart uploads to directly upload site data to the destination S3 bucket.**
      - S3 Transfer Acceleration은 Edge Location을 통해 데이터 전송을 가속화하여 전 세계 어디에서나 데이터를 빠르게 전송할 수 있습니다. 멀티파트 업로드를 사용하면 대용량 파일을 효과적으로 전송할 수 있습니다. 이러한 방법은 운영 복잡성을 최소화하면서도 데이터를 빠르게 집계할 수 있습니다.
        - 2번: S3 교차 리전 복제는 추가 비용이 발생하며, 데이터를 복제한 후 원본을 제거해야 하므로 운영 복잡성이 증가할 수 있습니다.
        - 3번: AWS Snowball Edge Storage Optimized 디바이스를 사용하면 초기 비용이 발생하며, 데이터 전송이 완료될 때까지 시간이 걸릴 수 있습니다.
        - 4번: Amazon EC2 인스턴스와 Amazon EBS 볼륨을 사용하면 추가 비용이 발생하며, 운영 복잡성이 증가할 수 있습니다.
    - ex) A company wants to host a scalable web application on AWS. The application will be accessed by users from different geographic regions of the world. Application users will be able to download and upload unique data up to gigabytes in size. The development team wants a cost-effective solution to minimize upload and download latency and maximize performance. What should a solutions architect do to accomplish this? -> **Use Amazon S3 with Transfer Acceleration to host the application.**
  - **Athena** : Serverless 대화형 쿼리 서비스
    - ex) A company needs the ability to analyze the log files of its proprietary application. The logs are stored in JSON format in an Amazon S3 bucket. Queries will be simple and will run on-demand. A solutions architect needs to perform the analysis with minimal changes to the existing architecture. What should the solutions architect do to meet these requirements with the LEAST amount of operational overhead? -> **Use Amazon Athena directly with Amazon S3 to run the queries as needed**
    - Amazon Athena is an interactive query service that makes it easy to analyze data directly in Amazon Simple Storage Service (Amazon S3) using standard SQL. With a few actions in the AWS Management Console, you can point Athena at your data stored in Amazon S3 and begin using standard SQL to run ad-hoc queries and get results in seconds. : 최소한의 변경, 낮은 오버헤드, 주문형 쿼리
    - (Redshift): 대용량 데이터 분석에 적합하지만, 모든 로그를 로드하고 관리해야 하므로 기존 아키텍처 변경과 설정 오버헤드가 높습니다. (CloudWatch Logs): 로그 수집 및 모니터링에 특화되어 있으며, 복잡한 분석 쿼리에는 제한적입니다. (AWS Glue & EMR): 데이터 전처리 및 복잡한 분석에 유용하지만, 설정 및 관리 복잡성이 높아 운영 오버헤드가 증가합니다.
  - Bucket Policy
    - ex) A company uses AWS Organizations to manage multiple AWS accounts for different departments. The management account has an Amazon S3 bucket that contains project reports. The company wants to limit access to this S3 bucket to only users of accounts within the organization in AWS Organizations. Which solution meets these requirements with the LEAST amount of operational overhead? -> **Add the aws PrincipalOrgID global condition key with a reference to the organization ID to the S3 bucket policy.**
      - PrincipalOrgID Validates if the principal accessing the resource belongs to an account in your organization.
      - aws:PrincipalOrgID is a global condition key that verifies the organization ID to which a user or role belongs. Setting this condition key in the master account's S3 bucket policy ensures that access to the bucket is granted only if the entity attempting to access it belongs to the specified organization ID. 사용자 또는 역할의 속한 조직 ID를 확인할 수 있는 전역 조건 키입니다. 마스터 계정의 S3 버킷 정책에 이 조건 키를 설정하면, 해당 버킷에 접근하려는 주체가 정확히 지정된 조직 ID에 속해야만 접근이 허용됩니다.
      - 2번 (OU 활용): 조직 단위(OU) 기반 접근 제어는 세분화된 관리에는 유용하지만, 조직 전체 범위에서의 간단하고 효율적인 접근 제한에는 오버헤드가 발생합니다. OU 설정 및 정책 관리가 추가적인 작업을 필요로 합니다. 3번 (CloudTrail 모니터링): CloudTrail은 이벤트 로깅 및 감사에 탁월하지만, 실시간 접근 제어 변경을 위한 자동화된 솔루션으로는 부적합합니다. 수동 개입이 필요하며, 운영 오버헤드가 증가합니다. 4번 (태그 기반 접근): 태그 기반 접근 제어는 유연하지만, 모든 사용자에게 태그를 일관되게 관리해야 하는 부담이 있습니다. 조직 전체 계정 사용자에 대한 자동화된 접근 제어에는 적합하지 않습니다.

■ **Glacier**
  - Vault, Archive
  - Retrieval options
    - Expedited
    - Standard
    - Bulk

■ **Storage Class** + S3 Analytics + S3 Lifecycle  
  - **S3 Standard** : **저장 공간 무제한**, 최대 파일 크기 <= 5TB
    - The total amount of storage offered by Amazon S3 is Unlimited.
  - **`S3 Standrad Infrequent Access(Standard-IA)`** : AZ 저장 개수 3개 이상, 자주 접근되지는 않으나 접근시 빠른 접근
  - **S3 One Zone IA(One Zone-IA)** : AZ 저장 개수 1개, 더 저렴, 덜 중요하고 자주 사용되지 않는 데이터
  - **S3 Intelligent Tiering**
  - Glacier / Deep Archive : Archiving, 장기백업, 저렴
  - Glacier / Flexible Retrieval : Utility function is vague and there is no need for flexible storage

■ IOPS : 초당 입출력 작업 수

## Module 4 (Adding a Database Layer)
■ **RDS(Relational Database Service)**
  - 관리형 데이터베이스 서비스
  - DB 이중화 : Multi AZ 사용
  - **OLTP(Online Transactional Processing) 용도**
  - 종류
    - **Amazon Aurora** : MySQL, PostgreSQL과 호환되는 관계형 DB 엔진
      - fully managed MySQL-compatible database
      - 가용성, 내구성, 성능 개선
      - ex) A company runs an ecommerce application on Amazon EC2 instances behind an Application Load Balancer. The instances run in an Amazon EC2 Auto Scaling group across multiple Availability Zones. The Auto Scaling group scales based on CPU utilization metrics. The ecommerce application stores the transaction data in a MySQL 8.0 database that is hosted on a large EC2 instance. The database's performance degrades quickly as application load increases. The application handles more read requests than write transactions. The company wants a solution that will automatically scale the database to meet the demand of unpredictable read workloads while maintaining high availability. Which solution will meet these requirements? -> **Use Amazon Aurora with a Multi-AZ deployment. Configure Aurora Auto Scaling with Aurora Replicas.**
        - 1번 (Amazon Redshift): Redshift는 주로 대규모 데이터웨어하우스와 분석 작업에 최적화되어 있어, 실시간 트랜잭션 처리에 적합하지 않으며 읽기 부하의 동적 확장에 특화되어 있지 않습니다.
        - 2번 (Amazon RDS 단일/다중 AZ): RDS의 다중 AZ 배포는 가용성을 향상시키지만, 주어진 상황에서 읽기 작업 부하의 자동 확장 기능이 제한적입니다. 특히 MySQL 8.0의 기본 설정으로는 읽기 복제본을 통한 부하 분산이 수동 설정에 의존하는 경향이 있습니다.
        - 3번 (Amazon Aurora with Aurora Replica Auto Scaling): Aurora는 MySQL과 호환되며, 고성능과 고가용성을 제공합니다. 특히 Aurora Replica를 활용하면 읽기 작업 부하를 효과적으로 분산시킬 수 있습니다. Aurora Auto Scaling 기능은 복제본 수를 자동으로 조정하여 예측 불가능한 읽기 부하에 대응할 수 있습니다. 이는 회사가 원하는 자동 확장과 고가용성을 동시에 만족시킵니다.
        - 4번 (ElastiCache with Memcached): ElastiCache는 캐싱 솔루션으로, 중간 계층에서 읽기 부하를 줄일 수 있지만, 데이터베이스 자체의 확장성과 데이터 일관성 유지 측면에서는 제한적입니다. 데이터베이스의 핵심 트랜잭션 처리와 직접적인 확장성 해결에는 적합하지 않습니다.
      - ex) A company is migrating a three-tier application to AWS. The application requires a MySQL database. In the past, the application users reported poor application performance when creating new entries. These performance issues were caused by users generating different real-time reports from the application during working hours. Which solution will improve the performance of the application when it is moved to AWS? -> **Create an Amazon Aurora MySQL Multi-AZ DB cluster with multiple read replicas. Configure the application to use the reader endpoint for reports.**
      - ex) A company runs a multi-tier web application that hosts news content. The application runs on Amazon EC2 instances behind an Application Load Balancer. The instances run in an EC2 Auto Scaling group across multiple Availability Zones and use an Amazon Aurora database. A solutions architect needs to make the application more resilient to periodic increases in request rates. -> **Add Aurora Replica. / Add an Amazon CloudFront distribution in front of the Application Load Balancer.**
      - ex) An application running on AWS uses an Amazon Aurora Multi-AZ deployment for its database. When evaluating performance metrics, a solutions architect discovered that the database reads are causing high I/O and adding latency to the write requests against the database. What should the solutions architect do to separate the read requests from the write requests? -> **Create a read replica and modify the application to use the appropriate endpoint** <-> There is no standby instance in Aurora.
    - MSSQL
    - Maria DB
    - Oracle
  - Scaling DB
    - Read Replica
    - Sharding
    - DB 트래픽 급증 처리

■ **`DynamoDB`**
  - **NoSQL DB**
  - **A key-value database** that provides sub-milliseconed latency on a large scale
  - 관리형 데이터베이스 서비스
  - 초저지연시간
  - Row 형식
  - Global Table
  - Consistency Option
    - Eventually
    - Strongly
  - Scaling DB
      - Auto Scaling/On Demand
      - Adaptive Capacity
  - ex) A solutions architect is designing a new service behind Amazon API Gateway. The request patterns for the service will be unpredictable and can change suddenly from 0 requests to over 500 per second. The total size of the data that needs to be persisted in a backend database is currently less than 1 GB with unpredictable future growth. Data can be queried using simple key-value requests. Which combination of AWS services would meet these requirements? (Choose two.) -> **AWS Lambda / Amazon DynamoDB**

■ DAX(DynamoDB Accelerator) : DynamoDB 전용 Cache

■ DocumentDB
  - NoSQL
  - MongoDB 호환 DB
  - Use cloud-native storage that provides replication across multiple Availability Zones by default

■ ElastiCache
  - In-Memory Cache
  - 성능 향상

■ Neptune
  - Graph DB
  - Use cloud-native storage that provides replication across multiple Availability Zones by default

■ QLDB(Quantum Ledger Database) : Serverless 완전 관리형 DB

■ Best Practice
  - **Managed database services : RDS, DynamoDB**

## Module 5 (Networking - Part I)
■ **`VPC`**
  - Resources(구성요소) : Subnet, IGW, NAT, Route Table, DNS Host Name, CIDR
  - Multi-VPC
  - Multi-Account
  - The scope of a VPC within the AWS network is A VPC can span all Availability Zones within an AWS Region.

■ VPC Limits
  - dfault vpc 5 per Region(Case open can be increased)

■ CIDR

■ **Subnet**
  - Public
    - Internet Gateway : VPC -> S3, DynamoDB(Direct 접근 불가능)
  - Private
    - NAT Gateway(vs. NAT Instance) : Direct 접근 가능

■ **IGW(Internet Gateway)**

■ **`Routing Table`**
  - 트래픽이 어디로 전달되어야 할지 알려주는 테이블
  - <-> Secutiry Group(SG), Network ACL(NACL)

■ ENI(Elastic Network Interface)

■ EIP(Elastic IP)

■ Security
  - Can be used to block network traffic to an instance
    - **`Security Group(SG)`**
      - 가상 **Stateful** 방화벽, 허용 규칙만 포함, **인스턴스(EC2, RDS) 수준**
      - Acts as a firewall for Amazon EC2 instances
      - Acts as an instance-level firewall to control inbound and outbound access
      - Gives a company the ability to control incoming traffic and outgoing traffic for Amazon EC2 instances
      - Security Group Chaining
    - **`Network ACL(NACL)`**
      - **Stateless**, IP주소만 포함, Subnet 수준
      - 트래픽 허용 여부를 결정할 때 가장 낮은 번호의 규칙부터 순서대로 규칙을 처리
      - Can be used to set up a firewall to control traffic going into and coming out of an Amazon VPC subnet
      - They process rules in order, starting with the lowest numbered rule, when deciding whether to allow traffic.
      - Acts as a VPC firewall at the subnet level
      - ex) S3 객체 액세스 제한
    - **`Network Firewall`**
      - ex) A company recently migrated to AWS and wants to implement a solution to protect the traffic that flows in and out of the production VPC. The company had an inspection server in its on-premises data center. The inspection server performed specific operations such as traffic flow inspection and traffic filtering. The company wants to have the same functionalities in the AWS Cloud. Which solution will meet these requirements? -> **Use AWS Network Firewall to create the required rules for traffic inspection and traffic filtering for the production VPC**
        - AWS 네트워크 방화벽 (Network Firewall)이 이 요구 사항에 가장 적합한 솔루션입니다.
          - 온프레미스 기능 복제: 온프레미스의 검사 서버 역할을 AWS 클라우드 내에서 동일하게 수행할 수 있습니다. 네트워크 방화벽은 VPC 수준에서 트래픽을 실시간으로 검사하고 필터링하여 악성 트래픽 차단, 허용 규칙 적용 등을 가능하게 합니다.
        - 1번 (Amazon GuardDuty): 침입 탐지 서비스로, 의심스러운 활동을 감지하는 데 중점을 둡니다. 트래픽 검사 및 필터링 기능은 제공하지 않습니다.
        - 2번 (트래픽 미러링): 트래픽을 미러링하여 분석할 수는 있지만, 실시간 필터링이나 차단 기능이 없습니다. 별도의 분석 도구와 함께 사용해야 합니다.
        - 4번 (AWS Firewall Manager): 여러 계정 및 VPC에 대한 보안 정책 관리를 간소화하지만, 기본적인 트래픽 검사 및 필터링 기능은 네트워크 방화벽에 의존합니다. 즉, 직접적인 트래픽 처리 기능은 제공하지 않습니다.

## Module 6 (Networking - Part II)
■ CGW(Customer Gateway)
  - On-Premise, 외부 네트워크

■ VGW(Virtual Private Gateway)

■ **VPN**(Virtual Private Network)
  - **`Site`-to-Site VPN`** : IPSec 통신, AWS VPC(**Virtual Private Gateway**) -> On-Premise(**Customer Gateway**) 네트워크 연결

■ Managed VPN

■ **`Direct Connect`**(DX) : **인터넷 없이 On-Premise <-> AWS Cloud** 네트워크 연결

■ **VPC Peering(VPC <-> VPC)** : **VPC 간 인터넷 없이 Private IP로 연결**

■ **VPC Endpoints**
  - 인터넷 없이 EC2 인스턴스 -> S3 버킷, DynamoDB용 VPC 엔드포인트 액세스
    - Offer gateway VPC endpoints that can be used to avoid sending traffic over the internet
  - Interface
  - VPC Enpoint
    - 트래픽이 인터넷을 통과하지 못함
  - Gateway Endpoint
    - 인터넷 연결 없이 VPC와 S3 간의 프라이빗 연결
    - 인터넷 게이트웨이나 NAT 디바이스 없이
    - ex) A company has application running on Amazon EC2 instances in a VPC. One of the applications needs to call an Amazon S3 API to store and read objects. The company's security policies restrict any internet-bound traffic from the applications. Which action will fulfill these requirements and maintain security? -> **Configure an S3 gateway endpoint.** <-> Can't be interface endpoint as there is no interface endpoint for S3 or DynamoDB. Gateway endpoint is only applicable for S3 and Dynamo DB.
  - Gateway Load Banlancer
  - ex) An application runs on an Amazon EC2 instance in a VPC. The application processes logs that are stored in an Amazon S3 bucket. The EC2 instance needs to access the S3 bucket without connectivity to the internet. Which solution will provide private network connectivity to Amazon S3? -> **Create a gateway VPC endpoint to the S3 bucket** This is the most efficient way to connect directly to S3 from within your VPC. A gateway endpoint provides a private network path between your VPC and S3, allowing you to securely exchange data without internet traffic.
    - You can set up interface VPC endpoint for CloudWatch Logs for private network from EC2 to CloudWatch. But from CloudWatch to S3 bucket: Log data can take up to 12 hours to become available for export and the requirement only need EC2 to S3. / Create an instance profile just grant access but not help EC2 connect to S3 privately. / API Gateway like the proxy which receive network from out site and it forward request to AWS Lambda, Amazon EC2, Elastic Load Balancing products such as Application Load Balancers or Classic Load Balancers, Amazon DynamoDB, Amazon Kinesis, or any publicly available HTTPS-based endpoint. But not S3.
    - (Using CloudWatch Logs): This is useful for centrally managing logs, but it doesn't provide a way for EC2 instances to directly access S3 without the internet. (Instance Profile): Granting S3 access to the EC2 instance via an IAM role does not meet the problem criteria as the access is via an internet connection. (API Gateway): API Gateway is an API management tool for inter-application communication and does not directly provide private network connectivity.
    - (S3 버킷에 대한 게이트웨이 VPC 엔드포인트 생성): VPC 내부에서 S3에 직접 연결하는 가장 효과적인 방법입니다. 게이트웨이 엔드포인트는 VPC와 S3 사이에 사설 네트워크 경로를 제공하여 인터넷 트래픽 없이 안전하게 데이터를 주고받을 수 있도록 합니다.
    - 2번 (CloudWatch Logs 이용): 로그를 중앙 집중적으로 관리하는 데 유용하지만, EC2 인스턴스가 인터넷 없이 S3에 직접 접근하는 방법을 제공하지 않습니다. 3번 (인스턴스 프로필): IAM 역할을 통해 EC2 인스턴스에 S3 액세스 권한을 부여하지만, 인터넷 연결을 통한 접근 방식이므로 문제 조건에 맞지 않습니다. 4번 (API Gateway): API Gateway는 애플리케이션 간 통신을 위한 API 관리 도구이며, 프라이빗 네트워크 연결을 직접 제공하지 않습니다.
  - ex) A company has a three-tier web application that is deployed on AWS. The web servers are deployed in a public subnet in a VPC. The application servers and database servers are deployed in private subnets in the same VPC. The company has deployed a third-party virtual firewall appliance from AWS Marketplace in an inspection VPC. The appliance is configured with an IP interface that can accept IP packets. A solutions architect needs to integrate the web application with the appliance to inspect all traffic to the application before the traffic reaches the web server. Which solution will meet these requirements with the LEAST operational overhead? -> **Deploy a Gateway Load Balancer in the inspection VPC. Create a Gateway Load Balancer endpoint to receive the incoming packets and forward the packets to the appliance.** (Gateway Load Balancer is a new type of load balancer that operates at layer 3 of the OSI model and is built on Hyperplane, which is capable of handling several thousands of connections per second. Gateway Load Balancer endpoints are configured in spoke VPCs originating or receiving traffic from the Internet. This architecture allows you to perform inline inspection of traffic from multiple spoke VPCs in a simplified and scalable fashion while still centralizing your virtual appliances.)
    - 게이트웨이 로드 밸런서(Gateway Load Balancer)는 VPC 경계에서 트래픽을 처리하는 데 최적화되어 있습니다. 이는 퍼블릭 인터넷에서 오는 모든 트래픽을 효율적으로 분산시키고 관리할 수 있는 능력을 제공합니다. 서버에 도달하기 전에 모든 트래픽을 검사해야 하는 상황에서, 게이트웨이 로드 밸런서는 VPC의 경계에서 트래픽을 수신하고 이를 타사 가상 방화벽 어플라이언스로 라우팅하는 데 이상적입니다. 최소한의 운영 오버헤드를 유지하려면, 게이트웨이 로드 밸런서는 설정과 관리가 비교적 간단하며, 자동 확장 및 고가용성을 지원합니다. 이는 직접적인 라우팅 테이블 조정이나 복잡한 로드 밸런서 구성보다 관리 부담이 적습니다. 옵션 1과 2의 Network Load Balancer와 Application Load Balancer는 주로 내부 트래픽이나 특정 애플리케이션 레이어의 트래픽을 처리하는 데 더 적합하며, 퍼블릭 인터넷 트래픽의 경계 처리에는 제약이 있을 수 있습니다. 옵션 3의 전송 게이트웨이는 주로 다른 VPC나 온프레미스 환경 간의 통신을 위한 것이므로, 이 문제의 요구 사항에 정확히 맞지 않습니다.

■ **Transit Gateway** : **여러 개 VPC 간 연결 <-> On-Premise 네트워크 연결**, Hub and Spoke 형태, 중앙 관리

■ **PrivateLink(Endpoint Service)** : **VPC와 AWS 서비스 간**에 프라이빗 연결

■ **ELB**
  - High Availability(고가용성) : 리소스에 장애가 발생하더라도 애플리케이션에 계속 액세스 보장
  - Options
    - Application(ALB) : L7
    - **`Network(NLB)`** : L4, TCP, UDP
    - Classic(CLB) : L4, L7

■ **Route53**
  - 라우팅 정책
    - 단순 라우팅 정책
    - 장애 조치 라우팅 정책
    - 지리 위치 라우팅 정책
    - 지리 근접 라우팅 정책
    - 지연 시간 라우팅 정책
    - 다중 응답 라우팅 정책
    - 가중치 기반 라우팅 정책
  - Traffic Flow
  - Latency Based Routing
  - Geo DNS(Geographic Location)
  - Private DNS
  - Health Checks and Monitoring
  - DNS Failover
  - Weighted Round Robin(WRR)
  - A highly available and scalable DNS web service
  - Provides domain registration, DNS routing, and service health checks
  
## Module 7 (IAM)
■ **Account Root User**
  - Root User Privileges
  - Best Practice
    - Enable multi-factor authentication (MFA) on the root user
    - Create an IAM user with administrator privileges for daily administrative tasks, instead of using the root user
    - The first sign-in identity that is available when an AWS account is created
    - Enable multi-factor authentication (MFA) for the root user
    - Control access to AWS service APIs and to other specific resource
    - Tasks that require root user
      - Changing the AWS Support plan
      - Change your account settings
      - Closing an AWS Account
    - Tasks that dose not require root user
      - Viewing billing information X
      - Starting and stopping Amazon EC2 instances X
      - Opening an AWS Support case  X

■ **IAM**
- **Always provied at no charge**
  - User / Federated User(AD, LDAP)
    - 회사가 개인의 AWS 접근 자격 증명 생성하는 경우
    - 회사에서 IAM 그룹에 사용자를 추가해야 하는 경우
    - A company should create an IAM user instead of an IAM role
      - When the company creates AWS access credentials for individuals
      - When the company needs to add users to IAM groups
  - Group
  - Policy
    - Resource-Based / Identity-Based
    - Manage / Inline
  - **`Role`**
    - **STS(Security Token Service)** : 제한된 권한의 임시 자격 증명 생성
    - Identity Broker
    - SAML
    - Amazon EC2 인스턴스에서 실행되는 애플리케이션이 다른 AWS 서비스에 액세스해야 하는 경우
    - 회사가 AWS에 요청하는 휴대폰에서 실행되는 애플리케이션을 생성하는 경우
    - ex) A company is implementing a new business application. The application runs on two Amazon EC2 instances and uses an Amazon S3 bucket for document storage. A solutions architect needs to ensure that the EC2 instances can access the S3 bucket. -> **Create an IAM role that grants access to the S3 bucket. Attach the role to the EC2 instances.**
      - EC2 인스턴스가 안전하고 효과적으로 S3 버킷에 액세스하려면, 인스턴스 자체에 직접 사용자 권한을 부여하는 대신 IAM 역할을 활용하는 것이 가장 적합합니다. IAM 역할은 인스턴스에 동적으로 첨부되어 해당 인스턴스에서 실행되는 애플리케이션이나 프로세스가 필요한 권한을 얻을 수 있게 합니다. 이 방식은 인스턴스의 신원을 기반으로 권한을 부여하며, 명시적으로 사용자 계정이나 그룹을 인스턴스에 연결하는 것보다 관리가 용이하고 보안상 이점이 큽니다.
  - Access
    - Least privilege access : Using IAM to grant access only to the resources needed to perform a task
    - Restricted access
    - As-needed access
    - Token access

■ **IAM Access Analyzer**
  - IAM 역할 or S3 버킷이 외부 엔티티와 공유되었는지 식별
  - Identifies whether an Amazon S3 bucket or an IAM role has been shared with an external entity.
  - Analyzes resource policies in your AWS environment to help you identify and address unintended access.
  - Analyzes IAM policies to identify potential issues and excessive permissions.
  - Grant permissions to applications that run on Amazon EC2 instances.
  - Checks access policies and offers actionable recommendations to help users set secure and functional policies.

■ **`IAM Credential Report`**
  - Provides detailed information about the rotation history of user passwords and access keys within the account. It shows dates of last password and access key rotation, along with usernames and key IDs. This aligns perfectly with the requirement of auditing password and access key rotation details for compliance purposes.
  - The date and time when an IAM user's password was last used to sign in to the AWS Management Console
  - Whether multi-factor authentication (MFA) has been enabled for an IAM user

■ **Cognito**(mobile/app User Pool Federated Identities)
  - 사용자 인증, 직접 로그인, 임시 권한
  - 웹 및 모바일 어플리케이션 사용자에 대한 자격 증명 제공
  - User Pool

■ Landing Zone

■ Multi Accounts
  - Cross-account access
  - **Organizations**
    - Multiple AWS accounts
    - Root, OU(Organizational Units), Policy
    - Can be used at no additional cost
    - Share pre-purchased Amazon EC2 resources across accounts
    - Service control policies (SCPs) manage permissions
    - Helps to centrally manage billing and allow controlled access to resources across AWS accounts
    - A user is able to set up a master payer account to view consolidated billing reports through **Organizations**
    - **Consolidated Billing** : 통합 결제 추구
      - Benefits : Volume discounts, One bill for multiple accounts
      - Can be used to track charges across multiple accounts and report the combined cost

■ **Systems Manager**
  - 애플리케이션 및 리소스를 위한 운영 허브
  - 안전하고 보안이 보장된 대규모 운영을 지원하는 하이브리드 클라우드 환경을 위한 안전한 End-to-End 관리 솔루션
  - Run Command
  - Maintenance Windows
  - Patch Management
  - State Manager
  - **Session Manager**
    - Systems Manager Session Manager : A company wants to improve its security and audit posture by limiting Amazon EC2 inbound access.
  - Inventory

■ Systems Manager Parameter Store
  - A company wants to design a centralized storage system to manage the configuration data and passwords for its critical business applications.

■ **Service Catalog**
  - 액세스 제한

■ **`Access keys`**

■ **Management Console**

■ **CLI**

■ Cloud Development Kit(CDK = Cloud SDK) : 프로그래밍 방식
  - Should a developer use to integrate AWS service features directly into an application
  - Allows users to connect with and deploy AWS services programmatically
  - Use to define cloud resources as code and provision the resources through AWS CloudFormation

■ CloudShell : Provides command line access to AWS tools and resources directly from a web browser

■ **`Directory Service`** : AD(Active Directory)를 위한 서비스

■ Managed Microsoft AD : MFA

■ AD Connector

■ Single Sign On(SSO) : 사용자가 회사 네트워크에서 인증을 받고 두 번째 로그인 없이 AWS를 사용할 수 있기를 원하는 경우

## Module 8(Costs and Support Plan)
■ Costs
  - **`Cost Explorer`** : 과거 데이터로 비용 탐색기, 그래프로 시각화
    - **`Cost Allocation Tags`** : 부서별 리소스 사용 비용
    - Consolidated Billing
    - Helps users visualize, understand, and manage spending and usage over time
  - **`Budgets`** : 예산 설정
  - Detailed Billing Report : 상세 비용 보고서
  - Simple Monthly Caculator : AWS 계정 없어도 예상 비용 확인
  - **Pricing Caculator** : On-Premise를 AWS 실행할 때 예상 비용 확인
  - **TCO Caculator** : On-Premise -> AWS Cloud 이전 비용 확인
    - AWS replaces upfront capital expenditures with pay-as-you-go costs
    - AWS uses economies of scale to continually reduce prices
  - **Cost Anomaly Detection** : 기계 학습으로 비용 및 사용량 모니터링
    - Uses machine learning to continuously monitor cost and usage for unusual cloud spending
  - **`Cost and Usage Reports`**
    - <-> Cost Explorer
  - **Compute Optimizer** : 리소스의 구성 및 사용률 지표를 AWS 분석하여 크기 조정 권장 사항을 제공하는 서비스
  - **Personal Health Dashboard**

■ **Support Plan**
  - **Basic Support**
    - Personal Health Dashboard 제공
    - The free plan that provides access to documentation, forums, and basic support features.
  - **Developer Support**
    - Designed for developers running non-production workloads. It includes business hours access to Cloud Support Engineers and is suitable for development and testing environments
  - **Business Support**
    - Trusted Advisor 제공
    - Includes 24/7 access to Cloud Support Engineers. It is suitable for businesses running production workloads
    - The least expensive AWS Support plan that contains a full set of AWS Trusted Advisor best practice checks
  - **`Enterprise Support`**
    - Trusted Advisor 제공
    - Technical Account Manager(TAM), SME 지원
    - Designated support from an AWS technical account manager (TAM)
    - Support of third-party software integration to AWS
    - IEM(Infrastructure Event Management) : 인프라 이벤트 관리
    - AWS에 대한 타사 소프트웨어 통합 지원
    - 연중 무휴 응답
    - 컨설팅 아키텍처 및 운영 지침을 제공
    - Concierge Support Team 지원
      - AWS Billing 및 AWS Support 연락 창구
      - A primary point of **contact** for AWS Billing and AWS Support
    - Business Critical System Stop < 15minutes
    - The premium Support plan providing a wide range of benefits, including 24/7 access to Cloud Support Engineers, a Technical Account Manager(TAM), and more. It is suitable for enterprises running business-critical workloads
    - A designated technical account manager (TAM) to assist in monitoring and optimization

■ Free Tier
  - A company is using the AWS Free Tier for several AWS services for an application. If the Free Tier usage period expires or if the application use exceeds the Free Tier usage limits, The company will be charged the standard pay-as-you-go service rates for the usage that exceeds the Free Tier usage.

## Module 9 (Service, Security and Credentials)
■ **`Global Accelerator`** : 즉시 대응 서비스, 애플리케이션의 전반적인 가용성 및 성능 개선
  - Helps deliver highly available applications with fast failover for multi-Region and Multi-AZ architectures
  - Improves network performance by sending traffic through the AWS worldwide network infrastructure
  - ex) A start-up company has a web application based in the us-east-1 Region with multiple Amazon EC2 instances running behind an Application Load Balancer across multiple Availability Zones. As the company's user base grows in the us-west-1 Region, it needs a solution with low latency and high availability. -> **Provision EC2 instances and configure an Application Load Balancer in us-west-1. Create an accelerator in AWS Global Accelerator that uses an endpoint group that includes the load balancer endpoints in both Regions.**

■ **`Config`**
  - 리소스 변경 사항 기록
  - 리소스 규정 준수 감사

■ **`Control Tower`** : 다중 계정 AWS 환경 자동화 및 관리

■ **Service Quotas** : 회사에서 서비스 한도 증가를 중앙에서 요청하고 추적하기 위해 사용해야 하는 서비스

■ **X-Ray** : Serverless, MSA 애플리케이션 분석 및 디버깅, 사용자 요청 추적
  - Provides the capability to view end-to-end performance metrics and troubleshoot distributed applications

■ **`Trusted Advisor`**
  - **AWS 모범 사례**에 따라 정밀조사, 권장사항, 오픈 액세스 권한 확인, **보안 검사** 제공하는 포털
  - AWS 리소스의 서비스 할당량을 사전에 모니터링하고 계획하는 데 사용할 수 있는 기능을 제공
  - Best Practice : 비용 최적화, 성능, 보안, 내결함성, 서비스 제한
  - Detecting underutilized resources to save costs
  - Improving security by proactively monitoring the AWS environment
  - Provides recommendations for optimizing AWS resources for cost savings, performance, security, and fault tolerance
  - Can be used to proactively monitor and plan for the service quotas of AWS resources

■ **Inspector**
  - **인프라**의 자동 보안 평가 서비스
  - 보안 및 규정 준수 개선
  - Assess application vulnerabilities and must identify infrastructure deployments that do not meet best practices
  - A company wants an automated process to continuously scan its Amazon EC2 instances for software vulnerabilities

■ **`Artifact`**
  - 보안 및 규정 준수 보고서 제공 서비스
  - ISO 인증 문서 제공
    - Artifact should provide AWS ISO certifications
  - Allows users to download security and compliance reports about the AWS infrastructure on demand.
  - Provides on-demand access to AWS compliance reports and documents.
  - Primarily used for managing and tracking infrastructure as code(IaC) configurations, not directly related to credential auditing.
  - The best resource for a user to find compliance-related information and reports about AWS

■ **Audit Manager**
  - Helps you continuously audit your AWS usage to simplify how you assess risk and compliance with regulations and incustry standards.
  - Offers a comprehensive platform for managing and automating audits across various AWS services, but it requires additional setup and configuration compared to the readily available IAM credential report.

■ **Compliance Program**

■ **Knowledge Center**
  - Provides answers to the most frequently asked security-related questions that AWS receives from its users.

■ **`GuardDuty`**
  - 지능형 위협 탐지 서비스
  - 머신러닝 알고리즘을 통해 이상 탐지 수행
  - A threat detection service that continuously monitors for malicious activity and unauthorized behavior in AWS accounts
  - Monitors AWS accounts for security threats
  - Provides threat detection by monitoring for malicious activities and unauthorized actions to protect AWS accounts, workloads, and data that is stored in Amazon S3
  - Automatic monitoring for threats to AWS workloads

■ Penetration Testing
  - 자체 인프라 공격해 보안 테스트

■ **Macie**
  - 완전 관리형 데이터 개인 정보 보호 서비스
  - S3 버킷 내 개인 식별 정보 찾음 및 경고
  - Gives users the ability to discover and protect **sensitive data** that is stored in Amazon S3 buckets
  - Automatically recognizes and classifies **sensitive data** or intellectual property on AWS
  - Uses machine learning to help discover, monitor, and protect **sensitive data** that is stored in Amazon S3 buckets

■ Certificate Manager(ACM)
  - SSL/TLS 인증서 쉽게 생성 및 관리 지원 서비스

■ **CloudHSM**
  - Hardware Security Module
  - 사용자가 직접 암호화 키 관리

■ **`KMS(Key Management Service)`**
  - Encrypt EBS 제공
  - Used to provide encryption for Amazon EBS
  - 암호와 키에 관한 액세스 권한 -> 데이터 액세스 권한
  - CMK(Customer Master Key) 생성 및 제어
  - Can be used to encrypt data at rest

■ **`Secrets Manager`**
  - **프로그래밍 방식**으로 암호 변경, 보안 정보 보호 및 관리
  - RDS에서 관리
  - ex) A company has an application that runs on Amazon EC2 instances and uses an Amazon Aurora database. The EC2 instances connect to the database by using user names and passwords that are stored locally in a file. The company wants to minimize the operational overhead of credential management. What should a solutions architect do to accomplish this goal? -> **Use AWS Secrets Manager. Turn on automatic rotation**
    - 자동 회전: Secrets Manager는 암호의 정기적인 갱신(자동 회전)을 지원하여 보안 위험을 최소화합니다. 이는 사용자 이름과 암호가 노출되는 위험을 줄이는 데 매우 중요합니다. 보안 및 관리 용이성: Secrets Manager는 액세스 제어 및 로깅 기능을 제공하여 자격 증명 관리의 복잡성을 줄이고 운영 오버헤드를 최소화합니다. 직접 파일로 관리하는 방식보다 안전하고 효율적입니다.
    - Parameter Store : Does not support auto rotation, unless the customer writes it themselves. Secure string parameter only apply for Parameter Store. All the data in AWS Secrets Manager is encrypted. Parameter Store can also be used for credential management, but its security features and automatic rotation capabilities are not as robust as Secrets Manager.
    - S3 is a secure data storage, but it lacks features specific to credential management (e.g., automatic rotation, access control policies).
    - EBS is a block storage solution that is not suitable for credential management and is not efficient in terms of security and management.
  - ex) A company performs monthly maintenance on its AWS infrastructure. During these maintenance activities, the company needs to rotate the credentials for its Amazon RDS for MySQL databases across multiple AWS Regions. Which solution will meet these requirements with the LEAST operational overhead? -> **Store the credentials as secrets in AWS Secrets Manager. Use multi-Region secret replication for the required Regions. Configure Secrets Manager to rotate the secrets on a schedule.** (Rotate the credentials for its Amazon RDS for MySQL databases across multiple AWS Regions. LEAST operational overhead)
    - AWS Secrets Manager는 데이터베이스 자격 증명과 같은 민감한 정보를 안전하게 저장하고 관리하도록 설계되었습니다.
      - 다중 리전 비밀 복제: Secrets Manager는 여러 리전에 비밀을 자동으로 복제하여 리전 간 접근성을 제공합니다. 이는 문제에서 요구하는 여러 리전에서의 자격 증명 교체를 효율적으로 처리합니다.
      - 자동화된 교체: Secrets Manager는 일정 기반 또는 트리거 기반으로 비밀 교체를 자동화할 수 있는 기능을 제공합니다. 이를 통해 수동 개입 없이 정기적인 자격 증명 교체를 수행할 수 있어 운영 오버헤드를 최소화합니다.
    - B: AWS Systems Manager는 인스턴스 관리 및 구성 관리에 중점을 두며 Secrets Manager만큼 비밀 관리에 특화되어 있지 않습니다.
    - C: S3는 객체 저장소이며, 민감한 자격 증명을 저장하기에는 보안 수준이 낮습니다. EventBridge와 Lambda를 사용한 교체 방식은 복잡하고 Secrets Manager의 간편한 솔루션보다 비효율적입니다.
    - D: DynamoDB는 데이터베이스이며 Secrets Manager보다 비밀 관리 기능이 제한적입니다. 다중 리전 관리와 자동 교체 로직 구현이 Secrets Manager보다 복잡합니다.

■ **Shield**
  - DDoS 방지
  - Help protect applications running on AWS from DDoS attacks
    ex) Auto Scaling, CloudFront, Route53
  - Standard, Advanced
  - ex) Shield Standard : Designed to protect a workload from SQL injections, cross-site scripting, and DDoS attacks

■ **`WAF`**
  - SQL Injection, XSS 방지, L7 필터링
  - Contains built-in engines to protect web applications that run in the cloud from SQL injection attacks and cross-site scripting
  - Designed to protect a workload from SQL injections, cross-site scripting, and DDoS attacks

■ Security Hub
  - 중앙 집중식 보안 위치
  - A cloud security posture management(CSPM) service that aggregates alerts from various AWS services and partner products in a standardized format

■ Detective
  - 보안 문제 및 의심스러운 활동 조사

■ Abuse
  - 부정 사용 및 불법적 목적 사용 감지

## Module 10 (Monitoring and Automation)
■ **`Auto Scaling`**
  - Scheduled/Dynamic/Predictive
  - Auto Scaling Group
    - Min/Max/Desired
    - Improved health and availability of applications
    - Optimized performance and costs
  - ex) A company's application runs on Amazon EC2 instances behind an Application Load Balancer (ALB). The instances run in an Amazon EC2 Auto Scaling group across multiple Availability Zones. On the first day of every month at midnight, the application becomes much slower when the month-end financial calculation batch executes. This causes the CPU utilization of the EC2 instances to immediately peak to 100%, which disrupts the application. What should a solutions architect recommend to ensure the application is able to handle the workload and avoid downtime? -> **Configure an EC2 Auto Scaling scheduled scaling policy based on the monthly schedule.**

■ **`CloudWatch`** : Metrics, Logs, Alarm, Event, Rule, Target

■ **`CloudTrail`** : 계정 활동 추척, 이벤트 및 API 호출 기록 X-Ray 등
  - Enables customers to audit API calls in their AWS accounts
  - Can identify when an Amazon EC2 instance was terminated
  - Tracks API calls and user activity
- VPC Flow Logs : 수신 및 발신 트래픽 정보
  - Provides log information of the inbound and outbound traffic on network interfaces in a VPC
- Service Health Dashboard
  - Used to capture information about inbound and outbound traffic in an Amazon VPC

■ **`CloudFormation`**
  - **Infrastructure as Code(IaC)**
  - Change set

■ Quick Start

■ OpsWorks

## Module 11 (Caching)
■ **CloudFront** : CDN
  - ex) A global company hosts its web application on Amazon EC2 instances behind an Application Load Balancer (ALB). The web application has static data and dynamic data. The company stores its static data in an Amazon S3 bucket. The company wants to improve performance and reduce latency for the static data and dynamic data. The company is using its own domain name registered with Amazon Route 53. What should a solutions architect do to meet these requirements? -> **Create an Amazon CloudFront distribution that has the S3 bucket and the ALB as origins. Configure Route 53 to route traffic to the CloudFront distribution.**
  - Amazon CloudFront 배포를 생성하여 정적 데이터와 동적 데이터에 대한 성능을 향상시키고 대기 시간을 줄일 수 있습니다. S3 버킷과 ALB를 오리진으로 포함하는 CloudFront 배포를 생성하면, 정적 데이터와 동적 데이터를 효율적으로 캐시하고 배포할 수 있습니다. Route 53을 구성하여 트래픽을 CloudFront 배포로 라우팅하면, 사용자에게 더 빠르고 안정적인 서비스를 제공할 수 있습니다.
  - AWS Global Accelerator is primarily used to provide high availability and low latency, and is generally not recommended for use with CloudFront. / Using CloudFront and Global Accelerator together is too complex and not a configuration necessary for improving performance. / Configurations that use two domain names are difficult to manage and can be confusing for users.
  - AWS Global Accelerator vs CloudFront
    • They both use the AWS global network and its edge locations around the world
    • Both services integrate with AWS Shield for DDoS protection.
  - CloudFront
    • Improves performance for both cacheable content (such as images and videos)
    • Dynamic content (such as API acceleration and dynamic site delivery)
    • Content is served at the edge
  - Global Accelerator
    • Improves performance for a wide range of applications over TCP or UDP
    • Proxying packets at the edge to applications running in one or more AWS Regions.
    • Good fit for non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP
    • Good for HTTP use cases that require static IP addresses
    • Good for HTTP use cases that required deterministic, fast
  - Enables companies to deploy an application close to end users
  - ex) A solutions architect is designing a solution where users will be directed to a backup static error page if the primary website is unavailable. The primary website's DNS records are hosted in Amazon Route 53 where their domain is pointing to an Application Load Balancer (ALB). Which configuration should the solutions architect use to meet the company's needs while minimizing changes and infrastructure overhead? -> **Set up a Route 53 active-passive failover configuration. Direct traffic to a static error page hosted within an Amazon S3 bucket when Route 53 health checks determine that the ALB endpoint is unhealthy.** (Set up a Route 53 active-passive failover configuration. Direct traffic to a static error page hosted within an Amazon S3 bucket when Route 53 health checks determine that the ALB endpoint is unhealthy. The Solutions Architect can have both the primary website and the backup static error page in a single S3 bucket. All the Solutions Architect needs to do is set up a Route 53 active-passive failover configuration which will direct the traffic to the static error page when Route 53 health check detects that the ALB endpoint is unhealthy. This configuration meets the company's needs while minimizing changes and infrastructure overhead.)
  - ex) A company serves content to its subscribers across the world using an application running on AWS. The application has several Amazon EC2 instances in a private subnet behind an Application Load Balancer (ALB). Due to a recent change in copyright restrictions, the chief information officer (CIO) wants **to block access for certain countries.** Which action will meet these requirements? -> **Use Amazon CloudFront to serve the application and deny access to blocked countries.**
  - ex) Organizers for a global event want to put daily reports online as static HTML pages. The pages are expected to generate millions of views from users around the world. The files are stored in an Amazon S3 bucket. A solutions architect has been asked to design an efficient and effective solution. Which action should the solutions architect take to accomplish this? -> **Use Amazon CloudFront with the S3 bucket as its origin.** (Static content on S3 and hence Cloudfront is the best way.)
  - Expire contents
    - TTL
    - Change object name
    - Invalidate Object

## Module 12 (Decoupled Architecture)
■ **SQS(Simple Queue Service)**
  - 배치 요청, 비동기화
  - Used for reliably transmitting messages between components but is not designed for sending text or email messages
  - Can be used to decouple applications
  - Helps developers use loose coupling and reliable messaging between microservices
  - **Serverless**, 완전 관리형, **분리(Decouple)**, 송신 비용만 존재
  - Type
    - Standard vs FIFO
  - Features
    - Dead letter queue
    - Visibility timeout
    - Long polling(vs Short polling)

■ **SNS(Simple Notification Service)**
  - **Send both text and email messages from distributed applications**
    - ex) A company has an application that ingests incoming messages. Dozens of other applications and microservices then quickly consume these messages. The number of messages varies drastically and sometimes increases suddenly to 100,000 each second. The company wants to decouple the solution and increase scalability. Which solution meets these requirements? -> **Publish the messages to an Amazon Simple Notification Service (Amazon SNS) topic with multiple Amazon Simple Queue Service (Amazon SOS) subscriptions. Configure the consumer applications to process the messages from the queues** 
      - The number of messages varies drastically. Sometimes increases suddenly to 100,000 each second. <-> Kinesis Data Streams can handle this case but we should increase the more shards but not single shard. / Auto Scaling group not scale well because it need time to check the CPU metric and need time to start up the EC2 and the messages varies drastically. Example: we have to scale from 10 to 100 EC2. Our servers may be down a while when it was scaling. / Kinesis Data Streams can handle this case but we should increase the more shards but not single shard.
      - High availability and scalability: Problem situations require processing hundreds of thousands of messages per second and sudden traffic spikes. Amazon SNS and SQS are message queuing systems ideal for these requirements. SNS enables parallel processing by distributing messages to multiple SQS queues. SQS securely stores messages and allows consuming applications to process them as needed. Using multiple SQS queues allows you to easily increase processing power through horizontal scaling.
      - 1번 (Kinesis Data Analytics): 실시간 분석에 최적화되었지만, 단순한 메시지 소비에는 과도한 기능입니다.  2번 (EC2 Auto Scaling): EC2 인스턴스 기반은 관리 오버헤드가 높고, 트래픽 변동에 대한 응답 속도가 SQS보다 느릴 수 있습니다. 3번 (Kinesis Data Streams + Lambda + DynamoDB): 추가적인 데이터 저장 및 처리 계층이 필요하며, 메시지 처리 속도와 확장성 측면에서 SQS보다 비효율적일 수 있습니다.
    - ex) An application development team is designing a microservice that will convert large images to smaller, compressed images. When a user uploads an image through the web interface, the microservice should store the image in an Amazon S3 bucket, process and compress the image with an AWS Lambda function, and store the image in its compressed form in a different S3 bucket. A solutions architect needs to design a solution that uses durable, **stateless** components to process the images automatically. Which combination of actions will meet these requirements? (Choose two.)
      - **Create an Amazon Simple Queue Service (Amazon SQS) queue. Configure the S3 bucket to send a notification to the SQS queue when an image is uploaded to the S3 bucket.**
      - **Configure the Lambda function to use the Amazon Simple Queue Service (Amazon SQS) queue as the invocation source. When the SQS message is successfully processed, delete the message in the queue.**
      - Amazon SQS 사용: 이미지가 Amazon S3 버킷에 업로드되면 S3 버킷을 설정하여 해당 이벤트를 Amazon SQS 대기열에 알림으로 전송합니다. 이렇게 하면 확장성과 내구성이 뛰어난 비저장 구성 요소를 활용해 이미지 업로드 이벤트를 안정적으로 처리할 수 있습니다. Lambda 함수와 SQS 통합: AWS Lambda 함수를 구성하여 SQS 대기열을 이벤트 소스로 사용하도록 설정합니다. Lambda 함수는 대기열에서 이미지 처리 요청을 가져와 처리한 후 해당 메시지를 성공적으로 완료되면 자동으로 삭제합니다. 이 방식은 자동 확장과 무정지 처리를 가능하게 합니다.
  - Sends notifications two ways, A2A and A2P
    - A2A provides high-throughput, push-based, many-to-many messaging between distributed systems, microservices, and event-driven serverless applications. These applications include Amazon Simple Queue Service(SQS), Amazon Kinesis Data Firehose, AWS Lambda, and other TTPS endpoints.
    - A2P functionality lets you send messages to your customers with SMS texts, push notifications, and email.
  - 완전 관리형 Pub/Sub 메세징
  - Lambda
  - SQS
    - ex) A company is migrating a distributed application to AWS. The application serves variable workloads. The legacy platform consists of a primary server that coordinates jobs across multiple compute nodes. The company wants to modernize the application with a solution that maximizes resiliency and scalability. How should a solutions architect design the architecture to meet these requirements? -> **Configure an Amazon Simple Queue Service (Amazon SQS) queue as a destination for the jobs.** Implement the compute nodes with Amazon EC2 instances that are managed in an Auto Scaling group. Configure EC2 Auto Scaling based **on the size of the queue** <-> Schedule scaling policy doesn't make sense to schedule auto-scaling. / CloudTrail would be helpful in this case, at all. Primary server should not be in same Auto Scaling group with compute nodes. / EventBridge is not really used for this purpose, wouldn't be very reliable
      - 1번 옵션은 예약된 조정을 사용하도록 EC2 Auto Scaling을 구성합니다. 이는 작업의 양에 따라 컴퓨팅 노드의 수를 자동으로 조정할 수 없으므로, 탄력성과 확장성이 부족합니다. 3번 옵션은 AWS CloudTrail을 작업 대상으로 구성합니다. 이는 작업을 조정하는 데 적합하지 않습니다. 4번 옵션은 Amazon EventBridge(Amazon CloudWatch Events)를 작업 대상으로 구성합니다. 이는 작업을 조정하는 데 적합하지 않습니다. 또한 컴퓨팅 노드의 로드를 기반으로 EC2 Auto Scaling을 구성합니다. 이는 작업의 양에 따라 자동으로 컴퓨팅 노드의 수를 조정할 수 없으므로, 탄력성과 확장성이 부족합니다.
  - HTTP / HTTPS
  - Email
  - SMS

■ SES(Simple Email Service)
  - Focused on sending email messages, not text messages

■ MQ : 온프레미스에서 실행되는 개방형 프로토콜

## Module 13 (Microservices and Serverless)
■ **Microservices**
  - Container vs Virtual Machines
    - ECR(Elastic Container Registry)
      - Can a company use to store and manage Docker images
    - **`ECS(Elastic Container Service)`**
      - Auto Scaling : 워크로드 변형
      - ex) A company has a legacy application that processes data in two parts. The second part of the process takes longer than the first, so the company has decided to rewrite the application as two microservices running on Amazon ECS that can scale independently. How should a solutions architect integrate the microservices? -> **Implement code in microservice 1 to send data to an Amazon SQS queue. Implement code in microservice 2 to process messages from the queue.**
How should a solutions architect integrate the microservices?
    - EKS(Elastic Kubernetes Service)

## Module 14 (RTO/RPO and Backup Recovery)
■ Availability Concepts
  - High Availability
  - Backup
  - Disaster Recovery
  - 비용 순서 : 백업 및 복원 < 가장 저렴한 파일럿 라이트 < 저렴한 웜 스탠바이 < 비용이 많이 드는 다중 사이트

■ RTO, RPO

■ AWS Services for DR
  - Storage
    - S3(Cross-region Replication)
      - ex) A company is planning to migrate a business-critical dataset to Amazon S3. The current solution design uses a single S3 bucket in the us-east-1 Region with versioning enabled to store the dataset. The company's disaster recovery policy states that all data multiple AWS Regions. How should a solutions architect design the S3 solution? -> **Create an additional S3 bucket with versioning in another Region and configure cross-Region replication.**
    - **EBS(Snapshot, Copy)**
      - ex) A company wants to improve its ability to clone large amounts of production data into a test environment in the same AWS Region. The data is stored in Amazon EC2 instances on Amazon Elastic Block Store (Amazon EBS) volumes. Modifications to the cloned data must not affect the production environment. The software that accesses this data requires consistently high I/O performance. A solutions architect needs to minimize the time that is required to clone the production data into the test environment. Which solution will meet these requirements? -> **Take EBS snapshots of the production EBS volumes. Turn on the EBS fast snapshot restore feature on the EBS snapshots. Restore the snapshots into new EBS volumes. Attach the new EBS volumes to EC2 instances in the test environment.** (Amazon EBS fast snapshot restore (FSR) enables you to create a volume from a snapshot that is fully initialized at creation. This eliminates the latency of I/O operations on a block when it is accessed for the first time. Volumes that are created using fast snapshot restore instantly deliver all of their provisioned performance.)
        - 빠른 복제: EBS의 빠른 스냅샷 복원 기능을 활용하면, 데이터 복제 시간을 크게 단축시킬 수 있습니다. 이는 대량의 프로덕션 데이터를 신속하게 테스트 환경으로 이동해야 하는 상황에 적합합니다. 데이터 독립성: 새 EBS 볼륨을 생성하고 복원하여 테스트 환경에 연결하므로, 테스트 환경 내에서 데이터를 수정하더라도 프로덕션 환경에는 영향을 미치지 않습니다. 이는 데이터의 무결성과 안정성을 보장합니다. 고성능 I/O: 새 EBS 볼륨은 필요에 따라 성능을 최적화할 수 있으며, 테스트 환경에서 요구되는 높은 I/O 성능을 유지할 수 있습니다.
        - 1번과 3번은 스냅샷 복원 과정에서 추가적인 시간이 소요될 수 있으며, 새 볼륨을 사용하지 않아 테스트 환경의 데이터 변경이 프로덕션에 간접적으로 영향을 줄 위험이 있습니다. 2번은 다중 연결 기능이 I/O 성능 향상에 직접적인 도움이 되지 않고, 프로덕션 볼륨을 직접 연결하므로 데이터 변경에 따른 프로덕션 환경의 위험성이 있습니다.
    - Snowball
      - ex) A company uses NFS to store large video files in on-premises network attached storage. Each video file ranges in size from 1 MB to 500 GB. The total storage is 70 TB and is no longer growing. The company decides to migrate the video files to Amazon S3. The company must migrate the video files as soon as possible while using **the least possible network bandwidth**. Which solution will meet these requirements? -> **Create an AWS Snowball Edge job. Receive a Snowball Edge device on premises. Use the Snowball Edge client to transfer data to the device. Return the device so that AWS can import the data into Amazon S3** <-> File Gateway uses the Internet, so maximum speed will be at most 1Gbps, so it'll take a minimum of 6.5 days and you use 70TB of Internet bandwidth. / You can achieve speeds of up to 10Gbps with Direct Connect. Total time 15.5 hours and you will use 70TB of bandwidth. Direct Connect does not use your Internet bandwidth, as you will have a dedicate peer to peer connectivity between your on-prem and the AWS Cloud, so technically, you're not using your "public" bandwidth.
        - Snowball can copy with speed and doesn't require internet. On a Snowball Edge device you can copy files with a speed of up to 100Gbps. 70TB will take around 5600 seconds, so very quickly, less than 2 hours. The downside is that it'll take between 4-6 working days to receive the device and then another 2-3 working days to send it back and for AWS to move the data onto S3 once it reaches them. Total time: 6-9 working days. Bandwidth used: 0.
        - The company has 70TB of total storage, and needs to migrate data as quickly as possible while minimizing network bandwidth. AWS Snowball Edge is a service that allows you to quickly and cost-effectively transfer large amounts of data to AWS. You can use a Snowball Edge device on-premises to transfer data, then return the device to AWS to import the data into Amazon S3. This method is the most efficient way to migrate data quickly while minimizing network bandwidth. Other options use a lot of network bandwidth or result in longer data transfer times. Therefore, the correct solution is to create an AWS Snowball Edge job, transfer data on-premises using a Snowball Edge device, and then return the device to AWS to import the data into Amazon S3.
        - Direct upload via AWS CLI causes significant network load and takes a long time for large amounts of data. The S3 File Gateway allows access to S3 via NFS, but it still requires data transfer over the network, making it inefficient for large-scale migrations. In particular, option 4 adds complexity due to the additional Direct Connect configuration.
        - 1번 (A): AWS CLI를 통한 직접 업로드는 대용량 데이터에 대한 네트워크 부하가 매우 커지고 시간이 오래 걸립니다. 3번 (C) & 4번 (D): S3 파일 게이트웨이는 NFS를 통해 S3에 접근하도록 하지만, 여전히 네트워크를 통해 데이터를 전송해야 하므로 대용량 마이그레이션에는 효율적이지 않습니다. 특히 4번의 경우 Direct Connect 설정 추가로 복잡성이 증가합니다.
    - DataSync : Migration, 감사, 파일 권한 보존
    - CloudEndure
  - Compute
    - **Custom AMI**
    - Custom Container Image
  - Networking
    - Route53(Load Balancing, Failover)
    - ELB(Load Balancing, Failover)
    - VPC
    - Region

## Module 15 (Migration)
■ Application Discovery Service

■ **`Application Migration Service`**

■ Data Migration Service(DMS)
  - used to migrate a company's on-premises MySQL database to Amazon RDS
  - **Snowball**
    - PetaByte(60TB)
    - The transfer of data from the Snowball Edge appliance into Amazon S3 at **No Cost**
    - ex) A recently acquired company is required to build its own infrastructure on AWS and migrate multiple applications to the cloud within a month. Each application has approximately 50 TB of data to be transferred. After the migration is complete, this company and its parent company will both require secure network connectivity with consistent throughput from their data centers to the applications. A solutions architect must ensure one-time data migration and ongoing network connectivity. Which solution will meet these requirements? -> **AWS Snowball for the initial transfer and AWS Direct Connect for ongoing connectivity.** (Definitely snowball for data transfer. As per the connectivity, 4 requirements here: Complete WITHIN a month, secure, consistent and BOTH companies need access. DirectConnect provides consistency, but it's not secure (it's private, that's different), it's a 1-to-1 direct connection and then you have up to 90 days to set it up (on average it takes more than 1 month, or at least that's what they said at the course). Site-to-Site VPN is secure and can be setup immediately in multiple sites. It's not consistent, but it can achieve consistency with Accelerated Site-to-Site VPN ("Accelerated Site-to-Site VPN makes user experience more consistent by using the highly available and congestion-free AWS global network.") So I'd go for Snowball and VPN (possibly Accelerated)))
  - **`Snowball Edge`** : 100TB(80TB)
    - 인터넷 연결이 간헐적이거나 없는 위치에서 데이터 수집
  - **Snowmobile** : ExaByte
    - Designed for large-scale data transfers

■ Server Migration Service

■ **`Storage Gateway`**
  - On-Premise Data Storage -> AWS Cloud 연결
  - **A hybrid cloud storage service** that provides on-premises users access to virtually unlimited cloud storage
  - 종류
    - Tape Gateway
    - Volumne Gateway
    - FSx File Gateway
      - ex) A company is migrating from an on-premises infrastructure to the AWS Cloud. One of the company's applications stores files on a Windows file server farm that uses Distributed File System Replication (DFSR) to keep data in sync. A solutions architect needs to replace the file server farm. -> **Amazon FSx**
    - **S3 File Gateway**
      - ex) A company is running an SMB file server in its data center. The file server stores large files that are accessed frequently for the first few days after the files are created. After 7 days the files are rarely accessed. The total data size is increasing and is close to the company's total storage capacity. A solutions architect must increase the company's available storage space without losing low-latency access to the most recently accessed files. The solutions architect must also provide file lifecycle management to avoid future storage issues. Which solution will meet these requirements? -> **Create an Amazon S3 File Gateway to extend the company's storage space. Create an S3 Lifecycle policy to transition the data to S3 Glacier Deep Archive after 7 days**
        - Amazon S3 파일 게이트웨이: 온프레미스 환경에서 S3에 파일을 저장하는 기능을 제공하며, 기존 SMB 환경과의 호환성을 유지하면서 저장 공간을 확장할 수 있습니다. S3 수명 주기 정책: 7일 후 데이터를 저렴한 저장소인 S3 Glacier Deep Archive로 자동 이동시켜 저장 비용을 절감하면서 장기 보관을 가능하게 합니다. S3 Glacier Deep Archive는 매우 저렴한 비용으로 장기간 데이터 보관에 최적화되어 있습니다.
        - AWS DataSync : Data copying method incurs initial data movement costs and does not guarantee real-time access. / FSx : It only expands storage space and lacks lifecycle management features. / Installing utilities on user computers increases management complexity and can make centralized management and access control difficult.

■ **Consulting Partner** : Migration 전문가

■ **Professional Services**

■ **Outposts**
  - AWS Cloud Service -> On-Premise 삽입 및 하이브리드 아키텍처 확장
  - 낮은 대기 시간 또는 로컬 시스템 상호 종속성이 필요한 애플리케이션에 유용
  - Supports a hybrid architecture that gives users the ability to extend AWS infrastructure, AWS services, APIs, and tools to data centers, co-location environments, or on-premises facilities

■ **Local Zones**
  - 대기 시간 최소화, 연결 중단 X
  - A type of AWS infrastructure deployment that places compute, storage, database, and other services closer to a specific geographic area
  - Designed to run workloads that require single-digit millisecond latency, like video rendering and graphics intensive, virtual desktop applications

■ **Wavelength**
  - Designed specifically to reduce the latency between devices and applications hosted on AWS by placing AWS compute and storage services at the edge of the 5G network

■ Best Practice

## Module 16 (Developer Tools)
■ CodeCommit : 버전 관리

■ CodeDeploy

■ CodePipeline

■ CodeGuru : 클라우드 기반 코드 검토 및 애플리케이션 성능 모니터링 서비스

■ EventBridge : An event bus service that enables event-driven architectures

## Module 17 (Business Productivity)
■ WorkDocs

■ WorkMail

■ **WorkSpaces**
  - Provide managed Windows virtual desktops and applications to its remote employees over secure network connections
  - Creates virtual desktops that can be used to create entire working environments for you and your team

■ **Chime** : 오디오와 비디오를 송수신하고 콘텐츠 공유를 허용하는 실시간 미디어 애플리케이션 구축

■ **`AppStream 2.0`** : 원격 작업 액세스를 지원하는 완전관리형의 비영구 데스크톱 및 애플리케이션 서비스
  - Provide managed Windows virtual desktops and applications to its **remote employees over secure network connections**
  - Focused on hosting individual applications on AWS

■ Marketplace

## Module 18 (Application)
■ API Gateway : 규모와 관계없이 REST 및 WebSocket API를 생성, 게시, 유지, 모니터링 및 보호하기 위한 서비스
  - A development team wants to publish and manage web services that provide REST APIs.
  - ex) A company is building an ecommerce web application on AWS. The application sends information about new orders to an Amazon API Gateway REST API to process. The company wants to ensure that orders are processed in the order that they are received. Which solution will meet these requirements? -> Use an API Gateway integration to send a message to an Amazon Simple Queue Service (Amazon SQS) FIFO queue when the application receives an order. **Configure the SQS FIFO queue to invoke an AWS Lambda function for processing**
    - (Amazon SNS) : SNS does not guarantee message delivery order. (API Gateway Authorizer) : Request blocking has no bearing on order processing order. (SQS Standard Queue) : Standard queues do not guarantee message processing order.

■ SWF

■ Elastic Transcoder

■ Step Functions

■ Cloud Search

## Module 19 (Analytics)
■ EMR : Elastic MapReduce, Hadoop Framework

■ Kinesis : Realtime Video & Data Stream Service 수집, 처리, 분석
  - ex) A company captures clickstream data from multiple websites and analyzes it using batch processing. The data is loaded nightly into Amazon Redshift and is consumed by business analysts. The company wants to move towards near-real-time data processing for timely insights. The solution should process the streaming data with minimal effort and operational overhead. Which combination of AWS services are MOST cost-effective for this solution? (Choose two.) -> **Amazon Kinesis Data Streams, Amazon Kinesis Data Analytics**

■ Elastic Search

■ **QuickSight** : Serverless, 시각적 보고서 생성, BI 서비스
  - ex) A company hosts a data lake on AWS. The data lake consists of data in Amazon S3 and Amazon RDS for PostgreSQL. The company needs a reporting solution that provides data visualization and includes all the data sources within the data lake. Only the company's management team should have full access to all the visualizations. The rest of the company should have only limited access. Which solution will meet these requirements? -> **Create an analysis in Amazon QuickSight. Connect all the data sources and create new datasets. Publish dashboards to visualize the data. Share the dashboards with the appropriate users and groups.**
    - Data lake on AWS.
    - Consists of data in Amazon S3 and Amazon RDS for PostgreSQL.
    - The company needs a reporting solution that provides data VISUALIZATION and includes ALL the data sources within the data lake.
    - 데이터 통합 및 시각화, 권한 관리, 사용자 친화적 접근성
    - 옵션 1은 IAM 역할을 언급하지만, 사용자 그룹 기반의 세분화된 액세스 관리가 덜 명확합니다. AWS Glue 테이블과 크롤러를 생성은 주로 ETL 프로세스와 보고서 생성에 초점을 맞추고 있어, 직접적인 데이터 시각화와 사용자별 액세스 제어의 효율성이 떨어집니다.

■ Data Pipeline : 서로 다른 사용자의 수백 건 요청 동시 처리

■ **`RedShift`**
  - Data Warehouse
  - Column 형식
  - OLAP(Online Analytical Processing) 용도

■ Glue : Serverless 관리형 ETL(Extract, Transform, Load)
  - Is used specifically for extract, transform, and load (ETL) data

■ Managed Blcokchain

## Module 20 (AI)
■ Machine Learning

■ **Rekognition** : Vision
  - Use to organize, characterize, and search large numbers of images

■ **`Polly`** : TTS
  - Used to turn text into lifelike speech

■ **`Lex`** : ChatBot, 자동 음성 인식, 자연어 이해 기능

■ Transcribe : STT, 자동 음성 인식, ASR 딥러닝 처리 사용

■ Translate : 번역

■ Comprehend : NLP, 이메일 메세지 감정분석 등 사용

■ SageMaker : 머신 러닝 모델 구축, 훈련, 배포, 기계 학습

■ ForeCast

■ Code Guru

■ Kendra : 문서 검색 서비스

■ Personalize : 개인화 추천

■ Textract : 텍스트 추출

■ DeepRacer : 자율 주행 경주용 자동차

■ Augmented AI(A2I) : 전문 인력 없이 모델 빌드

■ Ground Station : 인공위성 빌림

■ Fraud Detector : 잠재적인 온라인 사기 행위 식별

## Module 21 (Mobile)
■ Mobile Hub

■ Mobile SDK

■ Mobile Analytics

■ PinPoint : 마케팅 커뮤니케이션 서비스

■ Device Farm

## Module 22 (IOT)
■ IoT Platform

■ Greengrass

■ IoT Button

## Mobule 23 (Game)
■ GameLift

■ Lumberyard

## 문제
- <https://www.examtopics.com/exams/amazon/aws-certified-solutions-architect-associate-saa-c03/view/>
   - ~20까지 무료
- <https://www.examtopics.com/exams/amazon/aws-certified-solutions-architect-associate-saa-c02/view/>
- <https://www.koreadumps.com/SAA-C03-KR-practice-test.html>
- <https://newbt.kr/%EC%8B%9C%ED%97%98/AWS%20Solutions%20Architect%20Associate>

## 참고
- <https://velog.io/@gagaeun/AWS-SAA-C03-Examtopics-%ED%97%B7%EA%B0%88%EB%A6%AC%EB%8A%94-%EB%AC%B8%EC%A0%9C-%EC%A0%95%EB%A6%AC>
- <https://www.examtopics.com/discussions/amazon/>
- <https://github.com/serithemage/AWSCertifiedSolutionsArchitectUnofficialStudyGuide>
- <https://blog.naver.com/PostView.naver?blogId=gam_jaong&logNo=222909260062&parentCategoryNo=&categoryNo=18&viewDate=&isShowPopularPosts=false&from=postList>
- <https://wbluke.tistory.com/53>
- <https://onborders.tistory.com/>