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
    - EC2 instance store
      - Is ephemeral and is deleted when an Amazon EC2 instance is stopped or terminated
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

■ **`Fargate`** : **Conainer(ECS, EKS)**를 위한 Serverless 컴퓨팅 엔진
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
  - **Athena** : Serverless 대화형 쿼리 서비스

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
  - Gateway Load Banlancer

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
  - Enables companies to deploy an application close to end users
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
  - Sends notifications two ways, A2A and A2P
    - A2A provides high-throughput, push-based, many-to-many messaging between distributed systems, microservices, and event-driven serverless applications. These applications include Amazon Simple Queue Service(SQS), Amazon Kinesis Data Firehose, AWS Lambda, and other TTPS endpoints.
    - A2P functionality lets you send messages to your customers with SMS texts, push notifications, and email.
  - 완전 관리형 Pub/Sub 메세징
  - Lambda
  - SQS
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
    - **EBS(Snapshot, Copy)**
    - Snowball
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
    - **S3 File Gateway**

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

■ SWF

■ Elastic Transcoder

■ Step Functions

■ Cloud Search

## Module 19 (Analytics)
■ EMR : Elastic MapReduce, Hadoop Framework

■ Kinesis : Realtime Video & Data Stream Service 수집, 처리, 분석

■ Elastic Search

■ **QuickSight** : Serverless, 시각적 보고서 생성, BI 서비스

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
   - ~25까지 무료
- <https://www.examtopics.com/exams/amazon/aws-certified-solutions-architect-associate-saa-c02/view/>
- <https://www.koreadumps.com/SAA-C03-KR-practice-test.html>
- <https://newbt.kr/%EC%8B%9C%ED%97%98/AWS%20Solutions%20Architect%20Associate>

## 참고
- <https://www.examtopics.com/discussions/amazon/>
- <https://github.com/serithemage/AWSCertifiedSolutionsArchitectUnofficialStudyGuide>
- <https://blog.naver.com/PostView.naver?blogId=gam_jaong&logNo=222909260062&parentCategoryNo=&categoryNo=18&viewDate=&isShowPopularPosts=false&from=postList>
- <https://wbluke.tistory.com/53>
- <https://onborders.tistory.com/>