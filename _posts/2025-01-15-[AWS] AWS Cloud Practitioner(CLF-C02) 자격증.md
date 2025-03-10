---
title: '[AWS] AWS Cloud Practitioner(CLF-C02) 자격증'
date: 2025-01-15 18:10:00 +0900
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
    - ex) AWS has the ability to achieve lower pay-as-you-go pricing by aggregating usage across hundreds of thousands of users. -> **High economies of scale**
  - **Stop guessing capacity** : 용량 추정 불필요
    - Compute capacity that is adjusted on demand
    - No longer having to guess what capacity will be required
    - Reduced or eliminated tasks for hardware troubleshooting, capacity planning, and procurement
    - The ability to use the **pay-as-you-go model**
      - There are **no upfront cost commitments**
      - There are no infrastructure operating costs X -> EC2에 해당, AWS에 해당하지 않음
    - ex) An online retail company has seasonal sales spikes several times a year, primarily around holidays. Demand is lower at other times. The company finds it difficult to predict the increasing infrastructure demand for each season. **Elasticity and Pay-as-you-go pricing** would most benefit the company.
    - ex) An online retail company wants to migrate its on-premises workload to AWS. The company needs to automatically handle a seasonal workload increase in a cost- effective manner. **Pay-as-you-go pricing and Auto Scaling policies** will help the company meet this requirement.
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
    - ex) An online company was running a workload on premises and was struggling to launch new products and features. After migrating the workload to AWS, the company can **quickly launch products and features and can scale its infrastructure as required.**
  - **Scalability**
  - **`Elasticity`**
    - Resource Elasticity : An AWS value proposition that describes a user’s ability to scale infrastructure based on demand
    - Time-Bases, Volume-Based, Predictive-Based
    - The ability to rightsize resources as demand shifts
    - How easily resources can be procured when they are needed
    - Helps users eliminate underutilized CPU capacity
    - ex) A company is using Amazon EC2 Auto Scaling to scale its Amazon EC2 instances. **Elasticity** is the benefit of the AWS cloud this example illustrate.
  - **High Availability and Fault Tolerance**
    - Operational resilience (운영 회복력)
    - An architecture's ability to withstand failures with minimal downtime
    - Is shown by an architecture's ability to withstand failures with minimal downtime
  - **`Agility`**
    - The speed at which AWS resources are implemented
    - The ability to experiment quickly
    - ex) A company's on-premises application deployment cycle was 3-4 weeks. After migrating to the AWS Cloud, the company can deploy the application in 2-3 days. This company can have **Agility** experienced by moving to the AWS Cloud.
    - ex) A company needs to deliver new website features quickly in an iterative manner to minimize the time to market. **Agility** represent this requirement.
    - ex) The AWS infrastructure consists of isolated AWS Regions with independent Availability Zones that are connected with low-latency networking and redundant power supplies.

■ **The Well Architected Framework(WAF)**
  - **Operational Excellence(운영 우수성)** : **Anticipate failure, 프로세스 개선**
    - ex) 코드로 작업 수행, 문서에 주석 추가, 실패 예측, 되돌릴 수 있는 소규모 변경 자주 수행, 배포 파이프 라인을 사용한 변경 자동화, 트리거된 이벤트에 대한 응답, 회사는 구성 요소를 정기적으로 업데이트하고 작고 되돌릴 수 있는 증분으로 변경할 수 있도록 AWS 워크로드를 설계
    - ex) A company is designing its AWS workloads so that components can be updated regularly and so that changes can be made in small, reversible increments.
  - **Security(보안)** : **암호화, 데이터 무결성 검사**
    - ex) AWS Config를 사용하여 추적 가능성 활성화
    - ex) Enhanced security
    - ex) Using AWS Config to record, audit, and evaluate changes to AWS resources to enable **traceability**.
    - ex) A company wants to protect its AWS Cloud information, system,s and assets while performing risk assessment and migration tasks.
    - ex) A company has designed its AWS Cloud infrastructure to run its workloads effectively. The company also has protocols in place to continuously improve supporting processes.
  - **Reliability(안정성/신뢰성)** : **기능 수행, 복구, 가용성(High Availability)**
    - ex) 비정상 애플리케이션을 자동으로 복구, 가용성(High Availability) 유지, 가동 중지 시간 최소화, 장애로부터 신속하게 복구
    - ex) Testing recovery procedures
    - ex) Learn to improve from operational failures
    - ex) A system automatically recovers from failure when a company launches its workload on the AWS Cloud services platform.
    - ex) Pillar of the AWS Well-Architected Framework refers to the ability of a system to recover from infrastructure or service disruptions and dynamically acquire computing resources to meet demand.
    - ex) A company wants to launch its workload on AWS and requires the system to automatically recover from failure.
    - ex) A company implements an Amazon EC2 Auto Scaling policy along with an Application Load Balancer to automatically recover unhealthy applications that run on Amazon EC2 instances.
    - ex) Installing the database engine when a workload is running in Amazon RDS
    - ex) A company wants to increase its ability to recover its infrastructure in the case of a natural disaster.
  - **Performance Efficiency(성능 효율성)** : **Serverless, 고급 기술**
    - ex) 워크 로드 및 메모리 요구 사항에 따라 적합한 Amazon EC2 유형을 사용해 정보에 기반한 의사 결정을 수립하고 비즈니스 요구가 진화함에 따라 효율성 유지
    - ex) A company is planning to replace its physical on-premises compute servers with AWS serverless compute services. The company wants to be able to take advantage of advanced technologies quickly after the migration.
  - **`Cost Efficiency = Cost Optimization(비용 최적화)`**
    - ex) EC2 서버 비용 효과적인 크기 선택
    - ex) Compute instances can be launched and terminated as needed to optimize costs
    - ex) A company does not want to rely on elaborate forecasting to determine its usage of compute resources. Instead, the company wants to pay only for the resources that it uses. The company also needs the ability to increase or decrease its resource usage to meet business requirements.
  - Sustainability(지속 가능성)

■ Architecture Design Principle
  - ex) A company is running a monolitic on-premises application that does not scale and is difficult to maintain. The company has a plan to migrate the application to AWS and divide the application into microservices. -> **Implement loosely coupled dependencies.**
  - ex) Need to isolate failures between dependent components in the AWS Cloud ->** Loosely couple components**
  - ex) A benefit of decoupling an AWS Cloud architecture -> **Ability to upgrade components independently
  - ex) **Avoid monolithic architecture by segmenting workloads****
  - ex) A Well-Architected review helps identify design gaps and helps evaluate design decisions and related documents.

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
    - ex) A company wants to run a NoSQL database on Amazon EC2 instances. -> **Patch the physical infrastructure that hosts the EC2 instances.**
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
    - ex) An ecommerce company has migrated its IT infrastructure from an on-premises data center to the AWS Cloud. **Cost of application software licenses** is the company's direct responsibility.
  - **`A shared responsibility between AWS and its customers`**
    - Patch Management
    - Configuration Management
    - Cloud Awareness and Training
    - Resource configuration management
    - Employee Awareness and Training
    - ex) A company needs to transfer data between an Amazon S3 bucket and an on-premises application. The company is responsible for **the security of this data**, according to the AWS shared responsibility model.
    - ex) A company is using an Amazon RDS DB instance for an application that is deployed in the AWS Cloud. The company needs regular patching of the operating system of the server where the DB instance runs. The company is responsible for **Establishing a regular maintenance window that tells AWS when to patch the DB instance operating system.**

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
      - ex) A company is planning its migration to the AWS Cloud. The company is identifying its capability gaps by using the AWS Cloud Adoption Framework (AWS CAF) perspectives. -> Align
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
    - ex) A company has a single Amazon EC2 instance. The company wants to adopt a highly available architecture. The company can **Scale horizontally across multiple Availability Zones** to meet this requirement.
    - ex) A company wants to ensure that two Amazon EC2 instances are in separate data centers with minimal communication latency between the data centers. The company can **Place the EC2 instances in two separate Availability Zones** within the same AWS Region to meet this requirement.
  - **Region**
    - 물리적 위치
    - Compliance, Latency, AZ, Costs 고려
    - A component of the AWS Global Infrastructure
    - Region 종속 Service : EC2, Elastic Beanstalk, Lambda, Rekognition
    - Global Service : IAM, Route53, CloudFront, WAF
    - ex) Physical location of the AWS global infrastructure
    - ex) A company wants to migrate its on-premises relational databases to the AWS Cloud. The company wants to use infrastructure as close to its current geographical location as possible. The company should use AWS Regions to select its Amazon RDS deployment area.
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
      - ex) A company should use RDS, Elastic File System(EFS) to read and write data that changes frequently.
    - EC2 instance store
      - Is ephemeral and is deleted when an Amazon EC2 instance is stopped or terminated
  - Instance Size & Type
    - M5, C5, R5, T2 & T3, G3 & P3, I2, D2
  - Scale Up(수직 확장 : 인프라 추가 없이), Scale Out(수평 확장 : 인프라 추가)
  - **Pricing**
    - **`On-Demand Instance`** : 필요에 따라 인스턴스를 생성하고 제거
      - ex) An e-learning platform needs to run an application for 2 months each year. The applicaton will be deployed on Amazon EC2 inastances. Any application downtime during those 2 months must be avoided. **On-Demand Instances** will meet these requirements most cost-effectively.
      - ex) **On-Demand Instance** is the MOST cost efficient for an uninterruptible workload that runs once a year for 24 hours.
      - ex) A company needs to continuously run an experimental workload on an Amazon EC2 instance and stop the instance after 12 hours.
      - ex) A company needs to run an application on Amazon EC2 instances. The instances cannot be interrupted at any time. The company needs an instance purchasing option that requires no long-term commitment or upfront payment. **On-Demand Instances** will meet these requirements most cost-effectively. **(Cannot be interrupted(Spot Instances), No long-term commitment(Reserved Instances), or upfront payment(Dedicated Hosts))**
      - ex) A user is comparing purchase options for an application that runs on Amazon EC2 and Amazon RDS. The application cannot sustain any interruption. The application experiences a predictable amount of usage, including some seasonal spikes that last only a few weeks at a time. It is not possible to modify the application. -> Buy **`Reserved Instances`** for the predicted amount of usage throughout the year. Allow any seasonal usage to run at an **`On-Demand`** rate.
    - **`Reserved Instance(RI)`** : 결제 할인 옵션, 일정 사용량 악정 X, 예측 가능
      - ex) An online gaming company needs to choose a purchasing option to run its Amazon EC2 instances for 1 year. The web traffic is consistent, and any increases in traffic are predictable. The EC2 instances must be online and available without any disruption. **Reserved Instances** will meet these requirements most cost-effectively.
      - ex) A company wants to make an upfront commitment for continued use of its production Amazon EC2 instances in exchange for a reduced overall cost. **Reserved Instances** and Savings Plans meet these requirements with the lowest cost. (Uninterruptible + Years)
      - ex) A company has a compute workload that is steady, predictable, and uninterruptible. **Reserved Instances** and Savings Plans meet these requirements most cost-effectively.
      - ex) A company plans to migrate its application to AWS and run the application on Amazon EC2 instances. The application will have continuous usage for 1 year. **Reserved Instances** will meet these requirements most cost-effectively.
      - ex) A user is comparing purchase options for an application that runs on Amazon EC2 and Amazon RDS. The application cannot sustain any interruption. The application experiences a predictable amount of usage, including some seasonal spikes that last only a few weeks at a time. It is not possible to modify the application. -> Buy **Reserved Instances** for the predicted amount of usage throughout the year. Allow any seasonal usage to run at an On-Demand rate.
      - ex) A company has a database server that is always running. The company hosts the server on Amazon EC2 instances. The instance sizes are suitable for the workload. The workload will run for 1 year. **Standard Reserved Instances** will meet these requirements most cost-effectively. There is no change on the instance spec for a year, no need to use convertible RI.
      - ex) A company has a workload that will run continuously for 1 year. The workload cannot tolerate service interruptions. **All Upfront Reserved Instances** will be most cost-effective.
      - ex) A company needs an Amazon EC2 instance for a rightsized database server that must run constantly for 1 year. Standard Reserved Instance will meet these reqruiements most cost-effectively.
    - **Spot Instance** : 공급/수요에 따라 조정, 짧은 시간, CPU 사용량에 따라 확장
      - ex) A company runs thousands of simultaneous simulations using AWS Batch. Each simulation is stateless, is fault tolerant, and runs for up to 3 hours. **Spot Instances** enables the company to optimize costs and meet these requirements.
      - ex) A company has a test AWS environment. A company is planning on testing an application within AWS. The application testing can be interrupted and does not need to run continuously. **Spot instances** will meet these requirements most cost-effectively.
      - ex) A company is deploying a machine learning (ML) research project that will require a lot of compute power over several months. The ML processing jobs do not need to run at specific times. **Spot instances** will meet these requirements at the lowest cost.
      - ex) Interrupt a running Amazon EC2 instance if capacity becomes temporarily unavailable
    - **Dedicated Instance**
      - Dedicated Hosts : 코어당 소프트웨어 라이선스
      - After selecting an Amazon EC2 Dedicated Host reservation, **All upfront payment** pricing option would provide the largest discount.
      - ex) A company owns per-core software licenses. **Dedicated Hosts** must the company use for this license type.
    - **Savings Plan** : 1년 또는 3년 기간 동안 일정한 컴퓨팅 사용량 약정
      - ex) A company wants to make an upfront commitment for continued use of its production Amazon EC2 instances in exchange for a reduced overall cost. **Reserved Instances and Savings Plans** meet these requirements with the lowest cost. (Uninterruptible + Years)
      - ex) A company has a compute workload that is steady, predictable, and uninterruptible. **Reserved Instances and Savings Plans** meet these requirements most cost-effectively.
      - ex) A company has an uninterruptible application that runs on Amazon EC2 instances. The application constantly processes a backlog of files in an Amazon Simple Queue Service (Amazon SQS) queue. This usage is expected to continue to grow for years. **Savings Plans** is the most cost-effective EC2 instance purchasing model to meet these requirements.
      - ex) A company wants to run its workload on Amazon EC2 instances for more than 1 year. This workload will run continuously. **Savings Plans** offers a discounted hourly rate compared to the hourly rate of On-Demand Instances
      - ex) A company is launching an ecommerce application that must always be available. The application will run on Amazon EC2 instances continuously for the next 12 months. **Savings Plans** is the most cost-effective instance purchasing option that meets these requirements.
    - **Pay-as-you-go** : 종량제 가격 측정, 인프라 선행 투자 X
      - ex) A company wants to eliminate the need to guess infrastructure capacity before deployments. The company also wants to spend its budget on cloud resources only as the company uses the resources. **Pay-as-you-go pricing** matches the company's requirements.
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
  - ex) A company needs to perform data processing once a week that typically takes about 5 hours to complete. -> The company should use **EC2** for this workload.
  - ex) A company plans to deploy containers on AWS. The company wants full control of the compute resources that host the containers.
  - ex) A company launched an Amazon EC2 instance with the latest Amazon Linux 2 Amazon Machine Image (AMI). **Use Amazon EC2 Instance  Connect and AWS Systems Manager Session Manager can a system administrator take to connect to the EC2 instance.**
  - ex) A company would like to host its MySQL databases on AWS and maintain full control over the operating system, database installation, and configuration. The company should **use EC2 to host the databases.**
  - ex) A company is creating a document that defines the operating system patch routine for all the company's systems. The company should **include EC2 instances and Elastic Container Service(ECS) instances in thie document.**

■ **LightSail**
  - **가상 프라이빗 서버(Virtual Private Server)**
  - ex) A company wants to migrate a small website and database quickly from on-premises infrastructure to the AWS Cloud. The company has limited operational knowledge to perform the migration. **Lightsail** supports this use case.

■ **`Elastic BeanStalk`**
  - **간단하게 서버 배포**
  - 프로비저닝 필요 없이 빠르게 애플리케이션을 AWS에 업로드
  - 용량 조정(프로비저닝), 로드 밸런싱, Auto Scaling, 애플리케이션 상태 모니터링
  - ex) A developer wants to deploy an application quickly on AWS without manually creating the required resources.
  - ex) A company wants the ability to quickly upload its applications to the AWS Cloud without needing to provision underlying resources. **Elastic Beanstalk** will meet these requirements.

■ **Lambda** : **Serverless** (코드가 서버에서 실행되지만, 프로비저닝하거나 관리 필요 없음)
  - 비용 청구 기준 : 실행시간, 요청 수
  - 회사의 책임 : 코드 내부의 보안, 코드 작성 및 업데이트
  - **The responsibility of a company that is using AWS Lambda**
    - Security inside of code
    - Writing and updating of code
  - ex) A company wants to migrate a critical application to AWS. The application has a short runtime. The application is invoked by changes in data or by shifts in system state. The company needs a compute solution that maximizes operational efficiency and minimizes the cost of running the application. The company should use **Lambda** to meet these requirements.

■ **`Fargate`** : **Conainer(ECS, EKS)**를 위한 Serverless 컴퓨팅 엔진
  - A **serverless** compute engine for containers
  - ex) A company is running and managing its own Docker environment on Amazon EC2 instances. The company wants an alternative to help manage cluster size, scheduling, and environment maintenance.
  - ex) A company needs to install an application in a Docker container. **Fargate** eliminates the need to provision and manage the container hosts.

■ Batch


## Module 3 (The Simiplest Architecture)
■ **`S3`**
  - **Severless**
  - Data Encryption 제공
  - Provides highly durable object storage
  - Used to host static websites
  - ex) A company wants to query its server logs to gain insights about its customers’ experiences. S3 will store this data most cost-effectively.
  - ex) A company plans to create a data lake that uses Amazon S3. The selection of S3 storage tiers will have the most effect on cost.
  <br/>
  - Bucket, Object
  - Tag
    - ex) A user is storing objects in Amazon S3. The user needs to restrict access to the objects to meet compliance obligations. The user should Tag the objects in the S3 bucket to meet this requirement.
  - Access Control
    - ACL
    - Bucket Policy
    - CORS(Cross-Origin Resource Sharing)
  - **`Versioning`**
    - ex) A company is storing sensitive customer data in an Amazon S3 bucket. The company wants to protect the data from accidental **deletion or overwriting.** The company should use Versioning to meet these requirements.
    - ex) A company is launching an application in the AWS Cloud. The application will use Amazon S3 storage. A large team of researchers will have shared access to the data. The company must be able to recover data that is accidentally **overwritten or deleted.**
  - Multipart Upload
  - **Transfer Acceleration**
    - ex) A company needs to quickly and securely move files over long distances between its client and an Amazon S3 bucket.
  - **Athena** : Serverless 대화형 쿼리 서비스
    - ex) A company has 5 TB of data stored in Amazon S3. The company plans to occasionally run queries on the data for analysis. The company should use **Athena** to run these queries in the most cost-effective manner.

■ **Glacier**
  - ex) A bank needs to store recordings of calls made to its contact center for 6 years. The recordings must be accessible within 48 hours from the time they are requested. **Glacier** will provide a secure and cost-effective solution for retaining these files.
  - Vault, Archive
  - Retrieval options
    - Expedited
    - Standard
    - Bulk

■ **Storage Class** + S3 Analytics + S3 Lifecycle  
  - **S3 Standard** : **저장 공간 무제한**, 최대 파일 크기 <= 5TB
    - The total amount of storage offered by Amazon S3 is Unlimited.
  - **`S3 Standrad Infrequent Access(Standard-IA)`** : AZ 저장 개수 3개 이상, 자주 접근되지는 않으나 접근시 빠른 접근
    - ex) A company wants to use Amazon S3 to store its legacy data. The data is rarely accessed. However, the data is critical and cannot be recreated. The data needs to be available for retrieval within seconds. Standard-IA meets these requirements MOST cost-effectively.
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
  - ex) A company should use **RDS, Elastic File System(EFS)** to read and write data that changes frequently.
  - ex) A company's IT team is managing **MySQL database server clusters.** The IT team has to patch the database and take backup snapshots of the data in the clusters.
  - ex) The company wants to move this workload to AWS so that these tasks will be completed automatically. The company should Use **Amazon RDS with a MySQL database to meet these requirements.**
  - ex) A company needs to deploy a PostgreSQL database into Amazon RDS. The database must be highly available and fault tolerant. The company should use **RDS with multiple Availability Zones** to meet these requirements.
  - RDS Proxy
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
  - ex) AWS perform Encrypt data that is stored in **Amazon DynamoDB Task automatically.**
  - ex) A user needs to quickly deploy a non-relational database on AWS. The user does not want to manage the underlying hardware or the database software. **DynamoDB** can be used to acoomplish this.
  - ex) A global company is building a simple time-tracking mobile app. The app needs to operate globally and must store collected data in a database. Data must be accessible from the AWS Region that is closest to the user. The company should use **DynamoDB global tables** to meet these data storage requirements with the least amount of operational overhead.

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
  - ex) A retail company needs to build a highly available architecture for a new ecommerce platform. The company is using only AWS services that replicate data across multiple Availability Zones. The company should use** Aurora and DynamoDB** to mteet this requirement.

## Module 5 (Networking - Part I)
■ **`VPC`**
  - Resources(구성요소) : Subnet, IGW, NAT, Route Table, DNS Host Name, CIDR
  - Multi-VPC
  - Multi-Account
  - The scope of a VPC within the AWS network is A VPC can span all Availability Zones within an AWS Region.
  - ex) A company requires an isolated environment within AWS for security purposes. Create a separate VPC to host the resources to accomplish this.

■ VPC Limits
  - dfault vpc 5 per Region(Case open can be increased)

■ CIDR

■ **Subnet**
  - Public
    - Internet Gateway : VPC -> S3, DynamoDB(Direct 접근 불가능)
  - Private
    - NAT Gateway(vs. NAT Instance) : Direct 접근 가능

■ **IGW(Internet Gateway)**
  - ex) The purpose of having an internet gateway within a VPC is **to allow communication between the VPC and the internet.**

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
      - ex) A company recently deployed an Amazon RDS instance in its VPC. The company needs to implement a stateful firewall to limit traffic to the private corporate network. The company should use to limit network traffic directly to its RDS instance.
      - ex) A company has multiple applications and is now building a new multi-tier application. The company will host the new application on Amazon EC2 instances. The company wants the network routing and traffic between the various applications to follow the security principle of least privilege. The company should use Security groups to enforce this principle.
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
  - ex) Allows a user to establish a dedicated network connection between a comopany's on-premises data center and the AWS Cloud.
  - ex) Can be used to create a private connection between an on-premises workload and an AWS Cloud workload
  - ex) A company is generating large sets of critical data in its on-premises data center. The company needs to securely transfer the data to AWS for processing. These transfers must occur daily over a dedicated connection. The company should use **Direct Connect** to meet these requirements.
  - ex) A company wants to migrate its applications from its on-premises data center to a VPC in the AWS Cloud. These applications will need to access on-premises resources. -> Create a VPN connection between an on-premises device and a virtual private gateway in the VPC & Set up an AWS **Direct Connect** connection between the on-premises data center and AWS.

■ **VPC Peering(VPC <-> VPC)** : **VPC 간 인터넷 없이 Private IP로 연결**
- ex) A company needs to establish a connection between two VPCs. The VPCs are located in two different AWS Regions. The company wants to use the existing infrastructure of the VPCs for this connection. -> VPC peering can be used to establish this connection.

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
  - ex) A large enterprise with multiple VPCs in several AWS Regions around the world needs to connect and centrally manage network connectivity between its VPCs.
  - ex) A network engineer needs to build a hybrid cloud architecture connecting on-premises networks to the AWS Cloud using AWS Direct Connect. The company has a few VPCs in a single AWS Region and expects to increase the number of VPCs to hundreds over time. **Transit Gateway** should the engineer use to simplify and scale this connectivity as the VPCs increase in number.
  - ex) A pharmaceutical company operates its infrastructure in a single AWS Region. The company has thousands of VPCs in a various AWS accounts that it wants to interconnect. The company should use **Transit Gateway** to help simplify management and reduce operational costs.

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
  - ex) A company is migrating its public website to AWS. The company wants to host the domain name for the website on AWS. The company should use to meet this requirement.
  
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
      - ex) A company is developing an application that uses multiple AWS services. The application needs to use temporary, limited-privilege credentials for authentication with other AWS APIs. The company should use STS to meet these authentication reqruiements.
    - Identity Broker
    - SAML
    - Amazon EC2 인스턴스에서 실행되는 애플리케이션이 다른 AWS 서비스에 액세스해야 하는 경우
    - 회사가 AWS에 요청하는 휴대폰에서 실행되는 애플리케이션을 생성하는 경우
    - ex) A company wants to grant users in one AWS account access to resources in another AWS account. The users do not currently have permission to access the resources.
    - ex) A user wants to allow applications running on an Amazon EC2 instance to make calls to other AWS services. The access granted must be secure.
    - ex) A security best practice for access to sensitive data that is stored in an Amazon S3 bucket. -> Use IAM roles for applications that require access to the S3 bucket.
    - ex) A company hosts an application on an Amazon EC2 instance. The EC2 instance needs to access several AWS resources, including Amazon S3 and Amazon DynamoDB. -> Create an IAM role with the required permissions and  Attach the role to the EC2 instance is the most operationally efficient solution to delegate permissions.
    - ex) A company's web application requires AWS credentials and authorizations to use an AWS service. The company should use IAM role as best practice.
  - Access
    - Least privilege access : Using IAM to grant access only to the resources needed to perform a task
    - Restricted access
    - As-needed access
    - Token access
  - ex) A developer has been hired by a large company and needs AWS credentials.
    - Grant the developer access to only the AWS resources needed to perform the job
    - Ensure the account password policy requires a minimum length

■ **IAM Access Analyzer**
  - IAM 역할 or S3 버킷이 외부 엔티티와 공유되었는지 식별
  - Identifies whether an Amazon S3 bucket or an IAM role has been shared with an external entity.
  - Analyzes resource policies in your AWS environment to help you identify and address unintended access.
  - Analyzes IAM policies to identify potential issues and excessive permissions.
  - Grant permissions to applications that run on Amazon EC2 instances.
  - Checks access policies and offers actionable recommendations to help users set secure and functional policies.
  - ex) A user wants to review all Amazon S3 buckets with ACLs and S3 bucket policies in the S3 console.
  - ex) A company wants to identify Amazon S3 buckets that are shared with another AWS account.

■ **`IAM Credential Report`**
  - Provides detailed information about the rotation history of user passwords and access keys within the account. It shows dates of last password and access key rotation, along with usernames and key IDs. This aligns perfectly with the requirement of auditing password and access key rotation details for compliance purposes.
  - The date and time when an IAM user's password was last used to sign in to the AWS Management Console
  - Whether multi-factor authentication (MFA) has been enabled for an IAM user
  - ex) A company has an AWS account. The company wants to audit its password and access key rotation details for compliance purposes.

■ **Cognito**(mobile/app User Pool Federated Identities)
  - 사용자 인증, 직접 로그인, 임시 권한
  - 웹 및 모바일 어플리케이션 사용자에 대한 자격 증명 제공
  - User Pool
  - ex) A company needs a central user portal so that users can log in to third-party business applications that support Security Assertion Markup Language (SAML) 2.0.
  - ex) A company needs to implement identity management for a fleet of mobile apps that are running in the AWS Cloud. Cognito will meeet this requirement.
  - ex) A company needs to set up user authentication for a new application. Users must be able to sign in directly with a user name and password, or through a third- party provider. The company should use Cognito to meet these requirements.

■ Landing Zone

■ Multi Accounts
  - Cross-account access
  - ex) A large organization has a single AWS account. The advantages of reconfiguring the single account into multiple AWS accounts are 
    - It allows for administrative isolation between different workloads.
    - Having multiple accounts reduces the risks associated with malicious activity targeted at a single account.
  - **Organizations**
    - Multiple AWS accounts
    - Root, OU(Organizational Units), Policy
    - Can be used at no additional cost
    - Share pre-purchased Amazon EC2 resources across accounts
    - Service control policies (SCPs) manage permissions
    - Helps to centrally manage billing and allow controlled access to resources across AWS accounts
    - A user is able to set up a master payer account to view consolidated billing reports through **Organizations**
    - ex) A global media company uses AWS Organizations to manage multiple AWS accounts. -> The company can use **Service control policies (SCPs)** to limit the access to AWS services for member accounts.
    - ex) A company that has multiple business units wants to centrally manage and govern its AWS Cloud environments. The company wants to automate the creation of
AWS accounts, apply service control policies (SCPs), and simplify billing processes.
    - ex) A large company has multiple departments. Each department has its own AWS account. Each department has purchased Amazon EC2 Reserved Instances. Some departments do not use all the Reserved Instances that they purchased, and other departments need more Reserved Instances than they purchased. The company needs to manage the AWS accounts for all the departments so that the departments can share the Reserved Instances. -> The company should Organizations use to meet these requirements.
    - **Consolidated Billing** : 통합 결제 추구
      - Benefits : Volume discounts, One bill for multiple accounts
      - Can be used to track charges across multiple accounts and report the combined cost
      - ex) A company has multiple AWS accounts that include compute workloads that cannot be interrupted. The company wants to obtain billing discounts that are based on the company's use of AWS services.
      - ex) Allows users to create new AWS accounts, group multiple accounts to organize workflows, and apply policies to groups of accounts
      - ex) A company wants to migrate its on-premises workloads to the AWS Cloud. The company wants to separate workloads for chargeback to different departments. Consolidated billing and Multiple AWS accounts will meet these requirements.
      - ex) A company has several departments. Each department has its own AWS accounts for its applications. The company wants all AWS costs on a single invoice to simplify payment, but the company wants to know the costs that each department is incurring. Consolidated billing will provide this functionality.

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
  - ex) A company wants to manage deployed IT services and govern its infrastructure as code (IaC) templates.
  - ex) A company wants to limit its employees' AWS access to a portfolio of predefined AWS resources.

■ **`Access keys`**
  - ex) A user needs programmatic access to AWS resources through the AWS CLI or the AWS API. Access keys will provide the user with the appropriate access.
  - ex) A systems administrator created a new IAM user for a developer and assigned the user an access key instead of a user name and password. -> To access the AWS account through a CLI

■ **Management Console**
  - ex) A company wants to manage its AWS Cloud resources through a web interface.
  - ex) A user has been granted permission to change their own IAM user password. CLI and Management Console can the user use to change the password.

■ **CLI**
  - ex) A company wants a unified tool to provide a consistent method to interact with AWS services.
  - ex) A user has been granted permission to change their own IAM user password. CLI and Management Console can the user use to change the password.

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
      - ex) A company needs to label its AWS resources so that the company can categorize and track costs.
      - ex) A company wants to review its monthly costs of using Amazon EC2 and Amazon RDS for the past year.
      - ex) A company can use to determine which business unit is using specific AWS resources.
      - ex) A company needs to graphically visualize AWS billing and usage over time. The company also needs information about its AWS monthly costs.
      - ex) A company use to visualize, understand, and manage AWS spending and usage over time.
      - ex) A company is moving multiple applications to a single AWS account. The company wants to monitor the AWS Cloud costs incurred by each application. The company can create cost allocation tags to mett this requirement.
    - Consolidated Billing
    - Helps users visualize, understand, and manage spending and usage over time
    - ex) A company uses Amazon EC2 instances to run its web application. The company uses On-Demand Instances and Spot Instances. The company needs to visualize its monthly spending on both types of instances. Cost Explorer will meet this requirement.
  - **`Budgets`** : 예산 설정
    - ex) A company wants to receive a notifiaction when a specific AWS cost threshold is reached.
    - ex) Can a company set up to send notifications that a custom spending threshold has been reached or exceeded
    - ex) A company wants to forecast future costs and usage of AWS resources based on past consumption. Cost Explorer will provide this forecast.
  - Detailed Billing Report : 상세 비용 보고서
  - Simple Monthly Caculator : AWS 계정 없어도 예상 비용 확인
  - **Pricing Caculator** : On-Premise를 AWS 실행할 때 예상 비용 확인
    - ex) A company plans to migrate to AWS and wants to create cost estimates for its AWS use cases.
    - ex) A company is exploring the use of the AWS Cloud, and needs to create a cost estimate for a project before the infrastructure is provisioned. AWS Pricing Calculator can be used to estimate costs before deployment.
    - ex) A company runs its workloads on premises. The company wants to forecast the cost of running a large application on AWS.
    - ex) A company is planning an infrastructure deployment to the AWS Cloud. Before the deployment, the company wants a cost estimate for running the infrastructure. AWS Pricing Calculator can provide this information.
    - ex) Gives users the ability to plan their service usage, service costs, and instance reservations, and also allows them to set custom alerts when their costs or usage exceed established thresholds
    - ex) Can send alerts to customers if custom spending thresholds are exceeded
  - **TCO Caculator** : On-Premise -> AWS Cloud 이전 비용 확인
    - AWS replaces upfront capital expenditures with pay-as-you-go costs
    - AWS uses economies of scale to continually reduce prices
  - **Cost Anomaly Detection** : 기계 학습으로 비용 및 사용량 모니터링
    - Uses machine learning to continuously monitor cost and usage for unusual cloud spending
  - **`Cost and Usage Reports`**
    - ex) A company needs to generate reports that can break down cloud costs by product, by company-defined tags, and by hour, day, and month. The company should use Cost and Usage Reports to meet these requirements.
    - <-> Cost Explorer
  - **Compute Optimizer** : 리소스의 구성 및 사용률 지표를 AWS 분석하여 크기 조정 권장 사항을 제공하는 서비스
  - **Personal Health Dashboard**

■ **Support Plan**
  - **Basic Support**
    - Personal Health Dashboard 제공
    - The free plan that provides access to documentation, forums, and basic support features.
  - **Developer Support**
    - Designed for developers running non-production workloads. It includes business hours access to Cloud Support Engineers and is suitable for development and testing environments
    - ex) A company is starting to build its infrastructure in the AWS Cloud. The company wants access to technical support during business hours. The company also wants general architectural guidance as teams build and test new applications. Developer Support will meet these requirements at the lowest cost. (if you are testing or doing early development on AWS and want the ability to get technical support during business hours as well as general architectural guidance as you build and test.)
  - **Business Support**
    - Trusted Advisor 제공
    - Includes 24/7 access to Cloud Support Engineers. It is suitable for businesses running production workloads
    - The least expensive AWS Support plan that contains a full set of AWS Trusted Advisor best practice checks
    - ex) A company wants to run production workloads on AWS. The company wants access to technical support from engineers 24 hours a day, 7 days a week. The company also wants access to the AWS Health API and contextual architectural guidance for business use cases. The company has a strong IT support team and does not need concierge support. Business Support will meet these requirements at the lowest cost.
  - **`Enterprise Support`**
    - Trusted Advisor 제공
    - Technical Account Manager(TAM), SME 지원
    - Designated support from an AWS technical account manager (TAM)
    - Support of third-party software integration to AWS
    - IEM(Infrastructure Event Management) : 인프라 이벤트 관리
      - ex) A company is preparing to launch a new web store that is expected to receive high traffic for an upcoming event. The web store runs only on AWS, and the company has an AWS Enterprise Support plan. -> Infrastructure Event Management will provide guidance about how the company should scale its architecture and operational support during the event.
    - AWS에 대한 타사 소프트웨어 통합 지원
    - 연중 무휴 응답
    - 컨설팅 아키텍처 및 운영 지침을 제공
    - Concierge Support Team 지원
      - AWS Billing 및 AWS Support 연락 창구
      - A primary point of **contact** for AWS Billing and AWS Support
    - Business Critical System Stop < 15minutes
    - The premium Support plan providing a wide range of benefits, including 24/7 access to Cloud Support Engineers, a Technical Account Manager(TAM), and more. It is suitable for enterprises running business-critical workloads
    - A designated technical account manager (TAM) to assist in monitoring and optimization
    - ex) A company wants to assess its operational readiness. It also wants to identify and mitigate any operational risks ahead of a new product launch. Enterprise Support plan offers guidance and support for this kind of event at no additional charge.
    - ex) A company wants to run production workloads on AWS. The company needs concierge service, a designated AWS technical account manager (TAM), and technical support that is available 24 hours a day, 7 days a week.

■ Free Tier
  - A company is using the AWS Free Tier for several AWS services for an application. If the Free Tier usage period expires or if the application use exceeds the Free Tier usage limits, The company will be charged the standard pay-as-you-go service rates for the usage that exceeds the Free Tier usage.

## Module 9 (Service, Security and Credentials)
■ **`Global Accelerator`** : 즉시 대응 서비스, 애플리케이션의 전반적인 가용성 및 성능 개선
  - Helps deliver highly available applications with fast failover for multi-Region and Multi-AZ architectures
  - Improves network performance by sending traffic through the AWS worldwide network infrastructure
  - ex) A company wants to improve the overall availability and performance of its applications that are hosted on AWS.

■ **`Config`**
  - 리소스 변경 사항 기록
  - 리소스 규정 준수 감사

■ **`Control Tower`** : 다중 계정 AWS 환경 자동화 및 관리

■ **Service Quotas** : 회사에서 서비스 한도 증가를 중앙에서 요청하고 추적하기 위해 사용해야 하는 서비스
  - ex) A company should use to centrally request and track service limit increases

■ **X-Ray** : Serverless, MSA 애플리케이션 분석 및 디버깅, 사용자 요청 추적
  - Provides the capability to view end-to-end performance metrics and troubleshoot distributed applications
  - ex) A company has a serverless application that includes an Amazon API Gateway API, an AWS Lambda function, and an Amazon DynamoDB database.

■ **`Trusted Advisor`**
  - **AWS 모범 사례**에 따라 정밀조사, 권장사항, 오픈 액세스 권한 확인, **보안 검사** 제공하는 포털
  - AWS 리소스의 서비스 할당량을 사전에 모니터링하고 계획하는 데 사용할 수 있는 기능을 제공
  - Best Practice : 비용 최적화, 성능, 보안, 내결함성, 서비스 제한
  - Detecting underutilized resources to save costs
  - Improving security by proactively monitoring the AWS environment
  - Provides recommendations for optimizing AWS resources for cost savings, performance, security, and fault tolerance
  - Can be used to proactively monitor and plan for the service quotas of AWS resources
  - ex) A company wants to monitor for misconfigured security groups that are allowing unrestricted access to specific ports
  - ex) A company needs to evaluate its AWS environment and provide best practice recommendations in five categories: cost, performance, service limits, fault tolerance and security.

■ **Inspector**
  - **인프라**의 자동 보안 평가 서비스
  - 보안 및 규정 준수 개선
  - Assess application vulnerabilities and must identify infrastructure deployments that do not meet best practices
  - A company wants an automated process to continuously scan its Amazon EC2 instances for software vulnerabilities
  - ex) A company needs an automated security assessment report that will identify unintended network access to Amazon EC2 instances. The report also must identify operating system vulnerabilities on those instances. The company should use Inspector to meet this requirement.

■ **`Artifact`**
  - 보안 및 규정 준수 보고서 제공 서비스
  - ISO 인증 문서 제공
    - Artifact should provide AWS ISO certifications
  - Allows users to download security and compliance reports about the AWS infrastructure on demand.
  - Provides on-demand access to AWS compliance reports and documents.
  - Primarily used for managing and tracking infrastructure as code(IaC) configurations, not directly related to credential auditing.
  - The best resource for a user to find compliance-related information and reports about AWS
  - ex) A cloud practitioner needs to obtain AWS compliance reports before migrating an environment to the AWS Cloud.

■ **Audit Manager**
  - Helps you continuously audit your AWS usage to simplify how you assess risk and compliance with regulations and incustry standards.
  - Offers a comprehensive platform for managing and automating audits across various AWS services, but it requires additional setup and configuration compared to the readily available IAM credential report.

■ **Compliance Program**
  - ex) A company needs to build an application that uses AWS services. The application will be delivered to residents in European Counties. The company must abide by regional regulatory requirements.

■ **Knowledge Center**
  - Provides answers to the most frequently asked security-related questions that AWS receives from its users.

■ **`GuardDuty`**
  - 지능형 위협 탐지 서비스
  - 머신러닝 알고리즘을 통해 이상 탐지 수행
  - A threat detection service that continuously monitors for malicious activity and unauthorized behavior in AWS accounts
  - Monitors AWS accounts for security threats
  - Provides threat detection by monitoring for malicious activities and unauthorized actions to protect AWS accounts, workloads, and data that is stored in Amazon S3
  - Automatic monitoring for threats to AWS workloads
  - ex) A company wants to implement threat detection on its AWS infrastructure. However, the company does not want to deploy additional software.

■ Penetration Testing
  - 자체 인프라 공격해 보안 테스트

■ **Macie**
  - 완전 관리형 데이터 개인 정보 보호 서비스
  - S3 버킷 내 개인 식별 정보 찾음 및 경고
  - Gives users the ability to discover and protect **sensitive data** that is stored in Amazon S3 buckets
  - Automatically recognizes and classifies **sensitive data** or intellectual property on AWS
  - Uses machine learning to help discover, monitor, and protect **sensitive data** that is stored in Amazon S3 buckets
  - ex) A company needs to identify personally identifiable information (PII), such as credit card numbers, from data that is stored in Amazon S3. The company should use Macie to meet this requirement.

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
  - ex) A company wants to estabilish a schedule for rotating database user credentials. Secrets Manager will support this requirement with the LEAST amount of operational overhead.
  - ex) A user wants to securely automate the management and rotation of credentials that are shared between applications, while spending the least amount of time on managing tasks. Secrets Manager can be used to accomplish this.

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

■ Best Practice
  - ex) Amazon EC2 instance be given access to an Amazon S3 bucket -> Have the EC2 instance assume a role to obtain the privileges to upload the file
  - ex) A company is setting up AWS Identity and Access Management(IAM) on an AWS account. IAM security Best Practice -> Turn on multi-factor authentication(MFA) for added security during the login process.

## Module 10 (Monitoring and Automation)
■ **`Auto Scaling`**
  - Scheduled/Dynamic/Predictive
  - Auto Scaling Group
    - Min/Max/Desired
    - ex) A company deployed an application on an Amazon EC2 instance. The application ran as expected for 6 months in the past week, users have reported latency issues. A system administrator found that the CPU utilization was at 100% during business hours. The company wants a scalable solution to meet demand. The company should use to handle the load for its application during periods of high demand.
  - Benefits
    - Improved health and availability of applications
    - Optimized performance and costs
  - ex) A company wants to automatically add and remove Amazon EC2 instances. The company wants the EC2 instances to adjust to varying workloads dynamically. EC2 Auto Scaling will meet these requirements.

■ **`CloudWatch`** : Metrics, Logs, Alarm, Event, Rule, Target
  - ex) A company wants to receive a notifiaction when a specific AWS cost threshold is reached.
  - ex) Monitors CPU utilization on Amazon EC2 instances
  - ex) Used to monitor Amazon EC2 instances for CPU and network utilization
  - ex) An ecommerce company wants to use Amazon EC2 Auto Scaling to add and remove EC2 instances based on CPU utilization. CloudWatch alarm can initiate an EC2 Auto Scaling action to achieve this goal.

■ **`CloudTrail`** : 계정 활동 추척, 이벤트 및 API 호출 기록 X-Ray 등
  - Enables customers to audit API calls in their AWS accounts
  - Can identify when an Amazon EC2 instance was terminated
  - Tracks API calls and user activity
  - ex) A company needs to track the activity in its AWS accounts, and needs to know when an API call is made against its AWS resources.
  - ex) A company needs to identify the last time that a specific user accessed the AWS Management Console. CloudTrail will provide this information.
  - ex) Helps users audit API activity across their AWS account
  - ex) A user needs to determine whether an Amazon EC2 instance's security groups were modified in the last month. -> The user can use AWS CloudTrail to see if the security group was changed to see if a change was made.
- VPC Flow Logs : 수신 및 발신 트래픽 정보
  - Provides log information of the inbound and outbound traffic on network interfaces in a VPC
- Service Health Dashboard
  - Used to capture information about inbound and outbound traffic in an Amazon VPC

■ **`CloudFormation`**
  - **Infrastructure as Code(IaC)**
  - Change set
  - ex) A developer needs to maintain a development environment infrastructure and a production environment infrastructure in a repeatable fashion. CloudFormation should the developer use to meet these requirements.
  - ex) A company wants to automate infrastructure deployment by using infrastructure as code (IaC). The company wants to scale production stacks so the stacks can be deployed in multiple AWS Regions.
  - ex) A user wants to deploy a service to the AWS Cloud by using infrastructure-as-code (IaC) principles. CloudFormation can be used to meet this requirement.

■ Quick Start

■ OpsWorks

## Module 11 (Caching)
■ **CloudFront** : CDN
  - Enables companies to deploy an application close to end users
  - ex) A company is building an application that needs to deliver images and videos globally with minimal latency. Deliver the content through Amazon CloudFront can the company use to accomplish this in a cost effective manner.
  - ex) A company is planning to run a global marketing application in the AWS Cloud. The application will feature videos that can be viewed by users. The company must ensure that all users can view these videos with low latency.
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
  - ex) A company has a set of ecommerce applications. The applications need to be able to send messages to each other.
  - ex) Can a company use to achieve a loosely coupled architecture
  - ex) A company needs to simultaneously process hundreds of requests from different users. The company should use the combination of SQS and Lambda to build an operationally efficient solution.
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
      - ex) A company wants its Amazon EC2 instances to operate in a highly available environment, even if there is a natural disaster in a particular geographic area. -> Use EC2 instances in multiple AWS Regions
    - Direct Connect(Backup Replication)

■ Best Practice
  - ex) A company is running a critical workload on an Amazon RDS DB instance. The company needs the DB instance to be highly available with a recovery time of less than 5 minutes. Modify the DB instance to be a Multi-AZ deployment will meet these requirements.
  - ex) A company needs to design an AWS disaster recovery plan to cover multiple geographic areas. Configure the architecture across multiple AWS Regions will meet this requirement.
  - Backup and restore is the least expensive disaster recovery option.

## Module 15 (Migration)
■ Application Discovery Service

■ **`Application Migration Service`**

■ Data Migration Service(DMS)
  - used to migrate a company's on-premises MySQL database to Amazon RDS
  - **Snowball**
    - PetaByte(60TB)
    - The transfer of data from the Snowball Edge appliance into Amazon S3 at **No Cost**
    - ex) A company is migrating to Amazon S3. The company needs to transfer 60 TB of data from an on-premises data center to AWS within 10 days.
  - **`Snowball Edge`** : 100TB(80TB)
    - 인터넷 연결이 간헐적이거나 없는 위치에서 데이터 수집
    - ex) A company has a fleet of cargo ships. The cargo ships have sensors that collect data at sea, where there is intermittent or no internet connectivity. The company needs to collect, format, and process the data at sea and move the data to AWS later. The company should use Snowball Edge to meet these reqruiements. 
  - **Snowmobile** : ExaByte
    - Designed for large-scale data transfers
    - ex) A company is moving an on-premises data center to the AWS Cloud. The company must migrate 50 petabytes of file storage data to AWS with the least possible operational overhead.
  - Schema Conversion Toll (SCT)

■ Server Migration Service

■ **`Storage Gateway`**
  - On-Premise Data Storage -> AWS Cloud 연결
  - **A hybrid cloud storage service** that provides on-premises users access to virtually unlimited cloud storage
  - ex) A company has a centralized group of users with large file storage requirements that have exceeded the space available on premises. The company wants to extend its file storage capabilities for this group while retaining the performance benefit of sharing content locally. -> Configure and deploy an AWS Storage Gateway file gateway. Connect each user's workstation to the file gateway.
  - ex) A company is using a third-party service to back up 10 TB of data to a tape library. The on-premises backup server is running out of space. The company wants to use AWS services for the backups without changing its existing backup workflows.
  - ex) A company has a physical tape library to store data backups. The tape library is running out of space. The company needs to extend the tape library's capacity to the AWS Cloud. The company should use AWS Storage Gateway to meet this requirement.
  - 종류
    - Tape Gateway
    - Volumne Gateway
    - FSx File Gateway
    - **S3 File Gateway**
      - ex) A company wants to migrate its NFS on-premises workload to AWS.
      - ex) A company wants to store and retrieve files in Amazon S3 for its existing on-premises applications by using industry-standard file system protocols.

■ **Consulting Partner** : Migration 전문가
  - ex) A company wants to migrate its workloads to AWS, but it lacks expertise in AWS Cloud computing. Consulting Partners will help the company with its migration.

■ **Professional Services**
  - ex) A global company wants to migrate its third-party applications to the AWS Cloud. The company wants help from a global team of experts to complete the migration faster and more reliably in accordance with AWS internal best practices.

■ **Outposts**
  - AWS Cloud Service -> On-Premise 삽입 및 하이브리드 아키텍처 확장
  - 낮은 대기 시간 또는 로컬 시스템 상호 종속성이 필요한 애플리케이션에 유용
  - Supports a hybrid architecture that gives users the ability to extend AWS infrastructure, AWS services, APIs, and tools to data centers, co-location environments, or on-premises facilities
  - ex) A manufacturing company has a critical application that runs at a remote site that has a slow internet connection. The company wants to migrate the workload to AWS. The application is sensitive to latency and interruptions in connectivity. The company wants a solution that can host this application with minimum latency.
  - ex) A company is operating several factories where it builds products. The company needs the ability to process data, store data, and run applications with local system interdependencies that require low latency. The company should use Outposts to meet these requirements.
  - ex) A company wants to deploy some of its resources in the AWS Cloud. To meet regulatory requirements, the data must remain local and on premises. There must be low latency between AWS and the company resources.

■ **Local Zones**
  - 대기 시간 최소화, 연결 중단 X
  - A type of AWS infrastructure deployment that places compute, storage, database, and other services closer to a specific geographic area
  - Designed to run workloads that require single-digit millisecond latency, like video rendering and graphics intensive, virtual desktop applications

■ **Wavelength**
  - Designed specifically to reduce the latency between devices and applications hosted on AWS by placing AWS compute and storage services at the edge of the 5G network

■ Best Practice
  - ex) An ecommerce company has migrated its IT infrastructure from an on-premises data center to the AWS Cloud. Cost of application software licenses is the company's direct responsibility.
  - ex) A company wants to modernize and convert a monolithic application into microservices. The company wants to move the application to AWS. -> Refactor

## Module 16 (Developer Tools)
■ CodeCommit : 버전 관리
  - ex) A company wants to host a private version control system for its application code in the AWS Cloud. The company should use to meet this requirement.

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
  - ex) A company wants its employees to have access to virtual desktop infrastructure to securely access company-provided desktops through the employees' personal devices.

■ **Chime** : 오디오와 비디오를 송수신하고 콘텐츠 공유를 허용하는 실시간 미디어 애플리케이션 구축

■ **`AppStream 2.0`** : 원격 작업 액세스를 지원하는 완전관리형의 비영구 데스크톱 및 애플리케이션 서비스
  - Provide managed Windows virtual desktops and applications to its **remote employees over secure network connections**
  - Focused on hosting individual applications on AWS
  - ex) A company has an application with robust hardware requirements. The application must be accessed by students who are using lightweight, low-cost laptops. AppStream 2.0 will help the company deploy the application without investing in backend infrastructure or high-end client hardware
  - ex) A company wants to use the AWS Cloud to provide secure access to desktop applications that are running in a fully managed environment.

■ Marketplace
  - ex) A company wants to migrate to AWS and use the same security software it uses on premises. The security software vendor offers its security software as a service on AWS. The company can purchase the security solution.

## Module 18 (Application)
■ API Gateway : 규모와 관계없이 REST 및 WebSocket API를 생성, 게시, 유지, 모니터링 및 보호하기 위한 서비스
  - A development team wants to publish and manage web services that provide REST APIs.

■ SWF

■ Elastic Transcoder
  - ex) A company wants to convert video files and audio files from their source format into a format that will play on smartphones, tablets, and web browsers. Elastic Transcoder will meet these requirements.

■ Step Functions
  - ex) Can a company use to achieve a loosely coupled architecture

■ Cloud Search

## Module 19 (Analytics)
■ EMR : Elastic MapReduce, Hadoop Framework

■ Kinesis : Realtime Video & Data Stream Service 수집, 처리, 분석

■ Elastic Search

■ **QuickSight** : Serverless, 시각적 보고서 생성, BI 서비스
  - ex) A company is using a centrla data platform to manage multiple types of data for its customers. The compoany wants to use AWS services to discover, transform, and visualize the data.
  - ex) Gives users the ability to build interactive business intelligence dashboards that include machine learning insights
  - ex) Supports the creation of visual reports from AWS Cost and Usage Report data

■ Data Pipeline : 서로 다른 사용자의 수백 건 요청 동시 처리

■ **`RedShift`**
  - Data Warehouse
  - Column 형식
  - OLAP(Online Analytical Processing) 용도
  - ex) A company wants to operate a data warehouse to analyze data without managing the data warehouse infrastructure.
  - ex) A company needs to generate reports for business intelligence and operational analytics on petabytes of semistructured and structured data. These reports are produced from standard SQL queries on data that is in an Amazon S3 data lake. Redshift provides the ability to analyze this data.
  - ex) A company needs to set up a petabyte-scale data warehouse in the AWS Cloud.

■ Glue : Serverless 관리형 ETL(Extract, Transform, Load)
  - Is used specifically for extract, transform, and load (ETL) data
  - ex) A company is using a centrla data platform to manage multiple types of data for its customers. The compoany wants to use AWS services to discover, transform, and visualize the data.

■ Managed Blcokchain

## Module 20 (AI)
■ Machine Learning

■ **Rekognition** : Vision
  - Use to organize, characterize, and search large numbers of images
  - ex) A company has a social media platform in which users upload and share photos with other users. The company wants to identify and remove inappropriate photos. The company has no machine learning (ML) scientists and must build this detection capability with no ML expertise. The company should use Rekognition to build this capability.
  - ex) A company wants to add facial identification to its user verification process on an application. The company should use Rekognition to meet this requirement.

■ **`Polly`** : TTS
  - Used to turn text into lifelike speech

■ **`Lex`** : ChatBot, 자동 음성 인식, 자연어 이해 기능

■ Transcribe : STT, 자동 음성 인식, ASR 딥러닝 처리 사용

■ Translate : 번역

■ Comprehend : NLP, 이메일 메세지 감정분석 등 사용
  - ex) A company wants to perform sentiment analysis on customer service email messages that it receives. The company wants to identify whether the customer service engagement was positive or negative. The company should use comprehend to perfom this analysis.

■ SageMaker : 머신 러닝 모델 구축, 훈련, 배포, 기계 학습

■ ForeCast

■ Code Guru

■ Kendra : 문서 검색 서비스

■ Personalize : 개인화 추천
  - ex) A company wants an AWS service to provide product recommendations based on its customer data.

■ Textract : 텍스트 추출
  - ex) A company accepts enrollment applications on handwritten paper forms. The company uses a manual process to enter the form data into its backend systems. The company wants to automate the process by scanning the forms and capturing the enrollment data from scanned PDF files. The company should use Textract to build this process.

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

## 참고
- Serverless 서버리스 서비스 : Lambda, DynamoDB, Kinesis, Fargate, S3
- 내장 방화벽이 갖춰진 서비스 : WAF, CloudFront, ELB, Route53, VPC/보안그룹
- 고가용성 서비스 : Route53, CloudWatch, ELB, EIP, AutoScale
- PaaS : BeanStalk, CloudFormation, OpsWorks
- AWS랑 하이브리드 환경이랑 같이 사용할 수 있는 서비스

## 문제
- 65문항 100분
- 25.2.15(토) **`출제내용`**로 표시함
<br/>
- <https://kananinirav.com/practice-exam/exams.html>
- <https://www.examtopics.com/exams/amazon/aws-certified-cloud-practitioner/view/>
  - ~24 까지 무료
- <https://www.examtopics.com/exams/amazon/aws-certified-cloud-practitioner-clf-c02/view/>
  - ~17 까지 무료
- <https://www.exam-answer.com/aws-service-infrastructure-security-optimization-recommendations>

## 참고
- <https://docs.aws.amazon.com/>
- <https://tbvjrornfl.tistory.com/188>
- <https://jeongchul.tistory.com/681>
- <https://aws.amazon.com/ko/training/classroom/aws-cloud-practitioner-essentials/>
- <https://aws.amazon.com/ko/whitepapers/>
- <https://drive.google.com/file/d/1RyD96KoTByLR0UzJIAe2CvFQqtvE20k3/view?pli=1>
- <https://m.blog.naver.com/PostView.naver?blogId=thdud9943&logNo=222892401058&proxyReferer=https:%2F%2Fm.blog.naver.com%2FPostView.naver%3FblogId%3Dthdud9943%26logNo%3D222892236969%26proxyReferer%3D%26noTrackingCode%3Dtrue&trackingCode=blog_postview>
- <https://maplambda.tistory.com/m/7>
- <https://hahahohot.tistory.com/m/10>
- <https://jiny-dev.tistory.com/category/Study/AWS>
- <https://hyunsangwon93.tistory.com/m/31>
- <https://maplambda.tistory.com/m/23>
- <https://jiny-dev.tistory.com/117>