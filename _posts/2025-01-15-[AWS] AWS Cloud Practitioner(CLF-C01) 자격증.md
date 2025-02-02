---
title: '[AWS] AWS Cloud Practitioner(CLF-C01) 자격증'
date: 2025-01-15 18:10:00 +0900
categories: [인프라/클라우드, AWS]
tags: [AWS]
math: true
mermaid: true
---

## Module 1 (Introduction)
■ Cloud Computing Deploy Model
  - Cloud
    - No Infrastructure Operating Costs
  - On-Premise(Private Cloud)
  - Hybrid

■ AWS Global Infrastructure
  - Data Center
  - Availability Zone(AZ)
    - 논리적 개념
    - 1개 이상의 개별 데이터 센터(보통 3개 ~ 6개)
  - Region
    - 물리적 위치
    - Compliance, LAtency, AZ, Costs 고려
    - Region 종속 Service : EC2, Elastic Beanstalk, Lambda, Rekognition
    - Global Service : IAM, Route53, CloudFront, WAF
  - **Edge Location** : 서비스별 작업 수행하는 데이터 센터 ex) CloudFront, Global Accelerator

■ Six advantages of Cloud Computing
  - Trade capital expense (CAPEX) for operational expense(OPEX) : 선행 비용(고정 비용)을 가변 비용으로 대체
  - Benefit from massive economies of scale : 거대한 규모의 경제로 얻는 이점
  - Stop guessing capacity : 용량 추정 불필요
  - Increase speed and agility : 속도 및 민첩성 개선
    - ex) Migration
  - Stop spending money running and maintaining data centers : 데이터 센터 운영 및 유지 관리에 비용 투자 불필요
  - Ability to Deploy globally in minutes : 몇 분 안에 전 세계에 배포
    - ex) Migration

■ 클라우드로 해결한 문제점
  - Flexibility
  - Cost-Effectiveness
  - Scalability
  - Elasticity
  - High-availability and fault-tolerance
  - Agility 

■ **The Well Architected Framework(WAF)**
  - 운영 우수성 : 프로세스 개선, Anticipate failure
    - ex) 코드로 작업 수행, 문서에 주석 추가, 실패 예측, 되돌릴 수 있는 소규모 변경 자주 수행, 배포 파이프 라인을 사용한 변경 자동화, 트리거된 이벤트에 대한 응답, 회사는 구성 요소를 정기적으로 업데이트하고 작고 되돌릴 수 있는 증분으로 변경할 수 있도록 AWS 워크로드를 설계
  - 보안 : 암호화, 데이터 무결성 검사
    - ex) AWS Config를 사용하여 추적 가능성 활성화
  - 안정성/신뢰성(Reliability) : 기능 수행, 복구
    - ex) 비정상 애플리케이션을 자동으로 복구, 가용성(High Availability) 유지, 가동 중지 시간 최소화, 장애로부터 신속하게 복구
  - 성능 효율성 : Serverless, 고급 기술
    - ex) 워크 로드 및 메모리 요구 사항에 따라 적합한 Amazon EC2 유형을 사용해 정보에 기반한 의사 결정을 수립하고 비즈니스 요구가 진화함에 따라 효율성 유지
  - 비용 최적화
    - ex) EC2 서버 비용 효과적인 크기 선택
  - 지속 가능성

■ **Shared Responsibility Model**
  - AWS Responsibility
    - Physical and Environmental Controls
    - Update the Firmware of the Network Infrastructure
  - Customer Responsibility
    - Manage connections to the database

■ **Cloud Adoption Framework(CAF)**
  - Migration 조언
  - 비즈니스, 인력, 거버넌스, 플랫폼 관점, 보안, 운영 관점

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
  - Snowcone, Snowball, Snowmobile
  - Athena : Serverless 대화형 쿼리 서비스
  - Data Encryption 제공

■ Glacier  
  - Vault, Archive
  - Retrieval options
    - Expedited
    - Standard
    - Bulk

■ Storage Class + S3 Analytics + S3 Lifecycle  
  - S3 Standard : 저장 공간 무제한, 최대 파일 크기 <= 5TB
  - S3 Standrad IA : AZ 저장 개수 3개 이상
  - S3 One Zone IA : AZ 저장 개수 1개
  - S3 Intelligent Tiering
  - Glacier / Deep Archive : Archiving, 장기백업, 저렴

■ Choosing a Region

■ IOPS : 초당 입출력 작업 수

## Module 3 (Adding a Compute Layer)
■ **EC2**
  - AMI
  - User Data
  - Instance Metadata
  - Storing Data
    - EBS(Elastic Block Storage)
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
  - Instance Size & Type
    - M5, C5, R5, T2 & T3, G3 & P3, I2, D2
  - **Pricing**
    - On Demand
    - Reserved Instance(RI) : 결제 할인 옵션, 일정 사용량 악정 X
    - Spot Instance : 공급/수요에 따라 조정
    - Dedicated Instance
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

■ **LightSail**
  - 가상 프라이빗 서버(Virtual Private Server)

■ Elastic BeanStalk
  - 간단하게 서버 배포
  - 프로비저닝 필요 없이 빠르게 애플리케이션을 AWS에 업로드
  - 용량 조정(프로비저닝), 로드 밸런싱, Auto Scaling, 애플리케이션 상태 모니터링

■ **Lambda** : Serverless(코드가 서버에서 실행되지만, 프로비저닝하거나 관리 필요 없음)
  - 비용 청구 기준 : 실행시간, 요청 수
  - 회사의 책임 : 코드 내부의 보안, 코드 작성 및 업데이트
■ **Fargate** : Conainer(ECS, EKS)를 위한 Serverless 컴퓨팅 엔진

■ Batch

## Module 4 (Adding a Database Layer)
■ RDS(Relational Database Service)
  - 관리형 데이터베이스 서비스
  - DB 이중화 : Multi AZ 사용
  - OLTP(Online Transactional Processing) 용도
  - Amazon Aurora : MySQL, PostgreSQL과 호환되는 관계형 DB 엔진
  - MSSQL
  - Maria DB
  - Oracle

■ DynamoDB
  - 관리형 데이터베이스 서비스
  - NoSQL DB
  - 초저지연시간
  - Row 형식
  - Global Table
  - Consistency Option
    - Eventually
    - Strongly

■ DAX(DynamoDB Accelerator) : DynamoDB 전용 Cache

■ DocumentDB
  - MongoDB 호환 DB

■ ElastiCache
  - In-Memory Cache

■ Neptune
  - Graph DB

■ QLDB(Quantum Ledger Database) : Serverless 완전 관리형 DB

## Module 5 (Networking - Part I)
■ VPC
  - 구성요소 : Subnet, IGW, NAT, Route Table, DNS Host Name, CIDR
  - Multi-VPC
  - Multi-Account

■ VPC Limits
  - dfault vpc 5 per Region(Case open can be increased)

■ CIDR

■ Subnet
  - Public
    - Internet Gateway : VPC -> S3, DynamoDB(Direct 접근 불가능)
  - Private
    - NAT Gateway(vs. NAT Instance) : Direct 접근 가능

■ IGW(Internet Gateway)

■ Routing Table

■ ENI(Elastic Network Interface)

■ EIP(Elastic IP)

■ Security
  - **Security Group**
    - 가상 Stateful 방화벽, 허용 규칙만 포함, 인스턴스(EC2, RDS) 수준
    - Security Group Chaining
  - **Network ACL(NACL)**
    - Stateless, IP주소만 포함, Subnet 수준
    - 트래픽 허용 여부를 결정할 때 가장 낮은 번호의 규칙부터 순서대로 규칙을 처리
    - ex) S3 객체 액세스 제한

## Module 6 (Networking - Part II)
■ CGW(Customer Gateway)
  - On-Premise, 외부 네트워크

■ VGW(Virtual Private Gateway)
  - **VPN**(Virtual Private Network)
    - **Site-to-Site VPN** : IPSec 통신, AWS VPC(**Virtual Private Gateway**) -> On-Premise(**Customer Gateway**) 네트워크 연결
  - **Direct Connect**(DX) : On-Premise -> AWS Cloud 네트워크 연결

■ **Transit Gateway** : 여러 개 VPC 간 연결 -> On-Premise 네트워크 연결, Hub and Spoke 형태, 중앙 관리

■ Managed VPN

■ VPC Peering(VPC <-> VPC) : 인터넷 없이 Private IP로 연결

■ VPC Endpoints
  - 인터넷 없이 EC2 인스턴스 -> S3 버킷, DynamoDB용 VPC 엔드포인트 액세스
  - Interface
  - Gateway
  - Gateway Load Banlancer

■ PrivateLink(Endpoint Service) : VPC와 AWS 서비스 간에 프라이빗 연결

■ ELB
  - Options
    - Application(ALB) L7
    - Network(NLB) L4
    - Classic(CLB) L4, L7

■ High Availability(고가용성)
  - 리소스에 장애가 발생하더라도 애플리케이션에 계속 액세스 보장
  - ELB
  - Route53
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
    - DDoS 방지

## Module 7 (IAM)
■ Account Root User
  - Root User Privileges

■ **IAM**
  - User / Federated User(AD, LDAP)
    - 회사가 개인의 AWS 접근 자격 증명 생성하는 경우
    - 회사에서 IAM 그룹에 사용자를 추가해야 하는 경우
  - Group
  - Policy
    - Resource-Based / Identity-Based
    - Manage / Inline
  - Role
    - STS(Security Token Service) : 제한된 권한의 임시 자격 증명 생성
    - Identity Broker
    - SAML
    - Amazon EC2 인스턴스에서 실행되는 애플리케이션이 다른 AWS 서비스에 액세스해야 하는 경우
    - 회사가 AWS에 요청하는 휴대폰에서 실행되는 애플리케이션을 생성하는 경우

■ Cognito(mobile/app User Pool Federated Identities)
  - 사용자 인증, 직접 로그인, 임시 권한
  - 웹 및 모바일 어플리케이션 사용자에 대한 자격 증명 제공
  - User Pool

■ **IAM Access Analyzer**
  - IAM 역할 or S3 버킷이 외부 엔티티와 공유되었는지 식별

■ Landing Zone

■ Multi Accounts
  - Cross-account access
  - **Organizations**
    - Consolidated Billing : 통합 결제 추구
    - Root, OU(Organizational Units), Policy

■ Systems Manager
  - 애플리케이션 및 리소스를 위한 운영 허브
  - 안전하고 보안이 보장된 대규모 운영을 지원하는 하이브리드 클라우드 환경을 위한 안전한 End-to-End 관리 솔루션
  - Run Command
  - Maintenance Windows
  - Patch Management
  - State Manager
  - **Session Manager**
  - Inventory

■ Service Catalog
  - 액세스 제한

■ Management Console

■ CLI

■ SDK : 프로그래밍 방식

■ Managed Microsoft AD : MFA

■ AD Connector

■ Single Sign On(SSO) : 사용자가 회사 네트워크에서 인증을 받고 두 번째 로그인 없이 AWS를 사용할 수 있기를 원하는 경우

## Module 8 (Service, Security and Credentials)
■ **Global Accelerator** : 즉시 대응 서비스, 애플리케이션의 전반적인 가용성 및 성능 개선

■ **Control Tower** : 다중 계정 AWS 환경 자동화 및 관리

■ **Service Quota** : 회사에서 서비스 한도 증가를 중앙에서 요청하고 추적하기 위해 사용해야 하는 서비스

■ **X-Ray** : Serverless, MSA 애플리케이션 분석 및 디버깅, 사용자 요청 추적

■ **Config**
  - 리소스 변경 사항 기록
  - 리소스 규정 준수 감사

■ **Trusted Advisor**
  - AWS 모범 사례에 따라 정밀조사, 권장사항, 오픈 액세스 권한 확인, 보안 검사 제공하는 포털
  - AWS 리소스의 서비스 할당량을 사전에 모니터링하고 계획하는 데 사용할 수 있는 기능을 제공
  - Best Practice : 비용 최적화, 성능, 보안, 내결함성, 서비스 제한

■ **Inspector**
  - 인프라의 자동 보안 평가 서비스
  - 보안 및 규정 준수 개선

■ **Artifact**
  - 보안 및 규정 준수 보고서 제공 서비스
  - ISO 인증 문서 제공

■ **Guard Duty**
  - 지능형 위협 탐지 서비스
  - 머신러닝 알고리즘을 통해 이상 탐지 수행

■ Penetration Testing
  - 자체 인프라 공격해 보안 테스트

■ **Macie**
  - 완전 관리형 데이터 개인 정보 보호 서비스
  - S3 버킷 내 개인 식별 정보 찾음 및 경고

■ Certificate Manager(ACM)
  - SSL/TLS 인증서 쉽게 생성 및 관리 지원 서비스

■ **CloudHSM**
  - Hardware Security Module
  - 사용자가 직접 암호화 키 관리

■ Directory Service

■ **KMS**
  - Encrypt EBS 제공
  - 암호와 키에 관한 액세스 권한 -> 데이터 액세스 권한
  - CMK(Customer Master Key) 생성 및 제어

■ **Secret Manager**
  - 프로그래밍 방식으로 암호 변경, 보안 정보 보호 및 관리
  - RDS에서 관리

■ **Shield**
  - DDoS 방지
  - Standard, Advanced

■ **WAF**
  - SQL Injection, XSS 방지, L7 필터링

■ Security Hub
  - 중앙 집중식 보안 위치

■ Detective
  - 보안 문제 및 의심스러운 활동 조사

■ Abuse
  - 부정 사용 및 불법적 목적 사용 감지

## Module 9 (Elasticity, HA and Monitoring)
■ Elasticity
  - Time-Bases, Volume-Based, Predictive-Based

■ Monitoring
  - **CloudWatch** : Metrics, Logs, Alarm, Event, Rule, Target
  - **CloudTrail** : 계정 활동 추척, 이벤트 및 API 호출 기록 x-ray등
  - VPC Flow Logs : 수신 및 발신 트래픽 정보
  - Service Health Dashboard
  - **Personal Health Dashboard** : 모든 플랜 제공

■ Costs
  - **Cost Explorer** : 과거 데이터로 비용 탐색기
    - Cost Allocation Tags
    - Consolidated Billing
  - Budgets : 예산 설정
  - Detailed Billing Report : 상세 비용 보고서
  - Simple Monthly Caculator : AWS 계정 없어도 예상 비용 확인
  - **Pricing Caculator** : On-Premise를 AWS 실행할 때 예상 비용 확인
  - **TCO Caculator** : On-Premise -> AWS Cloud 이전 비용 확인
  - **Cost Anomaly Detection** : 기계 학습으로 비용 및 사용량 모니터링
  - **Cost Allocation Tags** : 부서별 리소스 사용 비용

■ **Support Plan**
  - Basic
  - Developer
  - Business
    - Trusted Advisor 제공
  - Enterprise
    - Trusted Advisor 제공
    - TAM, SME 지원
    - IEM(Infra Event Management) : 인프라 이벤트 관리
    - AWS에 대한 타사 소프트웨어 통합 지원
    - 연중 무휴 응답
    - Concierge Support Team 지원
      - AWS Billing 및 AWS Support 연락 창구
    - Businnes Critical System Stop < 15minutes

■ **Auto Scaling**
  - Scheduled/Dynamic/Predictive
  - Auto Scaling Group
    - Min/Max/Desired
  - DDoS 방지

■ Scaling Database
  - RDS
    - Read Replica
    - Sharding
  - DynamoDB
    - Auto Scaling/On Demand
    - Adaptive Capacity

## Module 10 (Automation)
■ **CloudFormation**
  - Infrastructure as Code(IaC)
  - Change set

■ Quick Start

■ OpsWorks

## Module 11 (Caching)
■ **CloudFront** : CDN, DDoS 방어
  - Expire contents
    - TTL
    - Change object name
    - Invalidate Object

## Module 12 (Decoupled Architecture)
■ SQS(Simple Queue Service)
  - Serverless, 완전 관리형, 분리(Decouple), 송신 비용만 존재
  - Type
    - Standard vs FIFO
  - Features
    - Dead letter queue
    - Visibility timeout
    - Long polling(vs Short polling)

■ SNS(Simple Notification Service)
  - 완전 관리형 Pub/Sub 메세징
  - Lambda
  - SQS
  - HTTP / HTTPS
  - Email
  - SMS

■ SES(Simple Email Service)

■ MQ : 온프레미스에서 실행되는 개방형 프로토콜

## Module 13 (Microservices and Serverless)
■ Microservices
  - Container vs Virtual Machines
    - ECR(Elastic Container Registry)
    - ECS(Elastic Container Service)
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
    - DataSync
    - CloudEndure
  - Compute
    - **Custom AMI**
    - Custom Container Image
  - Networking
    - Route53(Load Balancing, Failover)
    - ELB(Load Balancing, FailoveR)
    - VPC
    - Direct Connect(Backup Replication)

## Module 15 (Migration)
■ Application Discovery Service
■ Data Migration Service(DMS)
  - Snowball : PetaByte(60TB)
  - Snowball Edge : 100TB(80TB)
    - 인터넷 연결이 간헐적이거나 없는 위치에서 데이터 수집
  - Snowmobile : ExaByte
  - Schema Conversion Toll (SCT)
■ Server Migration Service
■ Storage Gateway
  - On-Premise Data Storage -> AWS Cloud 연결
■ **Consulting Partner** : Migration 전문가
■ **Outposts**
  - AWS Cloud Service -> On-Premise 삽입 및 하이브리드 아키텍처 확장
  - 낮은 대기 시간 또는 로컬 시스템 상호 종속성이 필요한 애플리케이션에 유용
■ **Local Zones** : 대기 시간 최소화, 연결 중단 X

## Module 16 (Developer Tools)
■ CodeCommit : 버전 관리

■ CodeDeploy

■ CodePipeline

## Module 17 (Business Productivity)
■ WorkDocs
■ WorkMail
■ **Chime**
■ WorkSpaces
■ AppStream : 원격 작업 액세스를 지원하는 완전관리형의 비영구 데스크톱 및 애플리케이션 서비스
■ Marketplace

## Module 18 (Application)
■ API Gateway
■ SWF
■ Elastic Transcoder
■ Step Functions
■ Cloud Search

## Module 19 (Analytics)
■ EMR : Elastic MapReduce, Hadoop Framework
■ Kinesis : Realtime Video & Data Stream Service 수집, 처리, 분석
■ Elastic Search
■ QuickSight : Serverless, 시각적 보고서 생성, BI 서비스
■ Data Pipeline : 서로 다른 사용자의 수백 건 요청 동시 처리
■ RedShift
  - Data Warehouse
  - Column 형식
  - OLAP(Online Analyticla Processing) 용도
■ Glue : Serverless 관리형 ETL(Extract, Transform, Load)
■ Managed Blcokchain

## Module 20 (AI)
■ Machine Learning
■ Rekognition : Vision
■ **Polly** : TTS
■ Lex : ChatBot, 자동 음성 인식, 자연어 이해 기능
■ Transcribe : STT, 자동 음성 인식, ASR 딥러닝 처리 사용
■ Translate : 번역
■ **Comprehend** : NLP, 이메일 메세지 감정분석 등 사용
■ SageMaker : 머신 러닝 모델 구축, 훈련, 배포
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

## Mobuld 23 (Game)
■ GameLift
■ Lumberyard

## 문제
- <https://kananinirav.com/practice-exam/exams.html>
- <https://www.examtopics.com/exams/amazon/aws-certified-cloud-practitioner/view/>

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