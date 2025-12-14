---
title: '[AWS] AWS Solutions Architect Associate(SAA-C03) 자격증 문제 풀이'
date: 2025-01-18 18:10:00 +0900
categories: [인프라/클라우드, AWS]
tags: [AWS]
math: true
mermaid: true
---

## Problem
21. An e-commerce company is launching a website on AWS that sells once a day. Exactly one product is sold every 24 hours. The company wants to be able to handle millions of requests per hour with millisecond latency during peak hours. What solution can meet these requirements with minimal operational overhead?
① A. Host your entire website in multiple S3 buckets using Amazon S3. Add an Amazon CloudFront distribution. Set the S3 bucket as the distribution's origin. Store order data in Amazon S3.
② B. Deploy your entire website on Amazon EC2 instances running in an Auto Scaling group across multiple Availability Zones. Add an Application Load Balancer (ALB) to distribute website traffic. Add another ALB for your backend API. Store your data in Amazon RDS for MySQL.
③ C. Migrate the entire application to run in containers. Host the containers on Amazon Elastic Kubernetes Service (Amazon EKS). Use Kubernetes Cluster Autoscaler to increase or decrease the number of pods to handle traffic bursts. Store data in Amazon RDS for MySQL.
**④ D. Host your website's static content using an Amazon S3 bucket. Deploy an Amazon CloudFront distribution. Set the S3 bucket as the origin. Use Amazon API Gateway and AWS Lambda functions for the backend API. Store data in Amazon DynamoDB.**
```
  최소한의 운영 오버헤드로 하루에 수백만 건의 요청을 처리할 수 있는 솔루션은 AWS의 서버리스 아키텍처를 사용하는 것이다. 정적 콘텐츠는 Amazon S3 버킷에 호스팅되고, Amazon CloudFront 배포를 사용하여 콘텐츠를 캐시하고 전달한다. 백엔드 API에는 Amazon API Gateway와 AWS Lambda 함수를 사용하여 서버리스로 구축하고, 데이터는 Amazon DynamoDB에 저장한다.这样하면 피크 시간에도 밀리초의 대기 시간으로 요청을 처리할 수 있고, 운영 오버헤드도 최소화할 수 있다.

  1번 (S3 + CloudFront): 정적 콘텐츠에 적합하지만, 실시간 주문 처리와 같은 동적 기능을 지원하기에는 부족합니다.
  2번 (EC2 + RDS): 높은 성능을 위해 EC2와 RDS를 사용할 수 있지만, Auto Scaling 설정 및 관리에 오버헤드가 발생합니다.
  3번 (EKS + RDS): 컨테이너 기반 접근 방식은 유연성이 높지만, 복잡한 배포 및 관리가 필요합니다.
  정답인 4번 (S3 + CloudFront + API Gateway + Lambda + DynamoDB)은 다음과 같은 이유로 최적입니다.

  Amazon S3와 CloudFront: 정적 웹 콘텐츠를 효율적으로 제공하며, CloudFront의 캐싱 기능으로 피크 시간대 성능을 향상시킵니다.
  API Gateway + Lambda: 백엔드 API를 서버리스 방식으로 구현하여 확장성과 관리 용이성을 높입니다.
  DynamoDB: NoSQL 데이터베이스로 트래픽 변동에 유연하게 대응하며, 스케일링이 자동화되어 운영 부담을 줄입니다.
```
22. A solution architect is designing a storage architecture for a new digital media application using Amazon S3. Media files must be resilient even if an Availability Zone is lost. Some files are accessed frequently, while others are accessed infrequently and in unpredictable patterns. The solution architect needs to minimize the cost of storing and retrieving media files.
Which storage option meets these requirements?
① A. S3 Standard
② B. S3 Intelligent-Tiering
③ C. S3 Standard-Infrequent Access (S3 Standard-IA)
④ D. S3 One Zone-Infrequent Access(S3 One Zone-IA)

  주어진 요구 사항을 충족하는 최적의 옵션은 B. S3 지능형 계층화(S3 Intelligent-Tiering)입니다.

  복원력: S3 Intelligent-Tiering은 기본적으로 다중 가용 영역에 데이터를 저장할 수 있도록 설정되어 있어, 일부 가용 영역의 손실에도 데이터를 보호합니다. 단, 이는 가용 영역 복제를 명시적으로 활성화해야 완전히 보장되지만, 기본적으로 높은 가용성을 제공합니다.

  액세스 패턴 다양성: 이 옵션은 자동으로 자주 액세스되는 데이터와 그렇지 않은 데이터를 감지하여 비용 효율적인 스토리지 티어로 자동 이동시킵니다. 자주 액세스되는 파일은 더 빠른 액세스 성능을 유지하면서, 거의 액세스되지 않는 파일은 비용 절감을 위해 저렴한 티어로 자동 이동됩니다.

  비용 최적화: 자동화된 계층화 기능 덕분에 자주 사용되는 파일과 그렇지 않은 파일에 대해 각각 최적의 비용 구조를 적용할 수 있습니다. 이는 예측 불가능한 액세스 패턴을 가진 파일들에 대해 특히 유리합니다.

  다른 옵션들과 비교했을 때,

  S3 표준은 일관된 고성능을 제공하지만, 액세스 패턴에 따른 비용 최적화는 제한적입니다.
  S3 스탠다드-IA는 자주 액세스되지 않는 데이터에 저렴한 비용을 제공하지만, 자동화된 액세스 패턴 인식 기능이 없습니다.
  S3 One Zone-IA는 비용 효율적이지만 단일 가용 영역에 데이터를 저장하므로 가용성 측면에서 약점이 있습니다.

23. A company uses Amazon S3 Standard storage to store backup files. These files are accessed frequently for one month. However, after one month, they become inaccessible. The company needs to retain these files indefinitely. What storage solution would most cost-effectively meet these requirements?
① A. Configure S3 Intelligent-Tiering to automatically migrate objects.
② B. Create an S3 lifecycle configuration to transition objects from S3 Standard to S3 Glacier Deep Archive after one month.
③ C. Create an S3 lifecycle configuration to transition objects from S3 Standard to S3 Standard-Infrequent Access (S3 Standard-IA) after one month.
④ D. Create an S3 lifecycle configuration to transition objects from S3 Standard to S3 One Zone-Infrequent Access (S3 One Zone-IA) after one month.

  Amazon S3에는 다양한 스토리지 클래스가 있으며, 클래스는 비용과 데이터 접근 빈도수에 따라 달라집니다. 会社에서 1ヶ月間 자주 접속되는 파일을 저장하고, 그 이후에는 접근할 수 없지만 무기한 보관해야 하는 경우, 비용을 효율적으로 절약하면서 요구 사항을 충족하는 스토리지 솔루션을 찾는 것입니다. 이러한 경우, 초기에는 자주 접근되는 파일을 위한 S3 Standard 스토리지를 사용한 후, 1개월 후에 접근 빈도가 낮아지는 파일을 위한 저렴한 스토리지 클래스로 전환하는 것이 좋습니다.S3 Glacier Deep Archive는 매우 저렴한 비용으로 장기간에 걸쳐 데이터를 보관할 수 있는 스토리지 클래스로, 1개월 후에 접근할 수 없는 파일을 S3 Standard에서 S3 Glacier Deep Archive로 전환하기 위한 S3 수명 주기 구성을 생성하는 것이 가장 비용 효율적으로 요구 사항을 충족하는 방법입니다.

  회사의 요구 사항은 1개월 동안 빈번한 접근 후 무기한 저렴한 비용으로 보관하는 것이다.

  1번 옵션의 S3 Intelligent-Tiering은 사용량에 따라 자동으로 데이터를 재배치하지만, 장기 보관 비용을 최적화하는 데는 적합하지 않다.

  3번과 4번 옵션의 S3 Standard-IA 및 S3 One Zone-IA는 액세스 빈도가 낮은 데이터에 더 적합하며, 여전히 비교적 높은 비용이 들 수 있다.

  정답인 2번 옵션은 S3 수명 주기 정책을 활용해 1개월 후 S3 Standard에서 S3 Glacier Deep Archive로 자동 전환하는 방법을 제시한다. S3 Glacier Deep Archive는 장기 보관에 최적화되어 있으며, 매우 저렴한 비용으로 데이터를 무기한 보관할 수 있다. 따라서 비용 효율성과 장기 보관 요구 사항을 동시에 충족하는 최적의 선택이다.

24. A company has noticed an increase in Amazon EC2 costs in their most recent bill. The billing team has discovered unwanted vertical scaling of several EC2 instances. A solutions architect needs to create a graph comparing EC2 costs over the past two months and conduct in-depth analysis to identify the root cause of the vertical scaling.
How can the solutions architect generate this information while minimizing operational overhead?
① A. Use AWS Budgets to generate budget reports and compare EC2 costs across instance types.
② B. Use Cost Explorer's granular filtering capabilities to perform in-depth analysis of your EC2 costs based on instance type.
③ C. Use the graphs on the AWS Billing and Cost Management dashboard to compare your EC2 costs by instance type over the past two months.
④ D. Generate a report using the AWS Cost and Usage Report and send it to an Amazon S3 bucket. Use Amazon QuickSight as a source with Amazon S3 to create interactive graphs based on instance type.

  INTRROAR (운영 오버헤드 최소화) _cost Explorer의 세분화된 필터링 기능을 사용하면 인스턴스 유형별 EC2 비용에 대한 심층 분석을 쉽게 수행할 수 있습니다. 이는 가장 효율적인 방법입니다. AWS 예산, AWS Billing and Cost Management 대시보드, AWS 비용 및 사용 보고서를 사용하면 정보를 생성할 수 있지만, 이들보다 Cost Explorer가 더 간단하고 효율적입니다. 따라서 Cost Explorer의 세분화된 필터링 기능을 사용하는 것이 가장 좋습니다.

  Cost Explorer는 고급 필터링과 세분화된 분석 기능을 제공하여 특정 기간과 인스턴스 유형별 비용 변화를 쉽게 파악할 수 있게 합니다. 이 방법은 다음과 같은 이유로 가장 적합합니다:

  즉시성: 실시간 데이터 분석이 가능해, 최근 2개월간의 비용 변동을 빠르게 비교 분석할 수 있습니다.
  깊이 있는 분석: 인스턴스 유형별로 비용을 세분화해 분석할 수 있어, 수직 확장의 정확한 원인을 쉽게 식별할 수 있습니다.
  간편성: 복잡한 설정 없이도 필요한 데이터를 쉽게 추출하고 시각화할 수 있어 운영 오버헤드를 최소화합니다.
  다른 옵션들도 유용하지만,

  1번과 3번은 기본적인 예산 및 대시보드 기능으로 심층 분석에 한계가 있습니다.
  4번은 데이터 추출과 시각화에 더 많은 수동적인 설정과 시간이 필요하며, 즉각적인 분석에는 적합하지 않습니다.

25. A company is designing an application. The application uses an AWS Lambda function to receive information through Amazon API Gateway and store it in an Amazon Aurora PostgreSQL database.
During the proof-of-concept phase, the company needs to significantly increase Lambda quotas to handle the large amount of data that needs to be loaded into the database. The solution architect needs to recommend a new design to improve scalability and minimize configuration effort.
What solution meets these requirements?
① A. Refactor your Lambda function code to Apache Tomcat code running on an Amazon EC2 instance. Connect to the database using the native Java Database Connectivity (JDBC) driver.
② B. Change the platform from Aurora to an Amazon DynamoDProvision DynamoDB Accelerator (DAX) cluster. Use the DAX client SDK to point existing DynamoDB API calls to the DAX cluster.
③ C. Set up two Lambda functions. Configure one function to receive the information. Configure the other function to load the information into a database. Integrate the Lambda functions using Amazon Simple Notification Service (Amazon SNS).
④ D. Set up two Lambda functions. Configure one function to receive information. Configure the other function to load the information into a database. Integrate the Lambda functions using an Amazon Simple Queue Service (Amazon SQS) queue.

  해당 문제는 AWS Lambda 함수의 확장성과 구성 노력을 최소화하는 방법을 묻는 것입니다. 이 문제를 해결하기 위해 두 개의 Lambda 함수를 설정하여 정보를 수신하고 데이터베이스에 로드하는 과정으로 나누어 처리하는 방법이 제시됩니다.

  이 경우 Amazon Simple Queue Service(Amazon SQS) 대기열을 사용하여 Lambda 함수를 통합하는 것이 적절한 해결책입니다. Lambda 함수를 두 개로 분리하여 정보를 수신하는 함수와 데이터베이스에 로드하는 함수로 나누면 각 함수의 역할을 분명하게 정의할 수 있습니다. Amazon SQS 대기열을 사용하면 정보를 수신하는 Lambda 함수가 큐에 메시지를 전송하고, 데이터베이스에 로드하는 Lambda 함수가 큐에서 메시지를 가져와 처리할 수 있습니다.

  이러한 설계는 확장성을 향상시키고, 구성 노력을 최소화하며, 대량의 데이터를 처리할 수 있는 능력을 제공합니다. 또한, Amazon SQS를 사용하여 Lambda 함수를 통합하면 함수 사이의 결합도를 낮추고, 오류 처리와 재시도를 쉽게 할 수 있습니다. 결론적으로, 두 개의 Lambda 함수를 설정하고 Amazon SQS 대기열을 사용하여 함수를 통합하는 방법이 이 문제의 해법입니다.

  대량의 데이터 처리를 위해 확장성과 구성의 간소화를 추구하는 상황에서, Amazon SQS를 활용하는 접근법이 가장 적합합니다.

  두 개의 Lambda 함수를 사용하는 방식은 작업을 분리하여 효율성을 높입니다:

  첫 번째 Lambda 함수는 API Gateway로부터 데이터를 수신합니다.
  수신된 데이터는 Amazon SQS 대기열에 적재됩니다. 이는 데이터 처리의 부하를 분산시키고, Lambda 함수가 직접적으로 부하 변동에 노출되는 것을 방지합니다.
  두 번째 Lambda 함수는 SQS에서 데이터를 가져와 Amazon Aurora PostgreSQL 데이터베이스에 안정적으로 로드합니다.
  이러한 구조는 다음과 같은 이점을 제공합니다:

  확장성: SQS는 자동으로 확장되므로, 데이터 수신량의 변동에 유연하게 대응할 수 있습니다.
  고가 Observatory: 대기열을 통해 데이터 처리의 병목 현상을 완화하고, 각 Lambda 함수가 독립적으로 확장될 수 있도록 합니다.
  안정성: 데이터 손실 위험을 줄이고, 메시지가 안전하게 처리될 때까지 대기열에 보관됩니다.
  다른 옵션들 중에서, EC2 인스턴스로의 이전은 관리 복잡성을 증가시키고, DynamoDB로의 전환은 원래 문제의 핵심인 PostgreSQL 데이터베이스 사용과 맞지 않습니다. 따라서 4번 옵션이 최적의 솔루션입니다.

26. A company needs to review its AWS Cloud deployment to ensure there are no unauthorized configuration changes to its Amazon S3 buckets. What should a solutions architect do to achieve this goal?
① A. Enable AWS Config using the appropriate rules.
② B. Enable AWS Trusted Advisor by performing appropriate checks.
③ C. Enable Amazon Inspector using the appropriate assessment template.
④ D. Enable Amazon S3 server access logging. Configure Amazon EventBridge (Amazon CloudWatch events).

  Amazon S3 버킷에 무단 구성 변경이 없는지 확인하기 위해 AWS 클라우드 배포를 검토해야 합니다. 이 목표를 달성하려면 솔루션 아키텍트가 AWS Config를 활성화해야 합니다. AWS Config는 AWS 리소스의 구성과 컴플라이언스를 모니터링하고 기록하는 데 사용됩니다. 적절한 규칙을 사용하여 AWS Config를 활성화하면 Amazon S3 버킷의 구성 변경을 검토하고 무단 변경을 обнаруж할 수 있습니다. 따라서 솔루션 아키텍트는 적절한 규칙을 사용하여 AWS Config를 활성화해야 합니다.

  회사가 Amazon S3 버킷의 무단 구성 변경을 감지하려면 AWS Config가 가장 적합한 솔루션입니다.

  AWS Config는 리소스의 상태를 기록하고 변경 사항을 추적합니다. 적절한 규칙을 설정하면 S3 버킷 구성 변경을 감지하고 알림을 받거나, 규정 준수를 위한 보고서를 생성할 수 있습니다.

  다른 옵션들은 이 목표에 적합하지 않습니다.

  AWS Trusted Advisor는 성능, 보안, 비용 최적화 등 다양한 측면에서 권장 사항을 제공하지만, 실시간으로 구성 변경을 감지하는 기능은 없습니다.
  Amazon Inspector는 취약점 스캐닝에 초점을 맞추고 있으며, 구성 변경 감지에는 적합하지 않습니다.
  S3 서버 액세스 로깅과 EventBridge는 활동을 모니터링하고 분석하는 데 유용하지만, 구성 변경 자체를 직접적으로 감지하기에는 제한적입니다.

27. A company is launching a new application and displays application metrics on an Amazon CloudWatch dashboard. The company's product manager needs regular access to this dashboard. The product manager does not have an AWS account. The solution architect must provide access to the product manager based on the principle of least privilege.
Which solution meets this requirement?
① A. Share the dashboard in the CloudWatch console. Enter the product manager's email address and complete the sharing steps. Provide the product manager with a shareable link to the dashboard.
② B. Create an IAM user specifically for the product manager. Attach the CloudWatchReadOnlyAccess AWS managed policy to the user. Share the new login credentials with the product manager. Share the browser URL for the appropriate dashboard with the product manager.
③ C. Create an IAM user for your company's employees. Attach the ViewOnlyAccess AWS managed policy to the IAM user. Share the new login credentials with your product manager. Ask your product manager to go to the CloudWatch console and find the dashboard by name in the Dashboards section.
④ D. Deploy a bastion server in the public subnet. If product managers need access to the dashboard, start the server and share the RDP credentials. Ensure that their browsers are configured to open the dashboard URL using cached AWS credentials that have the appropriate permissions to view the dashboard on the bastion server.

  PRODUCTS 관리자에게 AWS 계정이 없기 때문에 새로운 AWS 계정을 생성할 필요는 없습니다. 또한 대시보드에 대한 액세스만 필요한 상황이므로, 최소 권한 원칙에 따라 제품 관리자에게 필요한 권한만 부여하면 됩니다.

  CloudWatch 대시보드를 공유하는 것은 다른 사용자와 대시보드를 공유할 수 있는 간단한 방법입니다. 대시보드를 공유하면 제품 관리자는 대시보드에 대한 읽기 전용 액세스 권한을 갖게 되며, 별도의 AWS 계정이 필요하지 않습니다. 따라서 PRODUCTS 관리자에게는 CloudWatch 콘솔에서 대시보드를 공유하고 공유 가능한 링크를 제공하는 것이 가장 적절한 솔루션입니다.

  정답은 1번입니다.

  최소 권한 원칙에 따라 제품 관리자에게 필요 최소한의 접근 권한만 부여해야 합니다. 1번 옵션은 CloudWatch 대시보드를 공유 링크를 통해 접근할 수 있게 하는 방법을 제시합니다. 이 접근법은 제품 관리자에게 직접 AWS 계정이나 복잡한 IAM 설정 없이도 필요한 정보에 쉽게 접근할 수 있는 링크를 제공합니다.

  다른 옵션들은 다음과 같은 이유로 적절하지 않습니다:

  2번과 3번은 제품 관리자에게 IAM 사용자 계정을 생성하고 관리하는 과정이 필요합니다. 이는 관리의 복잡성을 증가시키고 최소 권한 원칙을 완벽하게 준수하지 못하게 할 수 있습니다.
  4번은 배스천 서버를 사용하는 방법으로, 보안 위험을 증가시키며 복잡한 인프라 관리가 필요하게 됩니다. 이는 단순성과 보안 측면에서 비효율적입니다.

28. A company is migrating its applications to AWS. The applications are deployed across different accounts. The company uses AWS Organizations to centrally manage the accounts. The company's security team needs a single sign-on (SSO) solution for all of its accounts. The company needs to continue managing users and groups in its on-premises, self-managed Microsoft Active Directory.
What solution meets these requirements?
① A. Activate AWS Single Sign-On (AWS SSO) in the AWS SSO console. Connect your company's self-managed Microsoft Active Directory to AWS SSO by creating a one-way forest trust or one-way domain trust using AWS Directory Service for Microsoft Active Directory.
② B. Enable AWS Single Sign-On (AWS SSO) in the AWS SSO console. Create a two-way forest trust using AWS Directory Service for Microsoft Active Directory to connect your company's self-managed Microsoft Active Directory to AWS SSO.
③ C. Use AWS Directory Service. Establish a two-way trust relationship with your company's self-managed Microsoft Active Directory.
④ D. Deploy an identity provider (IdP) on-premises. Enable AWS Single Sign-On (AWS SSO) in the AWS SSO console.

  한 회사에서 AWS로 애플리케이션을 마이그레이션하고 AWS Organizations를 사용해 계정을 중앙에서 관리할 때, 회사 보안 팀은 모든 계정에 대한 SSO 솔루션이 필요합니다. 이때 회사는 온프레미스 자체 관리형 Microsoft Active Directory에서 사용자 및 그룹을 계속 관리해야 합니다.

  AWS SSO를 활성화하고 Microsoft Active Directory용 AWS Directory Service를 사용해 양방향 포리스트 신뢰를 생성하면 회사 내 모든 계정에 대해 SSO를 제공할 수 있습니다. 이 설정은 회사의 자체 관리형 Microsoft Active Directory와 AWS SSO를 연결해 사용자 및 그룹을 계속 관리할 수 있게 해줍니다.

  따라서 suchen 요구 사항을 충족하는 올바른 솔루션은 AWS SSO를 활성화하고 Microsoft Active Directory용 AWS Directory Service를 사용해 양방향 포리스트 신뢰를 생성하는 것입니다. 이 방법은 회사의 온프레미스 자체 관리형 Microsoft Active Directory와 AWS 계정들 간의 신뢰 관계를 구축할 때 가장 효과적이고 유연한 솔루션입니다.

  회사의 요구 사항을 충족시키기 위해서는 AWS SSO를 통해 중앙화된 인증을 구축하면서, 동시에 기존의 온프레미스 Microsoft Active Directory를 유지하고 통합해야 합니다.

  옵션 1과 3은 단방향 트러스트를 제안하지만, 이는 온프레미스 AD와 AWS SSO 간의 양방향 인증 및 권한 관리를 완벽하게 지원하지 못합니다. 특히 회사가 온프레미스 AD에서 계속 사용자 및 그룹 관리를 필요로 하는 상황에서는 제한적입니다.
  옵션 4는 별도의 온프레미스 IdP를 배포하는 방식으로, 이미 잘 구축된 Active Directory를 활용하지 않는다는 점에서 비효율적입니다.
  정답인 옵션 2는 AWS SSO를 활성화하고, Microsoft Active Directory용 AWS Directory Service를 통해 양방향 포리스트 트러스트를 설정합니다. 이 방식은 다음과 같은 이점을 제공합니다:

  양방향 트러스트: 온프레미스 AD와 AWS SSO 간에 양방향 인증이 가능해져, 사용자가 온프레미스 또는 AWS 리소스에 원활하게 접근할 수 있습니다.
  통합 관리: 기존의 온프레미스 AD에서 사용자 및 그룹 관리를 계속 유지하면서, AWS 내 여러 계정에 대한 중앙화된 SSO 접근을 보장합니다.
  따라서, 회사의 모든 계정에 대한 보안과 관리 효율성을 동시에 충족시키는 최적의 솔루션은 옵션 2입니다.

29. A company offers a Voice over Internet Protocol (VoIP) service using UDP connections. This service consists of Amazon EC2 instances running in an Auto Scaling group. The company is deployed across multiple AWS regions.
The company needs to route users to the region with the lowest latency. The company also needs automated failover between regions.
What solution meets these requirements?
① A. Deploy a Network Load Balancer (NLB) and associated target group. Associate the target group with an Auto Scaling group. Use the NLB as an AWS Global Accelerator endpoint in each region.
② B. Deploy an Application Load Balancer (ALB) and associated target group. Associate the target group with an Auto Scaling group. Use the ALB as an AWS Global Accelerator endpoint in each region.
③ C. Deploy a Network Load Balancer (NLB) and associated target groups. Associate the target groups with an Auto Scaling group. Create an Amazon Route 53 latency record pointing to the alias of each NLB. Create an Amazon CloudFront distribution that uses the latency record as the origin.
④ D. Deploy an Application Load Balancer (ALB) and associated target groups. Associate the target groups with an Auto Scaling group. Create an Amazon Route 53 weighted record pointing to the alias of each ALB. Deploy an Amazon CloudFront distribution using the weighted record as the origin.

  이 lluminate 회사의 요구 사항을 만족하는 솔루션은 1번입니다. NLB는 udp 프로토콜을 지원하고 네트워크 구성이 복잡하지 않기 때문에 회사의 VoIP 서비스에 적합합니다. 각 리전에서 NLB를 AWS Global Accelerator 엔드포인트로 사용하면 사용자를 지연 시간이 가장 짧은 지역으로 라우팅할 수 있습니다. 또한 Global Accelerator는 자동 장애 조치 기능을 제공하므로 지역 간 자동화된 장애 조치도 가능합니다. 따라서 회사의 모든 요구 사항을 충족하는 솔루션은 NLB와 Global Accelerator를 사용하는 것입니다. ALB는 HTTP와 HTTPS 프로토콜만을 지원하기 때문에 이 회사의 VoIP 서비스에는 적합하지 않습니다. Route 53는 DNS 서비스이고 CloudFront는 CDN 서비스이므로 이 경우에는 NLB와 Global Accelerator만 사용하는 것이 더 적합합니다. 따라서 정답은 1번입니다.

  정답은 1번인 이유는 다음과 같습니다.

  UDP 기반의 VoIP 서비스는 저지연이 필수적입니다. NLB(Network Load Balancer)는 TCP와 UDP 프로토콜을 지원하며, 특히 UDP 트래픽에 최적화되어 저지연 특성을 제공합니다. 이는 VoIP 서비스의 요구 사항에 잘 맞습니다.

  AWS Global Accelerator는 각 리전의 NLB를 엔드포인트로 사용함으로써, 전 세계 어디서든 사용자에게 가장 가까운 NLB를 통해 트래픽을 라우팅하여 지연 시간을 최소화합니다.
  장애 조치: Global Accelerator는 자동으로 대체 경로를 제공하여 지역 간 장애 시에도 서비스 지속성을 보장합니다.
  다른 옵션들은 적합하지 않은 이유는:

  ALB(Application Load Balancer)는 주로 HTTP/HTTPS 트래픽에 최적화되어 있어 UDP 트래픽에 대한 성능과 저지연 특성이 NLB만큼 우수하지 않습니다.
  Route 53 지연 시간 기반 레코드와 CloudFront (옵션 C, D)는 추가적인 네트워크 홉을 통해 지연 시간이 증가할 가능성이 있으며, 복잡성도 높아집니다. 특히 VoIP와 같은 저지연 서비스에서는 이러한 추가 레이어가 불필요한 지연을 초래할 수 있습니다.
  따라서, 저지연과 자동화된 장애 조치를 동시에 만족시키는 가장 효율적인 솔루션은 NLB와 AWS Global Accelerator의 조합입니다. 정답은 1번입니다.

30. The development team runs resource-intensive tests monthly against a general-purpose Amazon RDS for MySQL DB instance with Performance Insights enabled. These tests run once a month for 48 hours and are the only processes that use the database. The team wants to reduce test execution costs without reducing the compute and memory properties of the DB instance.
What is the most cost-effective solution to meet these requirements?
① A. Stop the DB instance once testing is complete. Restart the DB instance if necessary.
② B. Use an Auto Scaling policy with your DB instance to automatically scale it once testing is complete.
③ C. Once testing is complete, create a snapshot. Terminate the DB instance and restore the snapshot if necessary.
④ D. After testing is complete, modify the DB instance to a smaller size. If necessary, modify the DB instance again.

  팀은 매월 48시간 동안 리소스 집약적인 테스트를 실행하며, 데이터베이스를 사용하는 유일한 프로세스입니다. 컴퓨팅 및 메모리 속성을 줄이지 않고 테스트 실행 비용을 줄이고 싶어합니다. 이러한 요구 사항을 가장 비용 효율적으로 충족하는 솔루션은 테스트가 완료되면 스냅샷을 생성하고 DB 인스턴스를 종료한 후 필요한 경우 스냅샷을 복원하는 것입니다. 이 방법은 DB 인스턴스를 중지하거나 수정할 필요가 없으며, 스냅샷을 사용하여 빠르게 복원할 수 있습니다. 또한 Auto Scaling 정책을 사용할 경우 인스턴스의 크기를 자동으로 조정해야 하므로 비용 효율성이 저하될 수 있습니다. 또한 DB 인스턴스를 저용량 인스턴스로 수정하는 경우에도 인스턴스의 속성이 변경되며, 이는 팀의 요구 사항과 다릅니다. 따라서 테스트가 완료되면 스냅샷을 생성하고 DB 인스턴스를 종료한 후 필요한 경우 스냅샷을 복원하는 것이 가장 비용 효율적인 솔루션입니다.

  테스트 기간 외에는 자원 활용이 불필요하기 때문에 비용 최적화를 위해 DB 인스턴스를 비활성화하는 것이 핵심입니다. 옵션 3은 테스트 종료 후 스냅샷을 생성하여 데이터의 안전성을 확보하고, 그 후 DB 인스턴스를 종료하는 방법을 제시합니다. 이렇게 하면 테스트 기간 외에는 요금이 부과되지 않아 가장 비용 효율적입니다.

  옵션 1은 인스턴스를 중지하는 방법으로 비용 절감 효과는 있지만, 중지 후 재개 시에는 인스턴스 복구 시간이 발생할 수 있으며, 일정 시간 경과 후 자동 재개되지 않을 때 추가 비용이 발생할 위험이 있습니다.

  옵션 2의 Auto Scaling은 테스트 기간 동안 필요한 리소스를 동적으로 조절하는 데 유용하지만, 테스트 외에도 일정한 비용이 지속적으로 발생하므로 전체적인 비용 절감 효과는 제한적입니다.

  옵션 4는 인스턴스 크기 조정을 제안하지만, 크기 조정 자체에도 비용이 들고, 테스트 기간 동안 필요한 성능을 확보하기 위해 원래의 인스턴스 크기로 다시 조정해야 하므로 효율적이지 않습니다.

  따라서, 데이터 보호와 함께 비용을 최소화하는 가장 효과적인 방법은 3번 옵션이 됩니다.

31. A company hosting a web application on AWS wants to ensure that all Amazon EC2 instances, Amazon RDS DB instances, and Amazon Redshift clusters are tagged. The company wants to minimize the effort required to configure and operate these checks. What should a solutions architect do to achieve this?
① A. Use AWS Config rules to define and detect resources that are not appropriately tagged.
② B. Use the Cost Explorer to identify resources that are not properly tagged. Manually tag these resources.
③ C. Write an API call that checks all resources for appropriate tag assignment. Run the code periodically on your EC2 instance.
④ D. Write an API call that checks all resources for appropriate tag assignment. Schedule an AWS Lambda function through Amazon CloudWatch to run the code periodically.

  AWS Config 규칙을 사용하여 적절하게 태그가 지정되지 않은 리소스를 정의하고 감지하면 회사의 노력과 비용을 줄일 수 있습니다. AWS Config는 AWS 리소스의 인벤토리 및 구성 추적, 리소스 구성의 감사 및 평가를 자동화하는 서비스입니다. AWS Config 규칙을 사용하면 태그가 제대로 지정되지 않은 리소스를 자동으로 감지하고 보고할 수 있습니다. 이를 통해 회사에서는 태그가 제대로 지정되지 않은 리소스를 쉽게 식별하고 제대로 태그를 지정할 수 있습니다. 또한 AWS Config는 자동으로 리소스를 감사하고 평가하여 태그가 제대로 지정되지 않은 리소스를 실시간으로 감지할 수 있습니다. 이를 통해 회사는 태그 지정과 관련한 수동적인 작업을 줄일 수 있고, 태그가 제대로 지정되지 않은 리소스를 자동으로 감지하고 수정할 수 있습니다.

  정답은 1번입니다. AWS Config는 리소스의 구성을 지속적으로 모니터링하고 규정 준수 여부를 확인하는 데 이상적입니다. 적절한 태그가 지정되지 않은 Amazon EC2 인스턴스, Amazon RDS DB 인스턴스, Amazon Redshift 클러스터 등을 자동으로 감지하고 알림을 제공할 수 있습니다. 이를 통해 수동 개입을 최소화하고 자동화된 방식으로 규정 준수를 유지할 수 있습니다.

  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  2번은 비용 탐색기는 주로 비용 분석에 초점을 맞추며, 태그 관리에 특화되어 있지 않습니다. 또한 수동 개입이 필요해 효율성이 떨어집니다.
  3번은 주기적인 API 호출과 코드 실행을 통해 가능하지만, 이는 수동 관리와 비슷한 수준의 노력이 필요하며 자동화 수준이 낮습니다.
  4번은 자동화 측면에서 개선된 접근이지만, AWS Config만큼 직접적이고 통합된 규정 준수 모니터링 및 알림 기능을 제공하지 않습니다. 따라서 가장 효율적이고 자동화된 해결책은 AWS Config 규칙을 활용하는 1번입니다.

32. The development team needs to host a website that other teams can access. Website content consists of HTML, CSS, client-side JavaScript, and images. What is the most cost-effective way to host the website?
① A. Containerize your website and host it on AWS Fargate.
② B. Create an Amazon S3 bucket and host your website there.
③ C. Host your website by deploying a web server on an Amazon EC2 instance.
④ D. Configure an Application Load Balancer as an AWS Lambda target using the Express.js framework.

  웹사이트 호스팅의 가장 비용 효과적인 방법을 결정하기 위해 각 선택지를 비교해 볼 수 있다.

  첫 번째 선택지는 컨테이너화된 웹사이트를 AWS Fargate에서 호스팅하는 것인데, 이 방법은 컨테이너화된 애플리케이션을 관리하고 확장하는 데 편리하지만, 단순한 정적 웹사이트 호스팅에는 비교적 복잡하고 비용이 높을 수 있다.

  두 번째 선택지는 Amazon S3 버킷을 생성하고 거기에서 웹사이트를 호스팅하는 것으로, 정적 웹사이트를 호스팅하는 데 적합하며 비용 효율적이다. S3는 정적 콘텐츠를 저장하고 제공하는 데 최적화되어おり,HttpServletRequest와 같은 동적 요청이 없，因此 비용이 낮다.

  세 번째 선택지는 Amazon EC2 인스턴스에 웹 서버를 배포하여 웹 사이트를 호스팅하는 것으로, 정적 웹사이트 호스팅에는 과도한 자원과 비용을 필요로 할 수 있다. EC2 인스턴스는 동적 콘텐츠나 복잡한 애플리케이션을 호스팅하는 데 적합하지만, 단순한 정적 웹사이트에는 불필요한 비용을 초래할 수 있다.

  네 번째 선택지는 Express.js 프레임워크를 사용하는 AWS Lambda 대상으로 Application Load Balancer를 구성하는 것으로, 동적 콘텐츠나 서버리스 아키텍처를 구축하는 데 적합하지만, 정적 웹사이트 호스팅에는 비교적 복잡하고 비용이 높을 수 있다.

  따라서 정적 웹사이트 콘텐츠를 호스팅하는 데에는 두 번째 선택지인 Amazon S3 버킷을 생성하고 거기에서 웹사이트를 호스팅하는 것이 가장 비용 효과적인 방법이다.

  정답은 2번입니다. Amazon S3 버킷을 사용한 웹사이트 호스팅이 가장 비용 효율적입니다.

  이유는 다음과 같습니다:

  Amazon S3는 정적 콘텐츠(HTML, CSS, 클라이언트측 JavaScript, 이미지 등)를 저장하고 제공하는 데 특화되어 있습니다.
  정적 웹사이트 호스팅 시 S3는 기본적으로 무료로 대역폭과 요청을 제공하며, 데이터 저장 비용만 청구됩니다.
  AWS Fargate (1번)와 EC2 인스턴스 (3번)는 컨테이너나 가상 서버를 필요로 하므로 지속적인 인스턴스 유지 비용과 함께 컴퓨팅 리소스 사용 비용이 발생합니다. 이는 정적 콘텐츠 호스팅에 비해 과도한 비용이 될 수 있습니다.
  AWS Lambda와 Application Load Balancer (4번)는 주로 서버리스 애플리케이션이나 백엔드 로직 처리에 최적화되어 있습니다. 클라이언트 측에서 직접적으로 액세스하는 정적 웹사이트 호스팅에는 불필요한 복잡성과 비용이 추가될 수 있습니다.
  따라서, 단순한 정적 웹사이트의 경우, Amazon S3가 가장 간단하고 비용 효율적인 선택입니다.

33. A company is running an online marketplace web application on AWS. This application serves hundreds of thousands of users during peak hours. The company needs a scalable, near-real-time solution to share details of millions of financial transactions with multiple internal applications. Transactions also need to be processed to remove sensitive data before being stored in a document database for low-latency retrieval.
What should a solution architect recommend to meet these requirements?
① A. Store transaction data in Amazon DynamoDB. Set up rules in DynamoDB to remove sensitive data from all transactions upon write. Share transaction data with other applications using DynamoDB Streams.
② B. Stream transaction data to Amazon Kinesis Data Firehose and store it in Amazon DynamoDB and Amazon S3. Use Kinesis Data Firehose integration with AWS Lambda to remove sensitive data. Other applications can use the data stored in Amazon S3.
③ C. Stream transaction data to Amazon Kinesis Data Streams. Use AWS Lambda integration to remove sensitive data from all transactions and then store the transaction data in Amazon DynamoDB. Other applications can then consume the transaction data from the Kinesis data stream.
④ D. Store the batched transaction data as files in Amazon S3. Before updating the files in Amazon S3, use AWS Lambda to process all files and remove sensitive data. The Lambda function then stores the data in Amazon DynamoDB. Other applications can use the transaction files stored in Amazon S3.

  거래 데이터의 처리와 민감한 정보의 제거를 위해 실시간에 가까운 솔루션이 필요합니다. 또한 짧은 지연 시간의 검색을 위해 문서 데이터베이스에 저장되기 전에 민감한 데이터를 제거해야 합니다.

  이러한 요구 사항을 충족하기 위해, 트랜잭션 데이터를 Amazon Kinesis Data Streams로 스트리밍하고, AWS Lambda 통합을 사용하여 모든 트랜잭션에서 중요한 데이터를 제거한 다음 Amazon DynamoDB에 트랜잭션 데이터를 저장하는 것이 적절합니다. 다른 애플리케이션은 Kinesis 데이터 스트림에서 트랜잭션 데이터를 사용할 수 있습니다.

  이러한 설계에서는 트랜잭션 데이터가 실시간에 가까운 속도로 처리되고 민감한 정보가 제거된 후에 다른 애플리케이션과 공유되며, 짧은 지연 시간의 검색도 지원됩니다openidesát supplementollectors가.withOpacity 말이 않_coefjav(getApplicationContext)보다Better`t 데이터 처리 및 공유를 위한 확장 가능한 솔루션을 제공합니다.

  정답은 3번입니다.

  실시간 처리와 확장성: Amazon Kinesis Data Streams는 높은 스루풋과 낮은 지연 시간으로 실시간 데이터 스트리밍에 이상적입니다. 이는 수십만 명의 사용자와 수백만 건의 거래를 처리하는 데 필요한 확장성을 제공합니다.
  데이터 처리와 민감 정보 제거: AWS Lambda와의 통합을 통해 Kinesis Data Streams에서 바로 트랜잭션 데이터를 처리할 수 있습니다. 이 과정에서 민감한 데이터를 실시간으로 제거할 수 있어 보안 요구 사항을 충족합니다.
  데이터 저장과 공유: 처리된 데이터를 Amazon DynamoDB에 저장하면 빠른 검색과 접근이 가능합니다. 다른 내부 애플리케이션들은 여전히 Kinesis Data Streams를 통해 실시간 데이터에 접근할 수 있어 다양한 사용 사례를 지원합니다.
  따라서, 옵션 3은 실시간 처리, 민감 데이터 보호, 확장성, 그리고 다양한 애플리케이션 간의 효율적인 데이터 공유를 모두 충족하는 최적의 솔루션입니다. 옵션 1은 DynamoDB의 규칙 설정이 제한적일 수 있고, 옵션 2와 4는 지연 시간이 증가할 가능성이 있어 요구 사항에 비해 덜 적합합니다.

34. A company hosts a multi-tier application on AWS. For compliance, governance, auditing, and security purposes, the company needs to track configuration changes to AWS resources and log API calls to these resources. What should a solution architect do to meet these requirements?
① A. Use AWS CloudTrail to track configuration changes and AWS Config to log API calls.
② B. Use AWS Config to track configuration changes and AWS CloudTrail to log API calls.
③ C. Use AWS Config to track configuration changes and Amazon CloudWatch to log API calls.
④ D. Use AWS CloudTrail to track configuration changes and Amazon CloudWatch to log API calls.

  회사는 AWS 리소스의 구성 변경 사항과 API 호출 기록을 모두 추적해야 합니다. 이를 위해 적합한 서비스 선택이 중요합니다.

  AWS Config는 리소스의 구성 상태를 시간에 따라 추적하고 변경 사항을 기록하는 데 최적화되어 있습니다. 즉, 리소스의 설정이나 구조적인 변경 사항을 모니터링하는 데 이상적입니다.
  AWS CloudTrail은 AWS 계정 내에서 이루어진 API 호출을 기록합니다. 이는 사용자 활동, 권한 변경, 서비스 사용 패턴 등에 대한 감사 로그를 제공합니다.
  따라서, 구성 변경 사항을 추적하려면 AWS Config를, API 호출 기록을 위해선 AWS CloudTrail을 사용해야 합니다. 이로 인해 정답은 2번입니다:

  B. AWS Config를 사용하여 구성 변경 사항을 추적하고 AWS CloudTrail을 사용하여 API 호출을 기록합니다.

35. A company is preparing to launch a public web application on the AWS Cloud. The architecture consists of Amazon EC2 instances within a VPC behind an Elastic Load Balancer (ELB). A third-party DNS service is used. The company's solution architect needs to recommend a solution capable of detecting and defending against large-scale DDoS attacks.
Which solution meets these requirements?
① A. Enable Amazon GuardDuty in your account.
② B. Enable Amazon Inspector on your EC2 instance.
③ C. Enable AWS Shield and assign Amazon Route 53.
④ D. Enable AWS Shield Advanced and assign an ELB to it.

  DDoS 공격을 탐지하고 방어하기 위해서는 적절한 솔루션이 필요합니다. 제시된 상황에서 ELB 뒤의 VPC 내의 Amazon EC2 인스턴스와 타사 서비스의 DNS가 사용됩니다. 이 경우 DDoS 공격을 방어하기 위해 AWS Shield Advanced를 활성화하고 ELB를 할당하는 것이 적절한 솔루션입니다.

  AWS Shield Advanced는 DDoS 공격을 자동으로 탐지 및 방어하여 애플리케이션의 가용성과 성능을 유지하도록 지원합니다. 또한 ELB를 할당하면 로드 밸런서를 통해 트래픽이 분산되는 동안 공격이 차단되므로 애플리케이션의 보안이 강화됩니다.

  다른 옵션은 DDoS 공격을 탐지하고 방어하는 데 필요한ระด의 종합적이고 자동화된 보호를 제공하지 않습니다. 따라서 AWS Shield Advanced와 ELB 할당이 सबस적합한 솔루션입니다.

  대규모 DDoS 공격에 대비한 최적의 솔루션은 D. AWS Shield Advanced를 활성화하고 여기에 ELB를 할당하는 것입니다.

  AWS Shield Basic은 기본적인 보호를 제공하지만, 대규모 공격에는 한계가 있습니다. AWS Shield Advanced는 더 강력한 DDoS 방어 기능을 제공하며, 특히 프로액티브 차단 및 실시간 모니터링을 통해 공격을 효과적으로 관리할 수 있습니다. 이 서비스를 Elastic Load Balancer (ELB)에 연결하면, 트래픽의 이상 징후를 즉시 감지하고 공격 트래픽을 차단하여 애플리케이션의 가용성을 유지할 수 있습니다.

  다른 옵션들을 살펴보면:

  Amazon GuardDuty (A)는 의심스러운 활동과 보안 위협을 감지하는데 효과적이지만, 직접적인 DDoS 방어에는 적합하지 않습니다.
  Amazon Inspector (B)는 주로 인스턴스의 보안 취약성을 스캔하는 데 사용되므로 DDoS 방어와는 직접 관련이 적습니다.
  AWS Shield와 Amazon Route 53 (C) 조합은 일부 보호를 제공하지만, Route 53은 주로 DNS 관리에 초점을 맞추고 있고, 대규모 공격에 대한 프로액티브 방어 기능은 AWS Shield Advanced만큼 강력하지 않습니다.
  따라서, 대규모 DDoS 공격을 효과적으로 탐지하고 방어하기 위해서는 D 옵션이 가장 적합합니다.

36. A company is building an application on the AWS Cloud. The application stores data in Amazon S3 buckets in two AWS regions. The company needs to encrypt all data stored in the S3 buckets using AWS Key Management Service (AWS KMS) customer-managed keys. Data in both S3 buckets must be encrypted and decrypted using the same KMS key. The data and keys must be stored in each region.
What solution can meet these requirements with minimal operational overhead?
① A. Create an S3 bucket in each region. Configure the S3 bucket to use server-side encryption with Amazon S3-managed encryption keys (SSE-S3). Configure replication between the S3 buckets.
② B. Create a customer-managed multi-region KMS key. Create an S3 bucket in each region. Configure replication between the S3 buckets. Configure your application to use the KMS key with client-side encryption.
③ C. Create a customer-managed KMS key and an S3 bucket in each region. Configure the S3 bucket to use server-side encryption with Amazon S3-managed encryption keys (SSE-S3). Configure replication between the S3 buckets.
④ D. Create a customer-managed KMS key and an S3 bucket in each region. Configure the S3 bucket to use server-side encryption with an AWS KMS key (SSE-KMS). Configure replication between the S3 buckets.

  최소한의 운영 오버헤드로 이러한 요구 사항을 충족하는 솔루션은 고객 관리형 다중 리전 KMS 키를 생성하는 것입니다. 각 리전에 S3 버کم을 생성하고 S3 버킷 간의 복제를 구성합니다. 클라이언트 측 암호화와 함께 KMS 키를 사용하도록 애플리케이션을 구성하면 두 리전의 S3 버킷에 저장된 모든 데이터를 동일한 KMS 키로 암호화하고 해독할 수 있습니다. 데이터와 키는 두 리전 각각에 저장되며 운영 오버헤드가 최소화됩니다.

  정답은 2번입니다. 이 솔루션이 주어진 요구 사항을 가장 효과적으로 충족하기 때문입니다.

  키 관리 및 일관성: 고객 관리형 다중 리전 KMS 키를 사용하면, 두 리전 간에 동일한 키를 공유하고 관리할 수 있어 데이터 암호화와 해독 과정에서 일관성을 유지할 수 있습니다.
  데이터 보안: 애플리케이션 측에서 KMS 키를 직접 사용하여 암호화를 수행하면, 서버 측 암호화 방식에 비해 보다 세밀한 제어와 보안 정책 적용이 가능합니다.
  지역별 키 저장: 각 리전에 KMS 키를 생성하고 관리하면, 데이터와 키의 지역 분산 저장 요구 사항을 만족합니다.
  최소 오버헤드: S3 버킷 간 복제를 구성하면 데이터의 동기화를 자동화할 수 있어 운영 오버헤드를 최소화합니다. 단, 여기서 중요한 점은 SSE-KMS (옵션 D와 유사하지만, 옵션 D는 각 리전별로 별도의 KMS 키를 생성하는 것을 암시할 수 있어 일관성과 관리 측면에서 2번이 더 적합하다는 점입니다).
  따라서, 2번 옵션이 데이터 암호화, 키 관리, 지역 분산 저장, 그리고 운영 효율성을 모두 만족하면서 최소한의 오버헤드를 유지하는 최적의 해결책입니다.

37. A company recently launched a variety of new workloads on Amazon EC2 instances in its AWS account. The company needs to develop a strategy for remotely and securely accessing and managing these instances. The company needs to implement repeatable processes that work with native AWS services and adhere to the AWS Well-Architected Framework.
What solution meets these requirements with minimal operational overhead?
① A. Use the EC2 Serial Console to directly access the terminal interface of each instance for management purposes.
② B. Associate the appropriate IAM role with each existing and new instance. Use AWS Systems Manager Session Manager to establish a remote SSH session.
③ C. Generate a management SSH key pair. Load the public key onto each EC2 instance. Deploy a bastion host in the public subnet to provide a tunnel for managing each instance.
④ D. Establish an AWS Site-to-Site VPN connection. Instruct your administrator to connect directly to the instance using a local on-premises machine using SSH keys over the VPN tunnel.

  회사는 다양한 워크로드를 시작한 Amazon EC2 인스턴스에 원격으로 안전하게 액세스하고 관리하기 위한 전략을 수립해야 합니다. 최소한의 운영 오버헤드로 이러한 요구 사항을 충족하는 솔루션은 기존 인스턴스와 새 인스턴스에 적절한 IAM 역할을 연결하고 AWS 시스템 관리자 세션 관리자를 사용하여 원격 SSH 세션을 설정하는 것입니다.

  이러한 접근 방식은 AWS Well-Architected 프레임워크와 기본 AWS 서비스와 작동하여 반복 가능한 프로세스를 구현합니다. 이를 통해 회사는 인스턴스에 안전하고 효율적으로 액세스하고 관리할 수 있습니다.

  이러한 방법은 인스턴스에 대한 보안 액세스를 제공하고 최소한의 운영 오버헤드를 유지하면서 관리를 단순화합니다.

  정답은 2번입니다.

  회사가 요구하는 솔루션은 원격 안전 액세스와 관리를 최소 운영 오버헤드로 수행하는 것입니다.

  1번은 EC2 직렬 콘솔은 비상 상황에 유용하지만 일반적인 관리에는 불편하고 보안 관리가 제한적입니다. 3번의 배스천 호스트 접근법은 추가적인 인프라 관리와 보안 위험을 수반하여 오버헤드가 증가합니다. 4번은 온프레미스를 통한 연결로 원격 관리의 편의성과 클라우드 네이티브 접근을 저하시킵니다.

  따라서 2번이 최적입니다. 적절한 IAM 역할을 통해 권한 관리를 정밀하게 조정하고, AWS Session Manager를 사용하면 SSH를 통해 안전하게 원격 관리가 가능합니다. 이 방법은 중앙 집중식 관리 인터페이스를 제공하여 반복 가능한 프로세스를 구축하고 AWS Well-Architected 프레임워크의 운영 효율성과 보안 원칙을 준수합니다. 별도의 하드웨어 또는 복잡한 네트워크 설정 없이 AWS 내에서 해결되므로 운영 오버헤드를 최소화합니다.

38. A company hosts a static website on Amazon S3 and uses Amazon Route 53 for DNS. This website is experiencing growing demand worldwide. The company needs to reduce latency for users accessing the website. What is the most cost-effective solution to meet these requirements?
① A. Replicate the S3 bucket containing your website to all AWS Regions. Add a Route 53 geolocation routing entry.
② B. Provision an accelerator in AWS Global Accelerator. Associate the provided IP address with an S3 bucket. Edit the Route 53 entry to point to the accelerator's IP address.
③ C. Add an Amazon CloudFront distribution in front of the S3 bucket. Edit the Route 53 entry to point to the CloudFront distribution.
④ D. Enable S3 Transfer Acceleration on the bucket. Edit the Route 53 entry to point to the new endpoint.

  이 문제는 웹사이트의 지연 시간을 줄이는 가장 비용 효율적인 솔루션을 찾는 것입니다.olleyuted 솔루션은 Amazon CloudFront를 사용하는 것입니다. CloudFront는 콘텐츠 전송 네트워크로, 사용자에게 더 가까운 에지 로케이션에 캐시된 콘텐츠를 제공하여 지연 시간을 줄입니다.

  S3 버킷 앞에 CloudFront 배포판을 추가하면 CloudFront가 사용자와 가장 가까운 에지 로케이션에서 콘텐츠를 제공할 수 있습니다. 이렇게 하면 지연 시간이 줄어들고 사용자 경험을 개선할 수 있습니다. 또한 CloudFront는 자동으로 콘텐츠가 변경될 때마다에지 로케이션을 업데이트 해줍니다.

  이 솔루션은 다른 옵션보다 비용 효율적입니다. S3 버킷을 모든 리전에 복제하는 것은 많은 데이터가 복제될 수 있으므로 비용이 많이 듭니다. AWS Global Accelerator를 사용하는 것은 지연 시간을 줄이는 데 도움이 될 수 있지만, CloudFront보다 비용이 더 많이 들 수 있습니다. S3 Transfer Acceleration을 활성화하는 것은 보다 간단한 솔루션입니다. 하지만 CloudFront만큼 지연 시간을 줄이는 데 효과적이지 않을 수 있습니다.

  따라서 가장 비용 효율적인 솔루션은 S3 버킷 앞에 Amazon CloudFront 배포판을 추가하는 것입니다. Route 53 항목을 편집하여 CloudFront 배포를 가리키도록 하면 사용자는 콘텐츠를 더 빠르게 로드할 수 있습니다.

  회사가 전 세계적인 사용자 지연 시간을 최소화하면서 비용 효율성을 추구하는 상황에서 가장 적합한 해결책은 C 옵션입니다.

  Amazon CloudFront를 사용하면 (옵션 C), 콘텐츠의 캐싱과 엣지 로케이션 네트워크를 통해 전 세계 어디서든 사용자에게 빠르게 콘텐츠를 제공할 수 있습니다. CloudFront는 데이터를 가장 가까운 엣지 로케이션에서 제공함으로써 지연 시간을 크게 줄입니다. 또한, 기본적으로 S3 버킷과 원활하게 통합되어 설정이 비교적 간편하며, Route 53을 통해 도메인 이름을 CloudFront 배포로 리디렉션하면 글로벌 트래픽 관리와 높은 성능을 동시에 달성할 수 있습니다.

  다른 옵션들에 비해:

  A 옵션의 리전별 S3 버킷 복제는 복잡하고 비용이 많이 들며 관리도 어려울 수 있습니다.
  B 옵션의 AWS Global Accelerator는 주로 애플리케이션 엔드포인트(예: ALB, ELB)에 더 적합하고, S3 버킷과 직접 연결하는 것은 일반적이지 않습니다.
  D 옵션의 S3 Transfer Acceleration은 주로 데이터 업로드/다운로드 속도를 개선하는데 초점을 맞추며, 전역적인 콘텐츠 제공 성능 향상에는 한계가 있습니다.
  따라서, C 옵션인 CloudFront 배포판을 활용하는 것이 성능 향상과 비용 효율성 측면에서 최적의 선택입니다.

39. A company maintains a searchable database of items on its website. The data is stored in an Amazon RDS for MySQL database table containing over ten million rows. The database has 2 TB of general-purpose SSD storage. Millions of updates are made to this data daily through the company website.
The company has observed that some insert operations take more than 10 seconds. The company has determined that the database storage performance is the issue.
What is the solution to this performance issue?
① A. Change the storage type to Provisioned IOPS SSD.
② B. Change the DB instance to a memory-optimized instance class.
③ C. Change the DB instance to a performance-scalable instance class.
④ D. Enable multi-AZ RDS read replicas with MySQL native asynchronous replication.

  이 문제의 정답은 1번, 스토리지 유형을 프로비저닝된 IOPS SSD로 변경하는 것입니다.

  회사에서 매일 수백만 개의 업데이트가 발생하고, 일부 삽입 작업이 10초 이상 걸리며, 데이터베이스 스토리지 성능이 문제라고 판단했습니다. 이러한 성능 문제를 해결하기 위해 스토리지 유형을 프로비저닝된 IOPS SSD로 변경하면, 데이터베이스의 입출력 작업에 대해 더 높은 성능을 제공할 수 있습니다.

  범용 SSD는 일반적인 사용 사례에 적합하지만, 고성능 입출력이 필요한 경우 프로비저닝된 IOPS SSD가 더 적합합니다. 因此, 스토리지 유형을 프로비저닝된 IOPS SSD로 변경하면 데이터베이스의 성능 문제를 해결할 수 있습니다.

  다른 선택지는 성능 문제를 해결하는데 직접적으로帮助하지 않습니다. DB 인스턴스를 메모리 최적화 인스턴스 클래스나 성능 확장이 가능한 인스턴스 클래스로 변경하는 것은 CPU나 메모리 성능을 개선할 수 있지만, 스토리지 관련된 성능 문제에는 직접적으로 도움이 되지 않습니다. MySQL 기본 비동기식 복제를 통해 다중 AZ RDS 읽기 전용 복제본을 활성화하는 것은 데이터베이스의 가용성과冗余성을 개선할 수 있지만, 성능 문제를 직접적으로 해결하지는 않습니다.

  데이터베이스 성능 저하의 주요 원인은 삽입 작업의 지연 시간 증가로, 이는 주로 스토리지 I/O 성능의 한계 때문일 가능성이 높습니다.

  1번 옵션인 프로비저닝된 IOPS SSD로 스토리지 유형을 변경하는 것이 가장 효과적인 해결책입니다. 프로비저닝된 IOPS SSD는 예측 가능하고 일관된 고성능 I/O를 제공하여, 특히 빈번한 업데이트와 삽입 작업이 많은 환경에서 응답 시간을 크게 개선할 수 있습니다.

  2번 옵션의 메모리 최적화 인스턴스는 주로 메모리 집약적인 워크로드에 유리하지만, 이 문제의 핵심은 I/O 성능에 있습니다.
  3번 옵션의 성능 확장 가능 인스턴스는 필요에 따라 스케일링이 가능하나, 기본적인 I/O 성능 향상에는 한계가 있을 수 있습니다.
  4번 옵션의 읽기 복제본 활성화는 읽기 부하 분산에 효과적이나, 삽입 작업의 성능 문제 해결에는 직접적인 영향을 미치지 않습니다.
  따라서, 일관된 고성능 I/O가 요구되는 상황에서는 프로비저닝된 IOPS SSD로의 변경이 가장 적합한 해결책입니다.

40. A company has thousands of edge devices that collectively generate 1 TB of health alerts daily. Each alert is approximately 2 KB in size. A solution architect needs to implement a solution that collects and stores these alerts for future analysis.
The company wants a highly available solution. However, it needs to minimize costs and doesn't want to manage additional infrastructure. Furthermore, the company wants to retain data for 14 days for immediate analysis and archive all data older than 14 days.
What is the most operationally efficient solution that meets these requirements?
① A. Create an Amazon Kinesis Data Firehose delivery stream to collect notifications. Configure the Kinesis Data Firehose stream to deliver notifications to an Amazon S3 bucket. Set the S3 lifecycle configuration to transition data to Amazon S3 Glacier after 14 days.
② B. Launch Amazon EC2 instances in two Availability Zones and place them behind an Elastic Load Balancer to collect notifications. Create a script for the EC2 instances to store the notifications in an Amazon S3 bucket. Set the S3 lifecycle configuration to transition data to Amazon S3 Glacier after 14 days.
③ C. Create an Amazon Kinesis Data Firehose delivery stream to collect notifications. Configure the Kinesis Data Firehose stream to deliver notifications to an Amazon OpenSearch Service (Amazon Elasticsearch Service) cluster. Configure the Amazon OpenSearch Service (Amazon Elasticsearch Service) cluster to create daily manual snapshots and delete data from the cluster older than 14 days.
④ D. Create an Amazon Simple Queue Service (Amazon SQS) standard queue to collect notifications and set the message retention period to 14 days. Configure a consumer to poll the SQS queue, check the message age, and analyze the message data as needed. When a message is 14 days old, the consumer should copy the message to an Amazon S3 bucket and delete it from the SQS queue.

  해당 회사의 요구 사항을 고려하면 1번이 가장 적합합니다. 회사는 매일 1TB의 상태 경고를 생성하는 수천 개의 엣지 장치로부터 경고를 수집하고 저장해야 하며, 14일 동안의 데이터를 보관하고 14일보다 오래된 데이터를 보관해야 합니다. 또한 가용성이 높은 솔루션을 원하며 비용을 최소화하고 추가 인프라를 관리하고 싶지 않습니다.

  1번의 솔루션은 Amazon Kinesis Data Firehose를 사용하여 경고를 수집하고 Amazon S3 버킷에 전달합니다. 이후 14일 후에 데이터를 Amazon S3 Glacier로 전환하도록 S3 수명 주기 구성을 설정하여 비용을 최소화하고 데이터를 장기적으로 보관할 수 있습니다. 이 솔루션은 가용성이 높고, 추가 인프라를 관리할 필요가 없으며, 비용을 효율적으로 관리할 수 있습니다.

  반면에 다른 옵션은 다음과 같은 이유로 적합하지 않습니다.

  2번은 EC2 인스턴스를 관리해야 하므로 추가 인프라를 관리할 필요가 있습니다.
  3번은 수동 스냅샷을 생성하고 데이터를 삭제해야 하므로 운영 효율성이 떨어집니다.
  4번은 메시지 보존 기간을 설정하고 소비자를 구성해야 하므로 복잡하고 비용이 많이 들 수 있습니다.
  따라서 1번이 가장 운영 효율적인 솔루션입니다.

  정답은 1번입니다.

  Amazon Kinesis Data Firehose는 실시간 데이터 스트리밍과 자동화된 데이터 전송에 최적화되어 있어, 수많은 엣지 장치에서 발생하는 경고 데이터를 효율적으로 수집하고 처리할 수 있습니다. Kinesis Data Firehose를 통해 데이터를 Amazon S3 버킷에 직접 전송하면, 추가적인 인프라 관리 부담을 최소화할 수 있습니다.

  S3의 수명 주기 정책을 활용해 14일 이내의 데이터는 일반 저장소에 유지하고, 그 이후의 데이터는 비용 효율적인 S3 Glacier로 자동 전환함으로써, 데이터 보관 기간 요구 사항을 충족하면서도 비용을 절감할 수 있습니다. 이 솔루션은 높은 가용성을 기본으로 제공하며, 관리의 복잡성을 줄이고 필요한 분석 준비 상태를 유지하는 데 가장 적합합니다.

  다른 옵션들과 비교했을 때, EC2 인스턴스를 사용하는 B 옵션은 관리 부담이 크고 비용 효율성이 떨어지며, OpenSearch Service를 사용하는 C 옵션은 분석에 더 초점을 맞추지만 수동 관리가 필요하고 자동화 수준이 낮습니다. SQS를 활용하는 D 옵션은 메시지 관리에 복잡성이 추가되고, 데이터의 자동화된 장기 보관 전략이 덜 효과적입니다. 따라서, 요구 사항을 가장 잘 충족하는 운영 효율적인 솔루션은 1번입니다.

41. The company's application integrates with multiple Software-as-a-Service (SaaS) sources for data collection. The company runs an Amazon EC2 instance to receive the data and upload it to an Amazon S3 bucket for analysis. The same EC2 instance that receives and uploads the data also notifies the user when the upload is complete. The company is noticing slow application performance and wants to improve performance as much as possible.
What solution can meet these requirements with minimal operational overhead?
① A. Create an Auto Scaling group to allow EC2 instances to scale. Configure S3 event notifications to send events to an Amazon Simple Notification Service (Amazon SNS) topic when uploads to the S3 bucket are complete.
② B. Create an Amazon AppFlow flow that transfers data between each SaaS source and the S3 bucket. Configure an S3 event notification to send an event to an Amazon Simple Notification Service (Amazon SNS) topic when the upload to the S3 bucket is complete.
③ C. Create an Amazon EventBridge (Amazon CloudWatch Events) rule for each SaaS source to send output data. Configure an S3 bucket as the target of the rule. Create a second EventBridge (CloudWatch Events) rule to send an event when the upload to the S3 bucket is complete. Configure an Amazon Simple Notification Service (Amazon SNS) topic as the target of the second rule.
④ D. Create a Docker container to use instead of an EC2 instance. Host the containerized application on Amazon Elastic Container Service (Amazon ECS). Configure Amazon CloudWatch Container Insights to send an event to an Amazon Simple Notification Service (Amazon SNS) topic when the upload to the S3 bucket is complete.

  이 문제의 해설은 다음과 같다.

  각 SaaS 소스와 Amazon S3 버킷 간에 데이터를 전송하는 Amazon AppFlow 흐름을 생성하면 EC2 인스턴스가 처리하던 일을 대신할 수 있다. 또한 S3 버킷에 대한 업로드가 완료되면 Amazon Simple 알림 서비스(Amazon SNS) 주제로 이벤트를 보내도록 S3 이벤트 알림을 구성할 수 있다. 이렇게 하면 EC2 인스턴스의 운영오버헤드를 최소화할 수 있으며, 성능 개선에도 도움이 된다.

  Amazon AppFlow는 AWS에서 제공하는 서비스로, 다양한 소스에서 데이터를 수집하고 처리할 수 있다. Amazon SNS는 메시지 큐를 사용하여 알림을 보내는 서비스로, 사용자에게 알림을 보낼 때 유용하게 사용될 수 있다. 따라서 이러한 서비스들을 사용하면 최소한의 운영 오버헤드로 성능을 개선할 수 있다.

  따라서 이 문제의 정답은 2번이다.

  정답은 2번입니다.

  현재 설정에서 EC2 인스턴스가 데이터 수집, 분석 및 알림 발송을 모두 처리하는 구조는 병목 현상을 일으키기 쉽습니다. 성능 개선을 위해 핵심은 작업 분담과 자동화입니다.

  2번 선택지는 이를 효과적으로 해결합니다.

  Amazon AppFlow: SaaS 소스와 S3 버킷 간 데이터 이동을 자동화하여 EC2 인스턴스 부담을 줄입니다. AppFlow는 데이터 전송을 효율적으로 처리하며 별도의 코드 개발 없이 통합 가능합니다.
  S3 이벤트 알림 + Amazon SNS: S3 버킷에 데이터 업로드 완료 시 자동으로 알림을 생성하여 사용자에게 전달합니다. 이는 EC2 인스턴스의 추가 작업 없이 알림 기능을 유지합니다.
  다른 선택지의 문제점:

  1번: EC2 인스턴스 확장만으로는 근본적인 병목 문제 해결에 한계가 있습니다.
  3번: EventBridge 규칙을 두 번 구성하는 것은 복잡성을 증가시키고 관리 부담을 높입니다.
  4번: 컨테이너화는 장점이 있지만, 본 문제의 핵심인 데이터 처리 자동화와 알림 기능 구현에 비해 과도한 접근입니다.

42. A company runs a highly available image processing application on Amazon EC2 instances in a single VPC. The EC2 instances run within multiple subnets across multiple Availability Zones. The EC2 instances do not communicate with each other. However, the EC2 instances download images from Amazon S3 and upload them to Amazon S3 through a single NAT gateway. The company is concerned about data transfer charges.
What is the most cost-effective way for the company to avoid regional data transfer charges?
① A. Start a NAT gateway in each availability zone.
② B. Replace the NAT gateway with a NAT instance.
③ C. Deploy a gateway VPC endpoint for Amazon S3.
④ D. Provision an EC2 dedicated host to run EC2 instances.

  회사가 지역 간 데이터 전송 요금을 피할 수 있는 가장 비용 효율적인 방법은 Amazon S3용 게이트웨이 VPC 엔드포인트를 배포하는 것입니다. 이 방법을 사용하면 EC2 인스턴스에서 Amazon S3로의 아웃바운드 트래픽이 VPC 내에서 직접 라우팅되므로 지역 간 데이터 전송 요금이 부과되지 않습니다. 게이트웨이 VPC 엔드포인트는 VPC와 Amazon S3 간에 비공개ネットワーク 연결을 제공하여 보안과 비용 효율성을 제공합니다. 이 방법은 NAT 게이트웨이와 같은 추가 인프라를 프로비저닝할 필요가 없기 때문에 비용 효율적입니다. 따라서 회사는 Amazon S3용 게이트웨이 VPC 엔드포인트를 배포하여 지역 간 데이터 전송 요금을 피할 수 있습니다.

  정답은 3번, Amazon S3용 게이트웨이 VPC 엔드포인트 배포입니다.

  현재 설정에서는 EC2 인스턴스가 외부 인터넷을 통해 S3에 접근하므로 지역 간 데이터 전송 요금이 발생합니다. 게이트웨이 VPC 엔드포인트를 사용하면 VPC 내부에서 S3에 직접 접근할 수 있게 되어, 이러한 외부 트래픽을 내부 네트워크 내에서 처리함으로써 지역 간 데이터 전송 요금을 피할 수 있습니다.

  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  1번: 각 가용 영역에 NAT 게이트웨이를 추가하면 관리 복잡성만 증가하고, 기본적인 지역 간 데이터 전송 문제는 해결되지 않습니다.
  2번: NAT 인스턴스는 NAT 게이트웨이와 유사하게 작동하며, 여전히 외부 트래픽을 통해 S3에 접근하므로 데이터 전송 요금 문제는 해결되지 않습니다.
  4번: EC2 전용 호스트는 인스턴스 프로비저닝 모델의 변화일 뿐, 데이터 전송 경로나 요금 구조를 직접적으로 개선하지 않습니다.
  따라서, 가장 비용 효율적인 해결책은 Amazon S3용 게이트웨이 VPC 엔드포인트를 배포하는 것입니다.

43. A company has an on-premises application that generates a large amount of time-sensitive data that is backed up to Amazon S3. The application has grown, and users are complaining about limited internet bandwidth. A solutions architect needs to design a long-term solution that can provide timely backups to Amazon S3 while minimizing the impact on internal users' internet connections.
What solution meets these requirements?
① A. Establish an AWS VPN connection and proxy all traffic through a VPC gateway endpoint.
② B. Establish a new AWS Direct Connect connection and route backup traffic over this new connection.
③ C. Order AWS Snowball devices daily. Load data onto the Snowball devices and return the devices to AWS daily.
④ D. Submit a support ticket through the AWS Management Console. Request that the S3 service limits be removed from your account.

  대량의 데이터를 생성하는 온프레미스 애플리케이션의 성장으로 인해 인터넷 대역폭 제한에 대한 사용자 불만이 증가하고 있습니다. 이 문제를 해결하기 위해 Amazon S3에 데이터를 백업하는 데 소요되는 시간을 줄이고 내부 사용자의 인터넷 연결에 미치는 영향을 최소화할 수 있는 장기적인 솔루션을 설계해야 합니다.

  이를 해결하는 방법은 AWS Direct Connect 연결을 설정하는 것입니다. AWS Direct Connect는 온프레미스 네트워크와 AWS 간에 전용 네트워크 연결을 제공하여 인터넷을 통해 데이터를 전송할 때 발생하는 대기 시간과 지연을 줄입니다. 이 연결을 통해 백업 트래픽을 전달하면 인터넷 대역폭 제한으로 인한 속도 저하를 피할 수 있습니다.

  AWS Direct Connect를 사용하면 대량의 데이터를 빠르게 전송할 수 있어 백업 시간을 단축할 수 있습니다. 또한, 전용 네트워크 연결을 통해 데이터 전송의 안정性과 보안성을 높일 수 있습니다. 따라서, 이러한 요구 사항을 충족하는 솔루션은 새로운 AWS Direct Connect 연결을 설정하고 이 새로운 연결을 통해 백업 트래픽을 전달하는 것입니다.

  정답은 2번입니다.

  인터넷 대역폭 제한을 극복하고 시간 민감 데이터의 시기적절한 백업을 위해 AWS Direct Connect가 최적의 솔루션입니다.

  직접 연결: AWS Direct Connect는 회사 네트워크와 AWS 클라우드를 직접 연결하는 전용 회로를 제공하여 인터넷 트래픽을 우회합니다. 이로써 백업 데이터 전송 속도가 향상되고 인터넷 대역폭 부담이 크게 줄어듭니다.
  안정성 및 신뢰성: Direct Connect는 높은 안정성과 신뢰성을 보장하며 중요한 데이터 백업에 필요한 일관성을 제공합니다.
  다른 옵션들은 적합하지 않습니다.

  1번: VPN은 데이터 암호화를 제공하지만 대역폭 제약을 해결하지 못합니다. VPC 게이트웨이 엔드포인트는 추가적인 네트워크 오버헤드를 발생시킬 수 있습니다.
  3번: Snowball은 대용량 데이터 이동에 유용하지만, 매일 데이터를 전송해야 하는 이 상황에서는 비효율적이고 시간이 오래 걸립니다.
  4번: S3 서비스 제한 제거 요청은 문제의 근본적인 해결책이 아닙니다. 대역폭 문제는 여전히 남아있습니다.

44. A company has an Amazon S3 bucket containing sensitive data. The company needs to protect the data from accidental deletion. To meet this requirement, what combination of steps should a solution architect perform? (Choose two.)
① A. Enable versioning on your S3 bucket.
② B. Enable MFA deletion on the S3 bucket.
③ C. Create a bucket policy on the S3 bucket.
④ D. Enable default encryption on your S3 bucket.
⑤ E. Create a lifecycle policy for objects in your S3 bucket.

  S3 버킷에서 중요한 데이터가 실수로 삭제되지 않도록 보호하는 방법은 버전 관리와 MFA 삭제를 활성화하는 것입니다. 버전 관리를 활성화하면 객체의 이전 버전을 유지하여 실수로 삭제하거나 덮어쓰기한 경우에도 데이터를 복원할 수 있습니다. MFA 삭제를 활성화하면 사용자가 MFA를 사용하여 삭제를 확인해야 하므로 실수로 데이터를 삭제하는 것을 방지할 수 있습니다. 따라서 올바른 방법은 S3 버킷에서 버전 관리와 MFA 삭제를 활성화하는 것입니다.

  주어진 문제에서 요구하는 주요 목표는 데이터의 실수 삭제를 방지하는 것입니다. 이를 위해 가장 효과적인 두 가지 방법을 선택해야 합니다.

  MFA 삭제 활성화 (B): 이 옵션은 Multi-Factor Authentication을 삭제 작업에 요구함으로써, 중요한 데이터 삭제를 실수로부터 보호합니다. 삭제를 시도할 때 추가 인증 단계가 필요하므로, 의도치 않은 삭제를 방지하는 데 매우 효과적입니다.

  버전 관리 활성화 (A): 또한 버전 관리 활성화(A)는 데이터 삭제에 대한 보호 측면에서도 유용합니다. 이 기능을 통해 이전 버전의 객체를 복원할 수 있어, 실수로 삭제된 데이터를 복구하는 데 도움이 됩니다. 하지만 문제에서 정답으로 명시된 것은 MFA 삭제 활성화입니다.

  따라서 주어진 정답에 따르면, MFA 삭제 활성화 (B)가 가장 핵심적인 보호 수단으로 선택되었습니다. 버킷 정책 생성(C)이나 기본 암호화(D), 수명 주기 정책(E)은 데이터 보호와 관련이 있지만, 특히 실수로 인한 삭제 방지에 직접적인 효과가 덜한 편입니다. 따라서 문제의 정답은 B 옵션이 핵심적입니다. 다만, 실제 설계에서는 B와 함께 A를 병행하는 것이 이상적일 수 있음을 덧붙일 수 있습니다.

45. A company has a data collection workflow consisting of:
• An Amazon Simple Notification Service (Amazon SNS) topic for notifications of new data deliveries
• An AWS Lambda function that processes the data and records metadata
The company observes that the collection workflow fails, sometimes due to network connectivity issues. When these failures occur, the Lambda function does not collect the data unless the company manually reruns the task.
What combination of actions should the solutions architect take to ensure that the Lambda function collects all data in the future? (Choose two.)
① A. Deploy Lambda functions across multiple Availability Zones.
② B. Create an Amazon Simple Queue Service (Amazon SQS) queue and subscribe to the SNS topic.
③ C. Increase the CPU and memory allocated to your Lambda function.
④ D. Increase the provisioned throughput of your Lambda function.
⑤ E. Modify the Lambda function to read from an Amazon Simple Queue Service (Amazon SQS) queue.

  네트워크 연결 문제로 인한 데이터 수집 실패를 해결하기 위해 솔루션 아키텍트는 다음과 같은 조치를 취해야 합니다:

  E. Amazon Simple Queue Service(Amazon SQS) 대기열에서 읽도록 Lambda 함수를 수정합니다.
  SQS를 사용하면 데이터 전송이 일시적으로 실패하더라도 메시지가 대기열에 저장되어 재시도가 가능합니다. 이로 인해 네트워크 문제 후에도 데이터 처리가 자동으로 재개될 수 있습니다.

  B. Amazon Simple Queue Service(Amazon SQS) 대기열을 생성하고 SNS 주제를 구독합니다.
  SNS 주제를 SQS 대기열에 연결하면, 데이터 전송 알림이 대기열에 안정적으로 전달됩니다. 이렇게 하면 Lambda 함수는 대기열에서 메시지를 가져와 처리할 수 있어, 일시적인 네트워크 문제에도 불구하고 데이터 손실 없이 작업을 재시도할 수 있습니다.

  이 두 가지 조치는 데이터의 안정적인 저장 및 재처리를 보장하여 수동 재시작 없이도 워크플로의 신뢰성을 크게 향상시킵니다. 다른 옵션들은 성능 향상이나 가용성 증대에 초점을 맞추고 있지만, 주어진 문제의 핵심인 재시도 메커니즘 구축에는 적합하지 않습니다.

46. ​​A company has an application that provides marketing services to stores. This service is based on previous purchases by store customers. Stores upload transaction data to the company via SFTP, which processes and analyzes the data to generate new marketing offers. Some files can exceed 200 GB in size.
Recently, the company discovered that some stores uploaded files containing personally identifiable information (PII) that should not be included. The company wants to notify administrators when PII is shared again. The company also wants to automate troubleshooting.
What should a solution architect do to meet these requirements with minimal development effort?
① A. Use an Amazon S3 bucket as a secure transfer point. Use Amazon Inspector to scan objects in the bucket. If an object contains PII, trigger an S3 lifecycle policy to remove the object.
② B. Use an Amazon S3 bucket as a secure transfer point. Scan objects in the bucket using Amazon Macie. If an object contains PII, use Amazon Simple Notification Service (Amazon SNS) to notify administrators to remove the object.
③ C. Implement a custom search algorithm in an AWS Lambda function. Trigger the function when an object is loaded into the bucket. If the object contains PII, use Amazon Simple Notification Service (Amazon SNS) to notify administrators to remove the object.
④ D. Implement a custom search algorithm in an AWS Lambda function. Trigger the function when an object is loaded into the bucket. If the object contains PII, use Amazon Simple Email Service (Amazon SES) to send a notification to an administrator and trigger an S3 lifecycle policy to remove the PII-containing file.

  이번 문제는 AWS 서비스를 사용하여 лич신 식별 정보(PII)가 포함된 파일을 업로드한 경우 관리자에게 알림을 보내고, 자동으로 문제를 해결하는 방법을 묻는 문제입니다. 관심 있는 사항은 개인 정보 보호와 자동화된 문제 해결입니다. Amazon S3를 안전한 전송 지점으로 사용하고, Amazon Macie를 사용하여 버킷의 객체를 스캔하여 PII를検出합니다. Amazon Macie는 AWS에서 제공하는 서비스로, S3 버킷의 객체를 스캔하여 PII를 검출하고, 알림을 보냅니다. 또한 Amazon Simple Notification Service(Amazon SNS)를 사용하여 관리자에게 PII가 포함된 객체를 제거하라는 알림을 보냅니다. 이 방법은 최소한의 개발 노력이 필요하여 적절한 솔루션입니다. 따라서 정답은 B입니다.

  정답은 B입니다.

  회사의 요구사항을 효율적으로 충족시키기 위해서는 최소한의 개발 노력과 자동화된 시스템이 필요합니다.

  Amazon S3를 보안 전송 지점으로 활용하여 데이터를 안전하게 저장합니다.
  Amazon Macie는 PII 감지에 특화된 서비스로, 복잡한 사용자 정의 알고리즘 개발 없이도 자동으로 민감한 데이터를 식별할 수 있습니다.
  Amazon SNS를 통해 감지된 PII 포함 객체에 대해 관리자에게 즉시 알림을 보내 관리 개입을 용이하게 합니다.
  옵션 1은 PII 감지 기능이 제한적이고 제거만을 통한 대응이 적절하지 않으며, 옵션 3과 4는 사용자 정의 알고리즘 구현을 요구해 개발 노력이 증가합니다. 특히 4번은 이메일 알림과 함께 데이터 삭제를 제안하지만, 기본적인 요구사항인 알림만으로도 자동화를 달성할 수 있어 B 옵션이 가장 적합합니다. 따라서 최소한의 개발 노력으로 요구사항을 효과적으로 충족시키는 가장 좋은 선택은 B입니다.

47. A company needs to guarantee Amazon EC2 capacity in three specific Availability Zones in a specific AWS Region for an event expected to last a week. How can the company guarantee EC2 capacity?
① A. Purchase a Reserved Instance specifying the required region.
② B. Create an on-demand capacity reservation specifying the required region.
③ C. Purchase a Reserved Instance specifying the required region and three Availability Zones.
④ D. Create an On-Demand Capacity Reservation specifying the required region and three Availability Zones.

  회사는 1주일 동안 지속될 예정인 이벤트를 위해 특정 AWS 지역의 3개 특정 가용 영역에서 Amazon EC2 용량을 보장해야 합니다. 이벤트 기간 동안 EC2 용량을 보장하려면 회사는 필요한 지역과 3개의 가용 영역을 지정하는 온디맨드 용량 예약을 생성해야 합니다. 이 방식으로 회사는 이벤트 기간 동안 필요한 EC2 용량을 보장 받을 수 있어 안정적으로 이벤트를 운영할 수 있습니다. 따라서 올바른 선택은 필요한 지역과 3개의 가용 영역을 지정하는 온디맨드 용량 예약을 생성하는 것입니다.

  정답은 4번입니다.

  회사가 특정 AWS 지역 내의 3개 가용 영역에서 EC2 인스턴스 용량을 확보하려면, 가용성과 안정성을 최대화하는 전략이 필요합니다. 온디맨드 용량 예약을 사용하면 미리 특정 지역과 그 내의 특정 가용 영역들에 대한 자원을 보장받을 수 있습니다.

  옵션 1과 3은 예약 인스턴스를 제안하지만, 예약 인스턴스는 EC2에 대한 장기적인 비용 절감과 예측 가능한 비용 관리에 더 초점을 맞춥니다. 또한, 가용 영역을 명시적으로 지정하는 기능이 제한적일 수 있습니다.
  옵션 2는 온디맨드 예약을 제안하지만, 가용 영역을 명시적으로 지정하는 기능이 부족합니다. 따라서 필요한 가용 영역들을 모두 커버하지 못할 위험이 있습니다.
  따라서, 4번 옵션처럼 필요한 AWS 지역과 그 안의 정확히 3개의 가용 영역을 지정하여 온디맨드 용량 예약을 생성하는 것이 최적입니다. 이렇게 하면 이벤트 기간 동안 필요한 모든 가용 영역에서 EC2 용량을 확보할 수 있습니다. 이 방법은 단기적인 요구 사항에 유연하게 대응하면서도 가용성을 높일 수 있습니다.

48. A company's website uses Amazon EC2 Instance Store for its item catalog. The company wants the catalog to be highly available and stored in a durable location. What should the solution architect do to meet these requirements?
① A. Migrate your catalog to Amazon ElastiCache for Redis.
② B. Deploy larger EC2 instances with larger instance stores.
③ C. Move the catalog from the instance store to Amazon S3 Glacier Deep Archive.
④ D. Move the catalog to an Amazon Elastic File System (Amazon EFS) file system.

  솔루션 설계자는 카탈로그의 가용성이 높고 내구성이 있도록 저장할 수 있는 방법을 찾아야 합니다. 인스턴스 스토어는 데이터가 EC2 인스턴스와 함께 삭제되므로 내구성 안정성이 낮은 편입니다. Amazon S3 Glacier Deep Archive는 낮은 비용으로 데이터를 보관할 수 있지만, 데이터에 빠른 접근이 필요할 경우 부적합합니다. Redis용 Amazon ElastiCache는 캐싱을 목적으로 하는 서비스로, 데이터의 내구성 유지를 위한 서비스가 아닙니다. 따라서 데이터 내구성과 가용성이 높고file 시스템으로 데이터에 접근할 수 있는 Amazon Elastic File System(Amazon EFS)로 카탈로그를 이동시키는 것이 적절합니다.

  정답은 4번입니다.

  회사의 요구사항은 카탈로그의 높은 가용성과 내구성입니다.

  EC2 인스턴스 스토어는 인스턴스에 종속적이기 때문에 인스턴스 장애 시 데이터 손실 위험이 있습니다. 따라서 가용성과 내구성 면에서 부족합니다.

  Amazon S3 Glacier Deep Archive (보기 C)는 매우 저렴한 장기 보관 서비스로, 자주 접근하는 카탈로그 데이터에 적합하지 않습니다.

  Redis용 Amazon ElastiCache (보기 A)는 캐싱 솔루션으로 데이터의 일관성 유지에 유리하지만, 기본 저장소로서의 내구성과 가용성 보장에는 한계가 있습니다.

  Amazon Elastic File System (Amazon EFS) (보기 D)는 여러 EC2 인스턴스가 공유할 수 있는 확장 가능하고 내구성이 뛰어난 파일 시스템입니다. EFS는 인스턴스 장애에 강하고, 데이터 복제를 통해 높은 가용성을 제공합니다. 따라서 카탈로그 데이터의 요구 사항을 가장 잘 충족합니다.

49. The company stores call log files monthly. Within a year of a call, users access the files randomly, but after that, they access them infrequently. The company wants to optimize its solution by providing users with the ability to query and retrieve files less than a year old as quickly as possible. Delays in retrieving older files are acceptable.
What solution meets these requirements most cost-effectively?
① A. Amazon S3 Glacier stores individual files with tags for instant retrieval. You can query tags to retrieve files in S3 Glacier Instant Retrieval.
② B. Store individual files in Amazon S3 Intelligent-Tiering. Use an S3 lifecycle policy to move files to S3 Glacier flexible retrieval after one year. Use Amazon Athena to query and search files in Amazon S3. Use S3 Glacier Select to query and search files in S3 Glacier.
③ C. Store individual files with tags in Amazon S3 Standard storage. Store retrieval metadata for each archive in Amazon S3 Standard storage. Use an S3 lifecycle policy to move files to S3 Glacier Instant Retrieval after one year. Retrieve metadata from Amazon S3 to query and retrieve files.
④ D. Store individual files in Amazon S3 Standard storage. Use an S3 lifecycle policy to move files to S3 Glacier Deep Archive after one year. Store retrieval metadata in Amazon RDS. Query files in Amazon RDS. Retrieve files from S3 Glacier Deep Archive.

  회사에서는 1년 미만의 파일을 빠르게 쿼리하고 검색할 수 있는 기능을 제공하고자 합니다. 1년이 지난 후에는 파일에 대한 액세스가 드물기 때문에 오래된 파일 검색이 지연되는 것은 허용됩니다. 이러한 요구 사항을 충족하는 가장 비용 효율적인 솔루션은 Amazon S3 Intelligent-Tiering을 사용하는 것입니다.

  Amazon S3 Intelligent-Tiering을 사용하여 개별 파일을 저장하면 파일에 대한 액세스 빈도에 따라 자동으로 스토리지 클래스를 변경할 수 있습니다. 1년이 지난 후에는 S3 수명 주기 정책을 사용하여 파일을 S3 Glacier 유연한 검색으로移動시켜 비용을 절감할 수 있습니다.

  Amazon Athena를 사용하여 Amazon S3에 있는 파일을 쿼리하고 검색할 수 있으며, S3 Glacier Select를 사용하여 S3 Glacier에 있는 파일을 쿼리하고 검색할 수 있습니다. таким образом, 이 솔루션은 회사要求 사항을 충족하면서 비용을 최적화합니다.

  정답은 B 옵션입니다.

  주요 요구사항은 다음과 같습니다:

  1년 이내의 파일은 빠르게 검색하고 쿼리할 수 있어야 함.
  1년 이후의 파일은 드물게 액세스되며 검색 지연이 허용됨.
  B 옵션은 다음과 같은 이점을 제공합니다:

  Amazon S3 Intelligent-Tiering: 자동으로 데이터를 비용 효율적인 저장소로 이동시켜, 최근에 자주 액세스되는 파일은 빠르게 접근 가능합니다.
  S3 수명 주기 정책: 1년 후 파일을 S3 Glacier Flexible Retrieval으로 이동하여 비용을 절감하면서도 필요할 때 비교적 빠르게 검색할 수 있습니다.
  Amazon Athena와 S3 Glacier Select: S3에 저장된 데이터를 SQL로 쿼리할 수 있어 빠른 분석이 가능합니다. 특히 S3 Glacier에 있는 파일도 Select를 통해 효율적으로 검색할 수 있습니다.
  이 조합은 빠른 액세스와 비용 효율성을 동시에 충족하며, 오래된 파일의 검색 지연을 허용하는 요구 사항에 부합합니다. 따라서 B 옵션이 가장 적합한 솔루션입니다.

50. A company has a production workload running on 1,000 Amazon EC2 Linux instances. The workload relies on third-party software. The company needs to patch the third-party software on all EC2 instances as quickly as possible to address critical security vulnerabilities. What should a solution architect do to meet this requirement?
① A. Create an AWS Lambda function that applies the patch to all EC2 instances.
② B. Configure AWS Systems Manager Patch Manager to apply patches to all EC2 instances.
③ C. Schedule an AWS Systems Manager maintenance window to apply patches to all EC2 instances.
④ D. Use AWS Systems Manager Run Command to run a custom command to apply patches to all EC2 instances.

  EC2 인스턴스에 패치를 적용하는 방법은 여러 가지가 있습니다. 그러나 가장 효율적이고 신속한 방법은 AWS 시스템 관리자 Run Command를 사용하는 것입니다. 이 방법을 사용하면 모든 EC2 인스턴스에 패치를 적용하는 사용자 지정 명령을 실행할 수 있습니다.

  AWS Lambda 함수를 생성하여 패치를 적용하는 방법은 복잡하고 시간이 소요될 수 있습니다. 또한 AWS 시스템 관리자 패치 관리자를 구성하여 패치를 자동으로 적용하는 방법은 일부 인스턴스에서 작동하지 않을 수 있습니다. 유지 관리 기간을 예약하여 패치를 적용하는 방법은 패치가 즉시 적용되지 않으므로 시간이 걸릴 수 있습니다.

  따라서, 모든 EC2 인스턴스에 패치를 빠르게 적용하기 위한 가장 좋은 방법은 AWS 시스템 관리자 Run Command를 사용하여 사용자 지정 명령을 실행하는 것입니다. 이 방법은 빠르고 효율적이며 모든 인스턴스에 패치를 신속하게 적용할 수 있습니다.

  정답은 4번입니다. AWS 시스템 관리자의 Run Command는 대규모 인스턴스 집합에 대한 명령어 실행을 효율적으로 관리하는 데 이상적입니다. 이 옵션을 사용하면 보안 패치를 포함한 사용자 정의 스크립트를 한 번에 여러 EC2 인스턴스에 배포하고 실행할 수 있습니다.

  1번의 AWS Lambda는 주로 이벤트 기반의 트리거 작업에 최적화되어 있어, 인스턴스 전체에 대한 일괄 패치 적용에는 적합하지 않습니다.

  2번의 AWS Systems Manager Patch Manager는 주로 Windows 인스턴스에 대한 패치 관리를 용이하게 하는데 특화되어 있으며, Linux 인스턴스에 대한 직접적이고 유연한 패치 적용에는 제한적일 수 있습니다.

  3번의 유지 관리 기간 설정은 인스턴스의 유지보수 작업을 스케줄링하는 데 유용하지만, 실제 패치 적용 명령을 자동화하거나 실행하는 기능은 제공하지 않습니다. 따라서, 신속하고 일괄적인 패치 적용이 요구되는 상황에서는 Run Command가 가장 적합한 솔루션입니다. Run Command를 통해 스크립트를 통해 패치를 자동화하고 일괄 적용할 수 있어, 1,000개의 인스턴스에 빠르게 보안 패치를 적용하는 데 효과적입니다.

51. A company is developing an application that provides order delivery statistics, retrievable via a REST API. The company wants to extract the delivery statistics, format the data in an easy-to-read HTML format, and send reports to multiple email addresses simultaneously every morning.
What combination of steps should the solution architect perform to meet these requirements? (Choose two.)
① A. Configure your application to send data to Amazon Kinesis Data Firehose.
② B. Format the data and email the report using Amazon Simple Email Service (Amazon SES).
③ C. Create an Amazon EventBridge (Amazon CloudWatch Events) scheduled event that calls an AWS Glue job to query your application's API for data.
④ D. Create an Amazon EventBridge (Amazon CloudWatch Events) scheduled event that calls an AWS Lambda function to query your application's API for data.
⑤ E. Store application data in Amazon S3. To send reports by email, create an Amazon Simple Notification Service (Amazon SNS) topic as the S3 event target.

  회사가 지정한 요구 사항을 충족하기 위해 솔루션 설계자는 매일 아침 동시에 여러 이메일 주소로 보고서를 보내야 합니다. 이를 달성하기 위해 설계자는 AWS Lambda 함수를 호출하여 애플리케이션의 API에 데이터를 쿼리하는 Amazon EventBridge 예약 이벤트를 생성해야 합니다.

  회사의 요구 사항을 충족하기 위해 가장 적합한 두 단계 조합은 다음과 같습니다:

  정답 선택: 4, B
  해설:
  4번 (D 변형): AWS Lambda 함수를 사용하여 매일 특정 시간에 예약된 EventBridge 트리거를 설정하고, 이 트리거는 Lambda 함수를 실행하여 REST API에서 필요한 배송 통계 데이터를 자동으로 추출할 수 있습니다. 이는 자동화된 데이터 수집에 이상적입니다.
  B번: 추출된 데이터를 읽기 쉬운 HTML 형식으로 구성한 후, Amazon SES를 통해 여러 이메일 주소로 보고서를 일괄적으로 전송할 수 있습니다. SES는 이메일 전송을 효율적이고 관리하기 쉽게 만들어줍니다.
  선택지 설명:

  A번 (Kinesis Data Firehose): 주로 실시간 데이터 스트리밍 및 분석에 초점을 맞추고 있어, 이 경우의 일괄 처리와 보고서 생성에는 적합하지 않습니다.
  C번 (AWS Glue와 EventBridge): AWS Glue는 주로 데이터 통합과 ETL 작업에 사용되므로, 이 시나리오의 직접적인 요구 사항에는 적합하지 않습니다.
  E번 (Amazon S3와 SNS): S3는 데이터 저장에 좋지만, 보고서 생성 및 직접 이메일 전송 과정에서 SNS를 통한 간접적인 이메일 전송은 복잡성을 추가할 수 있으며, 직접 SES를 사용하는 것보다 효율적이지 않습니다.
  따라서, 데이터 자동 추출과 이메일 보고서 전송을 가장 간결하고 효과적으로 처리하기 위해 4번과 B번이 최적의 조합입니다.

52. A company is looking to migrate its on-premises applications to AWS. The applications generate output files ranging in size from tens of gigabytes to hundreds of terabytes. The application data must be stored in a standard file system structure. The company wants a solution that scales automatically, is highly available, and requires minimal operational overhead.
What solution meets these requirements?
① A. Migrate your application to run as containers on Amazon Elastic Container Service (Amazon ECS). Use Amazon S3 for storage.
② B. Migrate your application to run as containers on Amazon Elastic Kubernetes Service (Amazon EKS). Use Amazon Elastic Block Store (Amazon EBS) for storage.
③ C. Migrate applications to Amazon EC2 instances in a multi-AZ Auto Scaling group. Use Amazon Elastic File System (Amazon EFS) for storage.
④ D. Migrate applications to Amazon EC2 instances in a multi-AZ Auto Scaling group. Use Amazon Elastic Block Store (Amazon EBS) for storage.

  정답은 3번입니다.

  이 솔루션은 다음과 같은 이유로 적합합니다:

  Amazon EC2 인스턴스와 다중 AZ Auto Scaling 그룹: 자동 확장 기능을 통해 애플리케이션의 부하 변동에 따라 자원을 동적으로 조정할 수 있습니다. 이는 최소한의 운영 오버헤드와 함께 효율적인 리소스 관리를 가능하게 합니다.

  Amazon EFS: 파일 시스템 구조를 유지해야 하는 요구 사항을 충족시키는 데 적합합니다. EFS는 여러 EC2 인스턴스 간에 공유 파일 시스템을 제공하며, 대용량 데이터 처리와 확장성에 강점을 가지고 있습니다. 특히, 다양한 크기의 출력 파일을 생성하는 애플리케이션에 적합한 수평 확장성을 제공합니다.

  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  Amazon S3 (1번): 대용량 파일의 직접적인 처리와 파일 시스템 구조 유지에 한계가 있습니다. 주로 객체 저장에 최적화되어 있어 애플리케이션 내에서 파일 시스템처럼 쉽게 접근하기 어렵습니다.

  Amazon EBS (2번, 4번): EBS는 블록 스토리지를 제공하므로 개별 인스턴스 내에서의 데이터 관리에는 유용하지만, 여러 인스턴스 간의 공유 파일 시스템 구조를 쉽게 지원하지 못합니다. 또한, EFS 대비 수평 확장성과 대규모 파일 공유에 제약이 있을 수 있습니다.

  따라서, 요구 사항인 자동 확장, 높은 가용성, 표준 파일 시스템 구조 유지, 그리고 운영 복잡성 최소화를 모두 만족시키는 최적의 선택은 3번입니다.

53. A company must store its accounting records in Amazon S3. Records must be readily accessible for one year and then retained for an additional nine years. No one in the company, including administrative users and root users, can delete the records for the entire ten-year period. Records must be stored with maximum resiliency. What solution meets these requirements?
① A. Store records in S3 Glacier for a full 10 years. Use access control policies to prevent records from being deleted for 10 years.
② B. Store records using S3 Intelligent-Tiering. Use an IAM policy to deny record deletion. After 10 years, change the IAM policy to allow deletion.
③ C. Use S3 Lifecycle policies to transition records from S3 Standard to S3 Glacier Deep Archive after one year. Use S3 Object Lock in compliance mode for 10 years.
④ D. Use S3 Lifecycle policies to transition records from S3 Standard to S3 One Zone-Infrequent Access (S3 One Zone-IA) after one year. Use S3 Object Lock in Governance mode for 10 years.

  회사는 회계 기록을 안전하게 저장해야 하며, 기록은 1년 동안 즉시 접근 가능해야 하며 이후 추가 9년 동안 보관되어야 합니다. 또한 누구도 10년 동안 기록을 삭제할 수 없도록 해야 합니다. 기록은 최대 복원력으로 저장되어야 하며, 즉시 접근이 가능해야 하므로 시작은 S3 Standard에서부터 합니다.

  1년 후에는 접근 빈도가 낮은 저장소로 이동해야 하므로 S3 Glacier나 S3 Glacier Deep Archive로 변경합니다. 기록 보관을 위해 S3 Glacier Deep Archive가 적합하며, 규정 준수 모드에서 S3 객체 잠금을 사용하면 10년 동안 기록을 삭제할 수 없도록 할 수 있습니다.

  따라서 정답은 C이며, 이는 회사의 요구 사항을 완벽하게 충족합니다. 다른 선택지는 기록 삭제를 완전히 방지하지 않거나, 최대 복원력을 보장하지 않습니다.

  정답은 3번입니다. 이 솔루션이 주어진 요구 사항을 가장 잘 충족하고 있습니다.

  즉시 접근성 1년 동안: S3 Standard는 빠른 액세스를 제공하므로, 초기 1년 동안 기록이 즉시 접근 가능합니다.
  이후 9년 보관: S3 수명 주기 정책을 통해 1년 후 기록은 비용 효율적이고 장기 보관에 적합한 S3 Glacier Deep Archive로 자동 전환됩니다.
  삭제 방지: 10년 동안 규정 준수 모드에서 S3 객체 잠금을 사용하면, 관리 사용자나 루트 사용자를 포함한 모든 사용자가 기록을 삭제하거나 수정할 수 없습니다. 이는 잠금 효과로 데이터의 보안과 불변성을 보장합니다.
  최대 복원력: S3 Glacier Deep Archive는 높은 내구성과 장기 보관에 최적화되어 있어 요구되는 최대 복원력을 제공합니다.
  다른 옵션들은 접근성, 삭제 방지 정책, 또는 장기 보관의 효율성과 보안 측면에서 요구 사항을 완전히 만족하지 못합니다. 따라서 옵션 3이 최적의 선택입니다.

54. A company runs multiple Windows workloads on AWS. Employees use a Windows file share hosted on two Amazon EC2 instances. The file shares synchronize data with each other and maintain redundant copies . The company wants a highly available and durable storage solution that preserves the way users currently access files.
What should a solution architect do to meet these requirements?
① A. Migrate all data to Amazon S3. Set up IAM authentication to allow users to access the files.
② B. Set up an Amazon S3 File Gateway. Mount the S3 File Gateway on an existing EC2 instance.
③ C. Extend your file sharing environment to Amazon FSx for Windows File Server using a Multi-AZ configuration. Migrate all data to FSx for Windows File Server.
④ D. Extend your file sharing environment to Amazon Elastic File System (Amazon EFS) using a Multi-AZ configuration. Migrate all data to Amazon EFS.

  회사는 현재 두 개의 Amazon EC2 인스턴스에서 호스팅되는 Windows 파일 공유를 사용하고 있습니다. 이 파일 공유는 서로 데이터를 동기화하고 중복 복사본을 유지합니다. 회사는 사용자가 현재 파일에 액세스하는 방식을 보존하는 가용성과 내구성이 뛰어난 스토리지 솔루션을 원합니다.

  이러한 요구 사항을 충족하기 위해 다중 AZ 구성을 사용하여 파일 공유 환경을 Windows 파일 서버용 Amazon FSx로 확장하는 것이 적절합니다. 모든 데이터를 FSx for Windows File Server로 마이그레이션하면 회사의 요구 사항을 충족할 수 있습니다. Amazon FSx는 Windows 파일 서버와 동일한 파일 시스템을 제공하며, 다중 AZ 구성은 고가용성과 내구성을 제공합니다. 또한, 기존의 Windows 파일 공유와의 호환성을 유지할 수 있습니다.

  회사가 요구하는 가용성과 내구성 향상, 그리고 현재 사용자 액세스 방식 유지의 관점에서 최적의 선택은 3번입니다.

  Amazon FSx for Windows File Server는 AWS 내에서 Windows 파일 시스템을 원활하게 지원하며, 다중 AZ 구성을 통해 높은 가용성을 보장합니다. 이 솔루션은 기존 Windows 환경과의 호환성이 뛰어나 현재 사용자들이 파일에 접근하는 방식을 그대로 유지할 수 있게 합니다. 또한, AWS 관리형 서비스이므로 데이터 관리와 복제에 대한 복잡성을 줄이고 내구성을 향상시킵니다.

  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  1번: Amazon S3는 객체 스토리지이므로 직접적인 파일 시스템 접근과 동기화 기능을 제공하지 않아 기존 액세스 패턴을 완전히 유지하기 어렵습니다.
  2번: S3 파일 게이트웨이는 온프레미스 환경에서 S3를 파일 시스템처럼 사용할 수 있게 하지만, 동기화와 고가용성 요구사항을 완벽하게 충족하진 못합니다.
  4번: Amazon EFS는 NFS 기반으로 주로 Linux 환경에 최적화되어 있으며, Windows 환경에 대한 네이티브 지원과 호환성 측면에서 FSx보다 제한적일 수 있습니다.
  따라서, 현재의 Windows 환경과 사용자 경험을 유지하면서 가용성과 내구성을 높이려면 3번의 Amazon FSx for Windows File Server가 가장 적합합니다.

55. A solution architect is developing a VPC architecture with multiple subnets. This architecture will host applications using Amazon EC2 instances and Amazon RDS DB instances. The architecture consists of six subnets across two Availability Zones. Each Availability Zone contains a public subnet, a private subnet, and a subnet dedicated to the database. Only EC2 instances running in the private subnets can access the RDS database.
What solution meets these requirements?
① A. Create a new routing table that excludes routes for the public subnet's CIDR block. Associate the route table with the database subnet.
② B. Create a security group that denies inbound traffic from security groups assigned to instances in the public subnet. Associate the security group with the DB instance.
③ C. Create a security group that allows inbound traffic from the security group assigned to instances in the private subnet. Associate the security group with the DB instance.
④ D. Create a new peering connection between the public and private subnets. Create another peering connection between the private subnet and the database subnet.

  문제의 요구 사항은 프라이빗 서브넷에서 실행되는 EC2 인스턴스만 RDS 데이터베이스에 액세스할 수 있도록 하는 것입니다. 이를 달성하기 위해 데이터베이스 인스턴스에 대한 액세스를 제한하는 보안 그룹을 생성해야 합니다.

  올바른 해결책은 퍼블릭 서브넷의 인스턴스에 할당된 보안 그룹의 인바운드 트래픽을 허용하는 것이 아니라, 프라이빗 서브넷의 인스턴스에 할당된 보안 그룹의 인바운드 트래픽을 허용하는 보안 그룹을 생성하는 것입니다. 이러한 보안 그룹을 RDS DB 인스턴스에 연결하면 프라이빗 서브넷에서 실행되는 EC2 인스턴스만 데이터베이스에 액세스할 수 있습니다.

  이와 같은 접근 방식은 네트워크 트래픽을 제어하고 데이터베이스에 대한 액세스를 제한하여 보안을 강화합니다. 따라서 정답은 C입니다.

  정답은 3번입니다.

  프라이빗 서브넷에서 실행되는 EC2 인스턴스만 RDS 데이터베이스에 접근해야 하는 요구사항을 충족시키기 위해서는 직접적인 네트워크 접근 제어가 필요합니다. 보안 그룹을 통해 트래픽을 세밀하게 관리할 수 있습니다.

  옵션 1은 라우팅 테이블을 통해 네트워크 경로를 제어하려고 하지만, RDS 데이터베이스에 대한 접근 제한은 네트워크 경로 설정으로만 해결되지 않습니다.
  옵션 2는 퍼블릭 서브넷의 인스턴스 트래픽을 거부하려 하지만, 이는 퍼블릭 서브넷과 DB 인스턴스 간의 문제만 다루며, 프라이빗 서브넷 내에서의 접근 제어에는 영향을 미치지 않습니다.
  옵션 4의 VPC 피어링은 서로 다른 VPC 간의 통신을 용이하게 하지만, 현재 요구 사항은 단일 VPC 내에서의 접근 제어에 초점이 맞춰져 있어 과도한 해결책입니다.
  따라서 옵션 3이 적합합니다. 프라이빗 서브넷의 EC2 인스턴스만 RDS DB 인스턴스에 접근할 수 있도록 해당 인스턴스의 보안 그룹을 설정하여 RDS DB 인스턴스에 대한 인바운드 트래픽을 허용하는 정책을 적용합니다. 이 방법으로 프라이빗 서브넷 외의 인스턴스들은 DB 접근이 차단됩니다. 이렇게 하면 요구 사항을 정확히 충족시킬 수 있습니다.

56. A company has registered a domain name with Amazon Route 53. The company uses Amazon API Gateway in the ca-central-1 region as the public interface for its backend microservice API. Third-party services use the API securely. The company wants to design the API Gateway URL with the company's domain name and its corresponding certificate so that the third-party service can use HTTPS . What solution meets these requirements?
① A. To override the default URL, create a stage variable in API Gateway using Name="Endpoint-URL" and Value="Company Domain Name." Import the public certificate associated with your company's domain name into AWS Certificate Manager (ACM).
② B. Create a Route 53 DNS record with your company's domain name. Point the alias record to the Regional API Gateway stage endpoint. Import the public certificate associated with your company's domain name into AWS Certificate Manager (ACM) in the us-east-1 region.
③ C. Create a regional API Gateway endpoint. Associate the API Gateway endpoint with your company's domain name. Import the public certificate associated with your company's domain name into AWS Certificate Manager (ACM) in the same region. Associate the certificate with the API Gateway endpoint. Configure Route 53 to route traffic to the API Gateway endpoint.
④ D. Create a regional API Gateway endpoint. Associate the API Gateway endpoint with your company's domain name. Import the public certificate associated with your company's domain name into AWS Certificate Manager (ACM) in the us-east-1 region. Associate the certificate with your API Gateway API. Create a Route 53 DNS record with your company's domain name. Point the A record to your company's domain name.

  회사가 안전한 HTTPS 연결을 위해 타사 서비스에서 사용할 수 있는 API 게이트웨이 URL을 만들기 위해 다음을 수행해야 합니다.

  regional API 게이트웨이 엔드포인트를 생성하고 회사의 도메인 이름과 연결합니다. 회사의 도메인 이름과 연결된 인증서를 동일한 리전의 AWS Certificate Manager로 가져옵니다. API 게이트웨이 엔드포인트에 인증서를 연결합니다. 트래픽을 API 게이트웨이 엔드포인트로 라우팅하도록 Route 53을 구성합니다. 이러한 단계를 수행하면 회사의 도메인 이름과 인증서로 HTTPS를 사용할 수 있는 API 게이트웨이 URL을 생성할 수 있습니다.

  정답은 3번입니다.

  회사의 요구사항을 충족하려면 다음과 같은 접근이 필요합니다:

  API Gateway 엔드포인트 생성: 먼저, 회사의 마이크로서비스 API를 위한 API Gateway 엔드포인트를 해당 리전(ca-central-1)에서 생성해야 합니다. 이는 API의 공용 인터페이스 역할을 합니다.

  도메인 이름 연결: 생성된 API Gateway 엔드포인트를 회사의 자체 도메인 이름과 직접 연결해야 합니다. 이렇게 하면 타사 서비스가 회사의 도메인을 통해 API에 접근할 수 있습니다.

  인증서 관리: 회사 도메인 이름에 대한 유효한 공인 인증서는 동일한 리전(ca-central-1)의 AWS Certificate Manager (ACM)에서 관리되어야 합니다. 인증서는 API Gateway 엔드포인트의 보안을 위해 필수적입니다.

  HTTPS 지원: API Gateway 엔드포인트에 인증서를 직접 연결함으로써 HTTPS 통신을 지원할 수 있습니다. 이를 통해 트래픽은 안전하게 암호화되어 전송됩니다.

  Route 53 구성: 마지막으로, Route 53을 통해 회사의 도메인 이름으로 DNS 레코드를 생성하고, 이 레코드가 생성된 API Gateway 엔드포인트로 트래픽을 라우팅하도록 설정해야 합니다. 하지만, 이 과정에서 별도의 별칭 레코드 대신 직접적인 구성이 더 적합하며, 문제의 옵션 중에서는 3번이 이러한 요소들을 가장 적절하게 포괄합니다.

  따라서, 옵션 3은 도메인 이름과 인증서의 적절한 관리, 그리고 HTTPS를 통한 보안 액세스를 포함한 전체적인 솔루션을 제시하므로 가장 적합합니다. 옵션 B와 D는 인증서를 잘못된 리전(us-east-1)에서 관리하려 하거나 구성 요소가 불완전하게 제시되어 있습니다. 옵션 1은 단계 변수 생성에 초점을 맞추고 있어 핵심적인 도메인 및 인증서 연결 부분이 부족합니다.

57. A company operates a popular social media website. The website allows users to upload images and share them with others. The company wants to ensure that the images do not contain inappropriate content . The company needs a solution that minimizes development effort.
What should a solution architect do to meet this requirement?
① A. Use Amazon Comprehend to detect inappropriate content. Use human review for low-confidence predictions.
② B. Use Amazon Rekognition to detect inappropriate content. Use human review for low-confidence predictions.
③ C. Use Amazon SageMaker to detect inappropriate content. Label low-confidence predictions using the correct answer.
④ D. Deploy a custom machine learning model using AWS Fargate to detect inappropriate content. Use the correct answer to label low-confidence predictions.

  이 회사는 이미지에 부적절한 내용이 포함되어 있지 않은지 확인하고자 합니다. 이를 위해 개발 노력을 최소화하는 솔루션이 필요합니다. 회사의 요구 사항을 충족하기 위해선 이미지에 대한 분석이 필요합니다.

  Amazon Comprehend는 텍스트 분석에 중점을 둔 서비스로 이미지 분석에는 적합하지 않습니다. Amazon SageMaker와 AWS Fargate는 기계 학습 모델을 생성하고 배포하는 데 사용되지만 이미지 분석을 위해 사전構築된 모델을 제공하지는 않습니다.

  Amazon Rekognition은 이미지와 비디오를 분석하여 부적절한 콘텐츠를 탐지할 수 있는 서비스입니다. 개발 노력을 최소화하면서도 정확한 결과를 얻을 수 있습니다. 신뢰도가 낮은 예측에는 사람의 검토를 사용하여 결과를 보완할 수 있습니다. 따라서 이 회사의 요구 사항을 충족하는 가장 적합한 솔루션은 Amazon Rekognition을 사용하여 부적절한 콘텐츠를 탐지하는 것입니다.

  정답은 2번인 이유는 다음과 같습니다.

  Amazon Rekognition은 이미지와 비디오 분석에 특화된 AWS 서비스로, 즉시 사용 가능한 기능을 통해 부적절하거나 민감한 콘텐츠를 효과적으로 감지할 수 있습니다. 특히, 이 서비스는 미리 훈련된 모델을 통해 성인용 콘텐츠, 폭력적인 이미지 등 다양한 유형의 부적절한 이미지를 식별하는 데 최적화되어 있습니다.

  1번의 Amazon Comprehend는 주로 텍스트 분석에 초점을 맞추고 있어 이미지 콘텐츠 분석에는 적합하지 않습니다.

  3번의 Amazon SageMaker와 4번의 AWS Fargate는 더 유연한 기계 학습 모델 개발 및 배포를 지원하지만, 이 경우 이미 고도화된 이미지 분석 기능이 필요하며 개발 및 훈련에 상당한 시간과 노력이 필요합니다. 이를 피하려는 회사의 요구사항에 맞지 않습니다.

  따라서, 최소한의 개발 노력으로 빠르게 부적절한 이미지를 감지하려는 목표를 달성하기 위해서는 Amazon Rekognition이 가장 적합한 선택입니다. 신뢰도가 낮은 예측에 대해서는 추가적인 인간 검토를 병행하는 방안도 제시되어 있어 실용적입니다.

58. A company wants to run critical applications in containers to meet scalability and availability requirements . The company prefers to focus on maintaining these critical applications. The company doesn't want to be responsible for provisioning and managing the underlying infrastructure that runs containerized workloads .
What should a solution architect do to meet these requirements?
① A. Use an Amazon EC2 instance and install Docker on the instance.
② B. Use Amazon Elastic Container Service (Amazon ECS) on Amazon EC2 worker nodes.
③ C. Use Amazon Elastic Container Service (Amazon ECS) on AWS Fargate.
④ D. Use Amazon EC2 instances from Amazon Machine Images (AMIs) optimized for Amazon Elastic Container Service (Amazon ECS).

  해당 회사는 중요 애플리케이션의 유지 보수에 집중하고 싶어하며, 컨테이너화된 워크로드를 실행하는 기본 인프라를 프로비저닝하고 관리하는 일을 원치 않습니다. 이러한 요구 사항을 충족하기 위해 솔루션 설계자는 AWS Fargate에서 Amazon Elastic Container Service(Amazon ECS)를 사용해야 합니다. AWS Fargate는 컨테이너화된 애플리케이션을 실행하는 데 사용되는 서버리스 컴퓨팅 엔진입니다. AWS Fargate를 사용하면phyl로 서버를 프로비저닝하거나 관리할 필요가 없으므로 회사의 요구 사항을 충족할 수 있습니다. Amazon ECS는 컨테이너화된 애플리케이션을 실행하고 관리하는 데 사용되는 오케스트레이션 서비스입니다. AWS Fargate와 함께 Amazon ECS를 사용하면 컨테이너화된 애플리케이션을 쉽게 실행하고 관리할 수 있습니다. 따라서 솔루션 설계자는 AWS Fargate에서 Amazon ECS를 사용하여 회사의 요구 사항을 충족해야 합니다.

  회사는 인프라 관리 부담을 최소화하고 애플리케이션 유지보수에 집중하고자 합니다. 이 요구 사항을 충족하려면, 직접 인스턴스 관리나 인프라 프로비저닝을 필요로 하지 않는 솔루션이 필요합니다.

  옵션 A는 EC2 인스턴스 직접 관리가 필요하여 인프라 관리 부담이 있습니다.
  옵션 B는 ECS를 사용하지만, 여전히 EC2 작업자 노드를 관리해야 합니다.
  옵션 D 역시 EC2 인스턴스를 직접 관리해야 하므로 인프라 관리 부담이 남아 있습니다.
  따라서, 정답은 C. AWS Fargate에서 Amazon Elastic Container Service(Amazon ECS)를 사용하십시오 입니다. Fargate는 서버리스 컨테이너 오케스트레이션을 제공하여, 사용자는 인스턴스 프로비저닝이나 인프라 관리 없이 컨테이너 워크로드를 실행할 수 있습니다. 이로 인해 회사는 애플리케이션에만 집중할 수 있게 됩니다.

59. A company hosts over 300 global websites and applications. They need a platform capable of analyzing over 30 TB of clickstream data daily. What should a solutions architect do to transmit and process the clickstream data?
① A. Design AWS Data Pipeline to store data in an Amazon S3 bucket and launch an Amazon EMR cluster with the data to generate analytics.
② B. Create an Auto Scaling group of Amazon EC2 instances to process the data and send it to an Amazon S3 data lake for analysis by Amazon Redshift.
③ C. Cache data in Amazon CloudFront. Store data in an Amazon S3 bucket. When an object is added to the S3 bucket, an AWS Lambda function is executed to process the data for analytics.
④ D. Collect data from Amazon Kinesis Data Streams. Use Amazon Kinesis Data Firehose to deliver the data to an Amazon S3 data lake. Load the data into Amazon Redshift for analysis.

  클릭스트림 데이터를 전송하고 처리하기 위해 솔루션 아키텍트는 실시간으로 대량의 데이터를 수집하고 처리할 수 있는 도구를 사용해야 합니다. 30TB가 넘는 클릭스트림 데이터를 분석하려면 데이터를 효율적으로 수집, 전송, 처리할 수 있는 시스템을 설계해야 합니다.

  Amazon Kinesis Data Streams를 사용하여 클릭스트림 데이터를 수집합니다. Amazon Kinesis Data Firehose를 사용하여 수집된 데이터를 Amazon S3 데이터 레이크로 전송하여 저장하고, 이후 분석을 위해 Amazon Redshift에 데이터를 로드합니다. 이 솔루션은 대량의 데이터를 처리하고 분석하기 위한 효율적인 아키텍처이며, 데이터 처리와 분석을 위한 각 단계에서 적절한 AWS 서비스를 활용합니다.

  이러한 아키텍처는 데이터를 실시간으로 수집하고 처리하며, 이후 분석을 위해 데이터를 저장하고 처리할 수 있습니다. 따라서 이 아키텍처는 클릭스트림 데이터를 분석하기 위한 적절한 솔루션입니다.

  정답은 4번입니다.

  초고속으로 유입되는 클릭스트림 데이터를 효율적으로 처리하려면, 실시간 스트리밍 데이터 처리 기능이 필수적입니다. Amazon Kinesis Data Streams는 이러한 실시간 데이터 스트리밍에 최적화되어 있어, 대량의 데이터를 즉시 수집하고 처리할 수 있습니다.

  그 후, Amazon Kinesis Data Firehose는 복잡한 설정 없이도 데이터를 안정적으로 Amazon S3 데이터 레이크로 전송하는 역할을 합니다. 이렇게 저장된 데이터는 구조화되고 분석 준비가 완료됩니다. 마지막으로, 분석을 위해 Amazon Redshift로 데이터를 로드하면 고성능의 SQL 기반 분석이 가능해집니다. 따라서, 실시간 수집, 안정적 전송, 그리고 효율적인 분석을 위한 솔루션으로 4번이 가장 적합합니다.

  다른 옵션들은 데이터 처리의 일부 측면에 강점이 있지만, 전체적인 실시간 데이터 처리와 분석 파이프라인 구축 측면에서는 4번이 최적의 선택입니다.

60. A company has a website hosted on AWS. The website is behind an Application Load Balancer (ALB) configured to handle HTTP and HTTPS separately . The company wants to route all requests to the website using HTTPS
. What should a solution architect do to meet this requirement?
① A. Update the ALB's network ACL to allow only HTTPS traffic.
② B. Create a rule to change HTTP in the URL to HTTPS.
③ C. Create a listener rule on the ALB to redirect HTTP traffic to HTTPS.
④ D. Replace the ALB with a Network Load Balancer configured to use Server Name Indication (SNI).

  회사가 모든 요청을 HTTPS를 사용하도록 웹 사이트로 전달하고자 하는데 이를 위해서는 ALB에 리스너 규칙을 생성하여 HTTP 트래픽을 HTTPS로 리디렉션해야 합니다. 이는 ALB의 기능 중 하나로, HTTP 요청을 HTTPS로 자동으로 전환하여 웹사이트가 HTTPS를 사용하도록 강제할 수 있습니다. 다른 옵션들은 이 요구 사항을 충족하지 못하므로,πό Listener 규칙을 생성하여 HTTP를 HTTPS로 리디렉션하는 것이 올바른 해결책입니다

  정답은 C. ALB에 리스너 규칙을 생성하여 HTTP 트래픽을 HTTPS로 리디렉션합니다. 입니다.

  회사는 모든 요청을 HTTPS로 강제하려는 것이 목표입니다.

  네트워크 ACL은 트래픽 허용 범위를 제어하지만, 요청을 리다이렉트하는 기능은 제공하지 않습니다. (1번은 틀림)
  URL 자체를 변경하는 것은 클라이언트 측에서 이루어져야 하며, 서버 측에서 직접적으로 해결할 수 없습니다. (2번은 틀림)
  Network Load Balancer는 기능이 다르며, 요구 사항을 해결하는 최적의 방법이 아닙니다. (4번은 틀림)
  따라서 ALB의 리스너 규칙을 설정하여 HTTP 요청을 받으면 자동으로 HTTPS로 리다이렉트하게 하는 것이 가장 효과적인 해결책입니다. (3번 정답)

61. A company is developing a two-tier web application on AWS. The company's developers have deployed the application on an Amazon EC2 instance that connects directly to a backend Amazon RDS database . The company must not hardcode database credentials into the application. Furthermore, the company must implement a solution that automatically rotates database credentials
periodically. What solution meets these requirements with minimal operational overhead?
① A. Store database credentials in instance metadata. Use an Amazon EventBridge (Amazon CloudWatch Events) rule to run a scheduled AWS Lambda function that updates RDS credentials and instance metadata simultaneously.
② B. Store database credentials in a configuration file in an encrypted Amazon S3 bucket. Use an Amazon EventBridge (Amazon CloudWatch Events) rule to run a scheduled AWS Lambda function that simultaneously updates the RDS credentials and the configuration file. S3 versioning allows you to revert to previous values.
③ C. Store the database credentials as a secret in AWS Secrets Manager. Enable automatic rotation for the secret. To grant access to the secret, attach the necessary permissions to the EC2 role.
④ D. Store database credentials as encrypted parameters in AWS Systems Manager Parameter Store. Enable automatic rotation for encrypted parameters. To grant access to encrypted parameters, attach the necessary permissions to the EC2 role.

  정답은 3번입니다. AWS Secrets Manager를 사용하는 것이 가장 적합한 솔루션입니다. 이유는 다음과 같습니다:

  보안: Secrets Manager는 데이터베이스 자격 증명과 같은 민감한 정보를 안전하게 저장하고 관리하도록 설계되었습니다. 직접 하드코딩 없이 보안을 강화합니다.
  자동 교체: Secrets Manager는 비밀의 자동 순환 기능을 제공하여 정기적으로 자격 증명을 교체할 수 있습니다. 이는 보안 위험을 최소화하는 데 효과적입니다.
  간편한 액세스: EC2 인스턴스에 적절한 IAM 역할을 부여하면, Secrets Manager에서 직접 비밀을 쉽게 참조할 수 있습니다. 이로 인해 개발자는 복잡한 메타데이터 관리나 추가적인 업데이트 로직 없이도 자격 증명을 안전하게 사용할 수 있습니다.
  다른 옵션들은 일부 요구 사항을 충족하지만, Secrets Manager만큼 효율적이고 보안에 최적화된 방식으로 자동 자격 증명 교체와 관리를 제공하지 않습니다. 특히, S3와 Parameter Store는 민감한 정보 관리에 특화된 기능이 Secrets Manager보다 제한적입니다.

62. A company is deploying a new public web application on AWS. The application runs behind an Application Load Balancer (ALB). The application must be encrypted at the edge using an SSL/TLS certificate issued by an external certificate authority (CA). The certificate must be replaced annually before its expiration.
What should the solution architect do to meet this requirement?
① A. Issue an SSL/TLS certificate using AWS Certificate Manager (ACM). Apply the certificate to your ALB. Use the managed renewal feature to automatically replace the certificate.
② B. Issue an SSL/TLS certificate using AWS Certificate Manager (ACM). Retrieve key material from the certificate. Apply the certificate to AL. Use the managed renewal feature to automatically replace the certificate.
③ C. Use AWS Certificate Manager (ACM) Private Certificate Authority to issue an SSL/TLS certificate from a root CA. Apply the certificate to ALB. Use the managed renewal feature to automatically replace the certificate.
④ D. Import an SSL/TLS certificate using AWS Certificate Manager (ACM). Apply the certificate to the ALB. Use Amazon EventBridge (Amazon CloudWatch Events) to send a notification when the certificate is nearing expiration. Manually rotate the certificate.

  이 질문의 정답은 D입니다. 솔루션 설계자는 외부 인증 기관에서 발급한 SSL/TLS 인증서를 얻은 후 ALB에 적용해야 합니다. 또한 Amazon EventBridge를 사용하여 인증서 만료 알림을 받은 후 인증서를 수동으로 갱신하여 교체해야 합니다. CA에서 발급한 인증서를 AWS Certificate Manager에서 관리해야 하므로 ACM을 사용하여 인증서를 가져와야 합니다. ACM을 사용하면 AWS 리소스에 대한 SSL/TLS 인증서를 쉽게 프로비저닝, 관리 및 갱신할 수 있기 때문입니다. 그러나 외부 CA에서 발급한 인증서를 사용하는 경우 관리형 갱신 기능을 사용할 수 없기 때문에 인증서 만료 시 수동으로 교체해야 합니다.

  정답은 4번입니다.

  ACM은 SSL/TLS 인증서 발급과 관리를 지원하지만, 자동 갱신 기능은 공용 인증서에만 적용됩니다. 문제에서 외부 CA로부터 발급받는 인증서를 언급하여, 이는 사설 인증서로 간주될 수 있으며, ACM의 관리형 갱신 기능은 공용 인증서에 한정되어 있어 적합하지 않습니다. 따라서 옵션 1과 3은 적절하지 않습니다.

  옵션 2는 키 자료를 직접 가져와야 한다고 언급하여 불필요한 복잡성을 추가하고, AL(ALB의 오역으로 보임)이라는 잘못된 용어를 사용해 정확성을 잃습니다.

  옵션 4는 AWS Certificate Manager를 통해 인증서를 관리하고, ALB에 적용하는 올바른 접근법을 제시합니다. 만료 전에 Amazon EventBridge(CloudWatch Events)를 활용해 알림을 설정하여 수동으로 인증서를 갱신하도록 하는 것은 주어진 조건을 가장 효과적으로 충족하는 방법입니다. 이는 외부 CA 인증서의 특성과 관리상의 현실적인 접근을 반영하고 있습니다.

63. A company runs its infrastructure on AWS and has a user base of 700,000 registered users for its document management application. The company plans to build a product that converts large .pdf files into .jpg image files. The average .pdf file size is 5MB. The company needs to retain both the original and converted files . The solution architect needs to design a scalable solution that can accommodate rapidly increasing demand over time.
What solution will most cost-effectively meet these requirements?
① A. Save the .pdf file to Amazon S3. Configure an S3 PUT event to invoke an AWS Lambda function that converts the file to .jpg format and saves it back to Amazon S3.
② B. Save the .pdf file to Amazon DynamoDB. Use the DynamoDB Streams feature to invoke an AWS Lambda function to convert the file to .jpg format and save it back to DynamoDB.
③ C. Upload the .pdf file to an AWS Elastic Beanstalk application that includes an Amazon EC2 instance, Amazon Elastic Block Store (Amazon EBS) storage, and an Auto Scaling group. Use a program on the EC2 instance to convert the file to .jpg format. Store the .pdf and .jpg files in the EBS store.
④ D. Upload the .pdf file to an AWS Elastic Beanstalk application that includes an Amazon EC2 instance, Amazon Elastic File System (Amazon EFS) storage, and an Auto Scaling group. Use a program on the EC2 instance to convert the file to .jpg format. Store the .pdf and .jpg files in the EBS store.

  첫 번째 방법은 pdf 파일을 아마존 S3에 저장하고 S3 PUT 이벤트를 통해 Lambda 함수를 호출하여 pdf 파일을 jpg 형식으로 변환하고 다시 S3에 저장하는 방법이다. 이 방법은_STORAGE 비용을 최소화하면서 수요에 따라 확장할 수 있다. 다른 방법은 스토리지와 컴퓨팅 리소스에 대한 비용이 더 많이 들며 확장성이 떨어진다. especialmente dynamoDB는 키-값 및 문서 및 테이블과 같은 스키마 더이상 될 수 없기 때문에 이경우에는 적합하지 않다. 따라서 첫 번째 방법이 가장 비용 효율적인 솔루션이다.

  정답은 1번입니다.

  Amazon S3와 AWS Lambda의 조합이 가장 비용 효율적이고 확장 가능한 솔루션입니다.

  S3: 대용량 파일 저장에 최적화되어 있어, 원본 .pdf와 변환된 .jpg 파일을 효율적으로 보관할 수 있습니다.
  S3 PUT 이벤트 트리거: 새로운 파일 업로드 시 자동으로 AWS Lambda 함수를 호출하여 파일 변환을 시작합니다.
  AWS Lambda: 필요에 따라 동적으로 실행되므로, 트래픽이 증가하더라도 사전에 설정된 리소스 없이도 확장 가능합니다. 이는 고정 비용을 최소화하면서 변동 수요에 대응할 수 있다는 장점이 있습니다.
  다른 옵션들은 다음과 같은 이유로 비효율적입니다:

  B (DynamoDB): 데이터베이스 서비스로 파일 저장에 최적화되어 있지 않으며, 스트림 기능을 사용해 파일 변환을 처리하는 것은 복잡하고 비용이 많이 들 수 있습니다.
  C와 D (EC2 + EBS/EFS + Auto Scaling): 인스턴스 기반 솔루션은 항상 실행 중인 인스턴스 비용이 발생하며, 자동 확장은 트래픽 변동에 대응하더라도 기본적인 리소스 유지 비용이 높아집니다. 특히 파일 저장에 EBS나 EFS를 사용하는 것은 S3보다 비용 효율성이 떨어집니다.
  따라서, 1번 옵션이 확장성과 비용 효율성 측면에서 가장 적합한 선택입니다.

64. A company has over 5 TB of file data on Windows file servers running on-premises. Users and applications interact with this data daily.
The company is migrating its Windows workloads to AWS. As the company continues this process, it needs to access AWS and its on-premises file storage with minimal latency . The company needs a solution that minimizes operational overhead and does not require significant changes to existing file access patterns. The company uses AWS Site-to-Site VPN connections to connect to AWS.
What should a solution architect do to meet these requirements?
① A. Deploy and configure Amazon FSx for Windows File Server on AWS. Move on-premises file data to FSx for Windows File Server. Reconfigure your workloads to use FSx for Windows File Server on AWS.
② B. Deploy and configure an Amazon S3 File Gateway on-premises. Move on-premises file data to the S3 File Gateway. Reconfigure both on-premises and cloud workloads to use the S3 File Gateway.
③ C. Deploy and configure an Amazon S3 File Gateway on-premises. Move on-premises file data to Amazon S3. Reconfigure your workloads to use Amazon S3 directly or use the S3 File Gateway. This depends on the location of each workload.
④ D. Deploy and configure Amazon FSx for Windows File Server on AWS. Deploy and configure an Amazon FSx File Gateway on-premises. Move on-premises file data to the FSx File Gateway. Configure cloud workloads to use FSx for Windows File Server on AWS. Configure on-premises workloads to use the FSx File Gateway.

  회사는 온프레미스와 AWS에서 실행되는 워크로드를 위한 최소한의 지연 시간으로 AWS 및 온프레미스 파일 스토리지에 액세스해야 합니다. 이는 온프레미스와 AWS 간에 파일을 쉽게 공유할 수 있도록 하는 솔루션이 필요함을 의미합니다.

  정답인 4번은 AWS에서 Windows 파일 서버용 Amazon FSx를 배포하고 구성합니다. 또한 온프레미스에 Amazon FSx 파일 게이트웨이를 배포하고 구성하여 온프레미스 파일 데이터를 FSx 파일 게이트웨이로 이동합니다. 이 솔루션은 AWS와 온프레미스 모두에서 워크로드가 파일에 액세스할 수 있도록 하며, 운영 오버헤드를 최소화하고 기존 파일 액세스 패턴을 크게 변경할 필요가 없습니다.

  이러한 요구 사항을 충족하기 위해 솔루션 설계자는 AWS에서 Windows 파일 서버용 Amazon FSx를 배포하고 구성하고, 온프레미스에 Amazon FSx 파일 게이트웨이를 배포 및 구성하여 최소한의 지연 시간과 쉬운 파일 공유를 지원하는 아키텍처를 구축해야 합니다.

  정답은 D입니다.

  회사의 요구사항은 최소한의 지연 시간으로 동시에 AWS와 온프레미스 스토리지에 액세스하면서, 기존의 파일 액세스 패턴을 크게 변경하지 않고 운영 오버헤드를 최소화하는 것입니다.

  1번(A)은 온프레미스에서 AWS의 Amazon FSx for Windows File Server로 데이터를 완전히 이전하는 방식으로, 이는 지연 시간을 최소화하려는 목표와 맞지 않으며, 온프레미스 액세스를 위해 추가적인 복잡성을 초래할 수 있습니다.

  2번(B)과 3번(C)은 S3 파일 게이트웨이를 활용하는 옵션입니다. 그러나 이들 방법은 주로 클라우드 기반의 데이터 관리에 초점을 맞추며, 온프레미스 애플리케이션의 지연 시간을 최소화하는데 한계가 있습니다. 특히 3번은 데이터를 완전히 S3로 이동하므로 기존 액세스 패턴을 크게 변경해야 할 수도 있습니다.

  4번(D)은 최적의 선택입니다. Amazon FSx for Windows File Server를 AWS에서 배포하여 클라우드 워크로드에 대한 고성능 파일 액세스를 제공하고, 동시에 온프레미스에 FSx 파일 게이트웨이를 구성하여 기존 온프레미스 시스템의 지연 시간을 최소화합니다. 이렇게 하면 회사는 온프레미스와 클라우드 사이에서 원활한 데이터 액세스를 유지하면서 운영 복잡성을 줄일 수 있습니다. AWS Site-to-Site VPN 연결을 통해 네트워크 안정성과 보안도 확보할 수 있습니다. 따라서 D 옵션이 주어진 요구 사항을 가장 잘 충족합니다.

65. A hospital recently deployed a RESTful API using Amazon API Gateway and AWS Lambda. The hospital uses API Gateway and Lambda to upload reports in PDF and JPEG formats. The hospital needs to modify the Lambda code to identify protected health information (PHI) in the reports.
What solution can meet these requirements with minimal operational overhead?
① A. Extract text from reports using existing Python libraries and identify PHI in the extracted text.
② B. Extract text from reports using Amazon Textract. Identify PHI from the extracted text using Amazon SageMaker.
③ C. Extract text from reports using Amazon Textract. Identify PHI from the extracted text using Amazon Comprehend Medical.
④ D. Extract text from reports using Amazon Rekognition. Identify PHI from the extracted text using Amazon Comprehend Medical.

  최소한의 운영 오버헤드로 병원에서 요구하는 기능을 제공하는 방법은 Amazon Textract와 Amazon Comprehend Medical을 사용하는 것입니다. Amazon Textract는 기계 학습을 사용하여 다양한 서식의 텍스트와 데이터를 자동으로 추출하는 완전 관리되는 서비스로, PDF나 JPEG 형식의 문서에서 텍스트를 추출하는 데 사용할 수 있습니다. Amazon Comprehend Medical은 텍스트에서 의료 정보를 분석하고 식별하는 서비스로, PHI를 식별하는 데 사용할 수 있습니다. 이러한 두 가지 서비스를 사용하면 코드를 많이 수정할 필요 없이 병원의 요구 사항을 충족할 수 있습니다. 따라서, 최소한의 운영 오버헤드로 이러한 요구 사항을 충족하는 솔루션은 C입니다.

  병원이 개인 건강 정보(PHI)를 효율적으로 식별하면서 운영 복잡성을 최소화하려면, 가장 적합한 솔루션은 3번입니다. Amazon Textract는 PDF와 JPEG 같은 다양한 형식의 문서에서 텍스트와 구조화된 데이터를 정확하게 추출하는 데 최적화되어 있습니다. 추출된 텍스트에서 PHI를 식별하는 데는 전문적인 의료 용어와 정보를 이해하고 처리하는 능력이 필수적인데, 이 부분에서 Amazon Comprehend Medical이 뛰어난 성능을 발휘합니다. 이 서비스는 의료 텍스트 분석에 특화되어 있어 PHI를 효과적으로 감지하고 분류할 수 있습니다.

  1번 옵션은 기존 라이브러리를 사용할 수 있지만, 의료 데이터의 복잡성과 정확성 요구를 충족시키는 데 한계가 있을 수 있습니다. 2번은 Textract를 올바르게 사용하지만, SageMaker는 일반적인 머신 러닝 모델 훈련에 더 초점을 맞추고 있어 의료 데이터 특화 솔루션인 Comprehend Medical보다 효율적이지 않을 수 있습니다. 마지막으로, 4번의 Amazon Rekognition은 주로 이미지와 비디오 분석에 강점을 두므로 텍스트 추출에 있어 Textract보다 적합하지 않습니다. 따라서, 최소한의 오버헤드와 높은 정확성을 위해 3번이 최적의 선택입니다.

66. A company has an application that generates a large number of files, each approximately 5 MB in size. The files are stored in Amazon S3. Company policy requires that the files be retained for four years before deletion. The files contain critical business data that cannot be easily recovered, so immediate access is essential. The files are frequently accessed during the first 30 days after creation, but are rarely accessed after that.
What is the most cost-effective storage solution?
① A. Create an S3 bucket lifecycle policy that moves files from S3 Standard to S3 Glacier 30 days after object creation. Delete files 4 years after object creation.
② B. Create an S3 bucket lifecycle policy that moves files from S3 Standard to S3 One Zone-Infrequent Access (S3 One Zone-IA) 30 days after object creation. Delete files 4 years after object creation.
③ C. Create an S3 bucket lifecycle policy that moves files from S3 Standard to S3 Standard-Infrequent Access (S3 Standard-IA) within 30 days of object creation. Delete files after 4 years from object creation.
④ D. Create an S3 bucket lifecycle policy that moves files from S3 Standard to S3 Standard-Infrequent Access (S3 Standard-IA) within 30 days of object creation. Move files to S3 Glacier after 4 years of object creation.

  가장 비용 효율적인 스토리지 솔루션을 찾기 위해서는 파일 액세스 패턴과 보관 요구사항을 고려해야 합니다. 파일은 생성 후 30일 동안 자주 액세스되지만 이후에는 거의 액세스되지 않으므로, 초기에는 높은 성능과 즉각적인 접근성이 필요한 스토리지가 필요합니다.

  그러나 30일이 지나면 파일은 자주 액세스되지 않으므로, 저장 비용이较 낮은 인프리퀀트 액세스 스토리지로 이동하는 것이 합리적입니다. S3 Standard-Infrequent Access(S3 Standard-IA) 는 자주 액세스되지 않는 데이터에 적합한 솔루션, 데이터를 즉시 액세스할 수 있습니다.

  또한, 4년이 지난 후에는 파일을 삭제할 수 있습니다. 따라서, 생성 후 30일이 지나면 S3 Standard에서 S3 Standard-IA로 파일을 이동하는 수명 주기 정책을 생성하고, 4년이 지나면 파일을 삭제하는 방안이 가장 비용 효율적입니다.

  정답은 3번입니다.

  회사의 요구 사항에 따르면, 파일은 초기 30일 동안 높은 접근성이 필요하지만 이후에는 액세싱 빈도가 급격히 줄어듭니다. 4년 동안 안전하게 보관해야 하므로 장기 보관 비용도 고려해야 합니다.

  옵션 A와 D는 4년 후에 파일을 삭제하거나 S3 Glacier로 이동시키므로 장기 보관이 필요한 정책에 맞지 않습니다.
  옵션 B의 S3 One Zone-IA는 저렴하지만, 데이터의 접근성과 가용성에 제약이 있을 수 있습니다. 특히 비즈니스 데이터의 즉각적인 접근성이 요구되는 상황에서는 적합하지 않습니다.
  옵션 C는 초기 30일 동안 S3 Standard로 높은 접근성을 유지하고, 그 이후에는 S3 Standard-IA로 이동하여 비용을 절감합니다. S3 Standard-IA는 Standard에 비해 저렴하면서도 필요한 경우 빠른 액세스가 가능해 비즈니스 요구 사항을 잘 충족시키며, 4년 후 삭제 정책으로 장기 보관 비용도 최소화합니다.
  따라서, 비용 효율성과 접근성 요구 사항을 모두 충족시키는 최적의 선택은 옵션 C입니다.

67. A company hosts an application on multiple Amazon EC2 instances. The application processes messages in an Amazon SQS queue, writes them to an Amazon RDS table, and then deletes the messages from the queue. Duplicate records are occasionally found in the RDS table. The SQS queue does not contain duplicate messages.
What should the solution architect do to ensure that messages are processed only once?
① A. Create a new queue using the CreateQueue API call.
② B. Add the appropriate permissions using the AddPermission API call.
③ C. Use the receiveMessage API call to set an appropriate wait time.
④ D. Increase the visibility timeout using the ChangeMessageVisibility API call.

  이 문제는 Amazon EC2 인스턴스와 Amazon SQS 대기열을 사용하여 애플리케이션을 호스팅하는 상황에서 중복 레코드의 문제를 해결하는 것입니다. 애플리케이션은 Amazon SQS 대기열의 메시지를 처리하고 Amazon RDS 테이블에 기록한 후 대기열에서 메시지를 삭제합니다. 그러나 RDS 테이블에서 가끔 중복 레코드가 발견됩니다. 이 문제를 해결하기 위해 ChangeMessageVisibility API 호출을 사용하여 가시성 제한 시간을 늘려야 합니다. 이렇게 하면 같은 메시지가 다시 처리되는 것을 방지할 수 있습니다. 즉, ChangeMessageVisibility API 호출을 사용하여 가시성 제한 시간을 늘리면 메시지가 한 번만 처리되도록 할 수 있습니다.

  중복 레코드 문제는 메시지가 여러 EC2 인스턴스에 의해 동시에 처리될 때 발생할 가능성이 큽니다. 이 상황에서 가장 효과적인 해결책은 메시지 처리의 원자성을 보장하는 것입니다.

  정답인 D 옵션, ChangeMessageVisibility API는 이 문제를 해결하는 핵심 방법입니다. 이 API를 사용하면 특정 메시지의 가시성을 제한 시간 동안 숨김으로써, 한 번에 하나의 인스턴스만 해당 메시지를 처리하도록 할 수 있습니다. 즉, 메시지를 받은 인스턴스가 처리를 완료할 때까지 다른 인스턴스들이 해당 메시지를 볼 수 없게 됩니다. 이렇게 하면 중복 처리를 방지하고 데이터 일관성을 유지할 수 있습니다.

  다른 옵션들은 다음과 같이 적합하지 않습니다:

  A 옵션은 새 대기열 생성으로 문제 해결에 직접적인 연관성이 없습니다.
  B 옵션은 권한 설정으로 메시지 처리의 중복 문제를 해결하지 못합니다.
  C 옵션의 대기 시간 설정은 메시지 수신 순서나 지연에 영향을 줄 수 있지만, 중복 처리 방지에는 도움이 되지 않습니다.
  따라서 중복 메시지 처리 문제를 해결하기 위해 가장 적합한 방법은 D 옵션의 가시성 제한 시간 설정입니다.

68. A solutions architect is designing a new hybrid architecture to extend a company's on-premises infrastructure to AWS. The company requires a consistently low-latency, highly available connection to AWS regions. The company must minimize costs and be willing to tolerate slower traffic if the primary connection fails.
What should the solutions architect do to meet these requirements?
① A. Provision an AWS Direct Connect connection for the region. Provision a VPN connection as a backup in case the primary Direct Connect connection fails.
② B. Provision a VPN tunnel connection to the region for private connectivity. Provision a second VPN tunnel for private connectivity and as a backup in case the primary VPN connection fails.
③ C. Provision an AWS Direct Connect connection to Region 1. If the primary Direct Connect connection fails, provision a second Direct Connect connection in the same Region as the backup.
④ D. Provision an AWS Direct Connect connection for the region. If the primary Direct Connect connection fails, use the Direct Connect failover properties in the AWS CLI to automatically create a backup connection.

  이 문제는 AWS에서 온프레미스 인프라를 확장하는 방법을 묻는 것입니다. 회사에서는 지연 시간이 일관되게 낮고 가용성이 높은 연결이 필요하며 비용을 최소화해야 합니다. 또한 기본 연결이 실패할 경우 느린 트래픽을 수용할 수 있다고 합니다.

  이러한 요구 사항을 충족하기 위해 솔루션 아키텍트는 리전에 대한 AWS Direct Connect 연결을 프로비저닝해야 합니다. Direct Connect는 온프레미스 인프라와 AWS 리지를 연결하는 전용 네트워크 연결입니다. 지연 시간이 짧고 가용성이 높은 연결을 제공합니다.

  그러나 기본 Direct Connect 연결이 실패할 경우를 대비하여 백업 연결을 프로비저닝해야 합니다. 이 경우 VPN 연결을 프로비저닝하여 기본 연결이 실패할 경우 트래픽을 백업 연결로 전환할 수 있습니다. VPN 연결은 느리지만 비용이 더 저렴하고 기본 연결의 백업으로 사용하기 적합합니다.

  따라서 솔루션 아키텍트는 리전에 대한 AWS Direct Connect 연결을 프로비저닝하고 기본 Direct Connect 연결이 실패할 경우 백업으로 VPN 연결을 프로비저닝해야 합니다. 이 방법으로 회사에서는 지연 시간이 짧고 가용성이 높은 연결을 유지하면서 비용을 최소화할 수 있습니다.

  정답 1번이 적절한 이유는 다음과 같습니다.

  회사의 주요 요구사항은 낮은 지연 시간과 높은 가용성을 갖춘 연결을 확보하면서 비용을 최소화하는 것입니다. 여기에 기본 연결 실패 시 느린 트래픽을 수용할 수 있는 백업 옵션이 필요합니다.

  AWS Direct Connect (보기 1): 이 옵션은 고속, 저지연, 그리고 안정적인 연결을 제공하여 주요 트래픽을 처리하는데 이상적입니다. Direct Connect는 온프레미스와 AWS 간에 물리적인 연결을 제공하므로, 일반 인터넷 연결보다 성능이 우수하고 일관된 지연 시간을 보장합니다.

  백업으로 VPN 연결 활용: 기본 연결인 Direct Connect가 실패할 경우, 상대적으로 느린 속도지만 무료 또는 저렴한 비용으로 운영 가능한 VPN을 백업으로 사용함으로써 비용 효율성을 유지할 수 있습니다.

  다른 옵션들과 비교하면:

  보기 2와 3: 두 번째 또는 백업 연결로 동일한 또는 추가적인 고가의 Direct Connect 연결을 사용하면 비용이 크게 증가합니다. 이는 회사의 비용 최소화 목표와 충돌합니다.
  보기 4: 자동 장애 조치 기능은 Direct Connect에 직접적으로 존재하지 않으며, 현재 AWS 서비스 내에서 명시적으로 제공되지 않는 기능입니다.
  따라서, 성능과 가용성을 극대화하면서 비용을 최소화하고 기본 연결 실패 시의 유연성을 제공하는 1번 옵션이 가장 적합합니다.

69. A company runs a business-critical web application on Amazon EC2 instances behind an Application Load Balancer. The EC2 instances are in an Auto Scaling group. The application uses an Amazon Aurora PostgreSQL database deployed in a single Availability Zone. The company wants to increase the availability of the application while minimizing downtime and data loss.
What solution can meet these requirements with minimal operational effort?
① A. Deploy EC2 instances in different AWS regions. Redirect traffic using Amazon Route 53 health checks. Use Aurora PostgreSQL cross-region replication.
② B. Configure an Auto Scaling group to use multiple Availability Zones. Configure the database as Multi-AZ. Configure an Amazon RDS Proxy instance for the database.
③ C. Configure the Auto Scaling group to use a single Availability Zone. Create hourly snapshots of the database. Recover the database from the snapshot in case of failure.
④ D. Configure an Auto Scaling group to use multiple AWS Regions. Write application data to Amazon S3. Use S3 event notifications to start an AWS Lambda function to write data to a database.

  정답은 2번입니다.

  현재 아키텍처는 단일 가용 영역에 국한되어 있어 장애 발생 시 높은 위험에 노출될 수 있습니다. 2번 솔루션은 다음과 같은 이유로 가장 적합합니다:

  다중 가용 영역 구성: Auto Scaling 그룹을 여러 가용 영역에 걸쳐 설정함으로써, 한 가용 영역에 장애가 발생하더라도 다른 영역의 인스턴스가 서비스를 계속할 수 있어 가동 중지 시간을 최소화합니다.
  Amazon Aurora 다중 AZ 설정: 데이터베이스를 다중 가용 영역에 배치하여 데이터베이스 자체의 가용성을 향상시키고 데이터 손실 위험을 줄입니다. Aurora의 내장된 복제 기능 덕분에 높은 가용성과 내구성을 유지할 수 있습니다.
  RDS 프록시 인스턴스: RDS 프록시를 사용하면 추가적인 부하 분산과 보안 강화가 가능하지만, 주요 요구사항인 가용성 향상에 직접적으로 크게 기여하는 요소보다는 보완적인 역할을 합니다. 그럼에도 불구하고, 이 요소는 전체 시스템의 안정성과 효율성을 높이는 데 도움이 됩니다.
  이 솔루션은 최소한의 운영 노력으로 가용성을 크게 향상시키며, 데이터 손실과 가동 중지 시간을 최소화하는데 효과적입니다. 다른 옵션들에 비해 1번은 리전 간 복제로 복잡성을 증가시키고, 3번은 단일 가용 영역 내에서의 복구 전략으로 위험을 완전히 해소하지 못하며, 4번은 다중 리전 및 S3 기반의 복잡한 아키텍처로 운영 부담이 증가하기 때문입니다.

70. The company's HTTP application is behind a Network Load Balancer (NLB). The NLB's target group is configured to use an Amazon EC2 Auto Scaling group containing multiple EC2 instances running web services.
The company discovered that the NLB fails to detect HTTP errors for the application. These errors require manual restarts of the EC2 instances running the web services. The company needs to improve application availability without writing custom scripts or code.
What should a solution architect do to meet these requirements?
① A. Enable HTTP health checks in NLB by providing the URL of your company application.
② B. Add a cron job to your EC2 instance to check the local application logs once a minute. If an HTTP error is detected, the application will be restarted.
③ C. Replace NLB with an Application Load Balancer. Enable HTTP health checks by providing the URL of your company's application. Configure Auto Scaling to replace unhealthy instances.
④ D. Create an Amazon CloudWatch alarm that monitors the UnhealthyHostCount metric for NLB. Configure an Auto Scaling action to replace unhealthy instances when the alarm enters the ALARM state.

  회사의 HTTP 애플리케이션은 NLB 뒤에 있으며 NLB의 대상 그룹은 웹 서비스를 실행하는 여러 EC2 인스턴스가 있는 Amazon EC2 Auto Scaling 그룹을 사용하도록 구성되어 있습니다. 그런데 NLB가 애플리케이션에 대한 HTTP 오류를 감지하지 못하는 문제가 있습니다. 이를 해결하려면 NLB를 Application Load Balancer로 교체해야 합니다. Application Load Balancer는 NLB와 달리 HTTP 상태 확인을 지원하기 때문입니다. 회사의 애플리케이션 URL을 제공하여 HTTP 상태 확인을 활성화하고 비정상 인스턴스를 교체하도록 Auto Scaling 작업을 구성하면 사용자 정의 스크립트나 코드를 작성하지 않고도 애플리케이션의 가용성을 향상시킬 수 있습니다.

  NLB는 기본적으로 TCP 레벨에서만 연결을 모니터링하므로 HTTP 수준의 오류를 감지하지 못합니다. 따라서 주어진 문제 상황에서 사용자 정의 스크립트 없이 HTTP 오류를 감지하고 자동으로 해결하기 위한 최적의 방법은 다음과 같습니다:

  C번 옵션이 정답입니다. NLB를 Application Load Balancer (ALB)로 전환하는 것이 핵심입니다. ALB는 HTTP/HTTPS 프로토콜을 지원하여 URL 기반의 사용자 정의 상태 확인을 활성화할 수 있습니다. 이를 통해 HTTP 오류를 직접 감지하고 비정상 인스턴스를 식별할 수 있습니다. 또한, 이렇게 구성된 ALB와 연동된 Auto Scaling 그룹을 통해 비정상 인스턴스를 자동으로 교체할 수 있어 가용성을 향상시킬 수 있습니다. 이 접근법은 별도의 스크립트 작성 없이도 문제를 해결하는 효과적인 방법입니다. 다른 옵션들은 수동 개입이 필요하거나 NLB의 제한을 극복하지 못합니다.

600. A company plans to migrate its TCP-based applications to its VPC . The applications are publicly accessible via a non-standard TCP port through a hardware appliance in the company's data center. This public endpoint can handle up to 3 million requests per second with low latency . The company requires the same level of performance for its new AWS public endpoint.
What should the solution architect recommend to meet this requirement?
① A. Deploy a Network Load Balancer (NLB). Configure the NLB to be publicly accessible via the TCP ports required by your application.
② B. Deploy an Application Load Balancer (ALB). Configure the ALB to be publicly accessible via the TCP ports required by your application.
③ C. Deploy an Amazon CloudFront distribution that listens on the TCP ports required by your application. Use an Application Load Balancer as the origin.
④ D. Deploy an Amazon API Gateway API configured with the TCP ports required by your application. Configure an AWS Lambda function with provisioned concurrency to handle requests.

  이 상황에서 요구되는 것은 최단 시간에 최대 300만 개의 요청을 처리하는 것입니다. 이를 해결할 수 있는 방법은 NLB(Network Load Balancer)를 배포하는 것입니다. NLB는 고성능의 로드 밸런싱을 제공하여 짧은 대기 시간으로 초당 최대 300만 개의 요청을 처리할 수 있습니다. 또한 TCP 기반 애플리케이션을 지원하기 때문에 이 경우에 적합한 솔루션입니다. NLB를 통해 애플리케이션에 필요한 TCP 포트를 통해 공개적으로 액세스할 수 있도록 구성할 수 있습니다. 따라서 솔루션 설계자는 NLB를 배포하고 애플리케이션에 필요한 TCP 포트를 통해 공개적으로 액세스할 수 있도록 구성해야 합니다.

  정답은 1번입니다. NLB (Network Load Balancer)는 고성능과 낮은 지연 시간을 요구하는 TCP 기반 애플리케이션에 최적화되어 있습니다. 주요 이유는 다음과 같습니다:

  낮은 대기 시간: NLB는 단일 자릿수 밀리세컨드의 대기 시간을 제공하여 초당 수백만 건의 요청 처리를 요구하는 고성능 요건을 충족합니다.
  고정 IP 주소: 외부 클라이언트에게 안정적인 연결을 제공하며, 이는 퍼블릭 엔드포인트의 신뢰성과 연결성을 유지하는 데 중요합니다.
  TCP 프로토콜 지원: 문제에서 언급된 비표준 TCP 포트를 통해 액세스해야 하는 애플리케이션에 이상적입니다. ALB는 주로 HTTP/HTTPS 트래픽에 최적화되어 있습니다.
  대용량 처리 능력: NLB는 높은 연결 용량을 지원하여 초당 수많은 요청을 효과적으로 분산시킬 수 있습니다.
  반면, ALB는 주로 애플리케이션 레이어의 요청 처리와 HTTP/HTTPS 트래픽 최적화에 초점을 맞추고 있으며, ttlCloudFront는 주로 콘텐츠 배달 네트워크 용도로 사용되며, API Gateway와 Lambda 조합은 주로 RESTful API와 가벼운 백엔드 처리에 더 적합합니다. 따라서 주어진 성능 요구 사항을 만족시키는 가장 적합한 선택은 NLB의 배포입니다.

601. A company runs a critical database on an Amazon RDS for PostgreSQL DB instance. The company wants to migrate to Amazon Aurora PostgreSQL while minimizing downtime and data loss . What solution can meet these requirements with minimal operational overhead?
① A. Populate a new Aurora PostgreSQL DB cluster by creating a DB snapshot of the RDS for PostgreSQL DB instance.
② B. Create an Aurora read replica of the RDS for PostgreSQL DB instance. Promote the Aurora read replica to the new Aurora PostgreSQL DB cluster.
③ C. Migrate the database to an Aurora PostgreSQL DB cluster using data import from Amazon S3.
④ D. Back up the RDS database for PostgreSQL using the pg_dump utility. Restore the backup to the new Aurora PostgreSQL DB cluster.

  가동 중지 시간과 데이터 손실을 최소화하면서 Amazon Aurora PostgreSQL로 마이그레이션하는 방법은 RDS for PostgreSQL DB 인스턴스의 Aurora 읽기 전용 복제본을 생성하고, Aurora 읽기 복제를 새로운 Aurora PostgreSQL DB 클러스터로 승격하는 것입니다. 이 방법은 최소한의 운영 오버헤드와 데이터 손실 없이 마이그레이션을 수행할 수 있습니다.

  방법은 다음과 같은 이유로 적합하지 않습니다.

  DB 스냅샷을 생성하여 새로운 Aurora PostgreSQL DB 클러스터를 채우는 방법은 시간이 걸릴 수 있고, 데이터 손실이 발생할 수 있습니다.
  Amazon S3에서 데이터 가져오기를 사용하여 데이터베이스를 Aurora PostgreSQL DB 클러스터로 마이그레이션하는 방법은 복잡하고 시간이 걸릴 수 있습니다.
  pg_dump 유틸리티를 사용하여 PostgreSQL용 RDS 데이터베이스를 백업하고, 새 Aurora PostgreSQL DB 클러스터로 백업을 복원하는 방법은 시간이 걸릴 수 있고, 데이터 손실이 발생할 수 있습니다.
  따라서, 가동 중지 시간과 데이터 손실을 최소화하면서 Amazon Aurora PostgreSQL로 마이그레이션하는 솔루션은 RDS for PostgreSQL DB 인스턴스의 Aurora 읽기 전용 복제본을 생성하여 새로운 Aurora PostgreSQL DB 클러스터로 승격하는 것입니다.

  정답은 2번입니다.

  Aurora 읽기 전용 복제본을 생성하는 방법은 가동 중지 시간을 최소화하는 데 효과적입니다. 이 방법을 통해 원래 PostgreSQL 인스턴스는 계속 운영되면서 동시에 Aurora PostgreSQL 클러스터의 읽기 복제본을 통해 데이터의 일관성을 유지할 수 있습니다. 마이그레이션 과정에서 문제가 발생하거나 확인이 필요할 경우 안전하게 기존 시스템을 유지할 수 있으며, 준비가 완료되면 복제본을 주 노드로 승격시켜 서비스 중단을 최소화한 채로 마이그레이션을 완료할 수 있습니다.

  다른 옵션들과 비교했을 때:

  1번은 스냅샷 기반 마이그레이션으로, 일시적인 중지 시간이 발생할 수 있습니다.
  3번의 S3 데이터 마이그레이션은 복잡하고 시간이 오래 걸리며, 데이터 일관성 유지가 어려울 수 있습니다.
  4번의 pg_dump 및 복원 방식도 가동 중지 시간이 필요하며, 전체 프로세스가 수동적이고 복잡할 수 있습니다.
  따라서 최소한의 가동 중지 시간과 데이터 손실을 보장하며 운영 오버헤드를 최소화하는 최적의 선택은 2번입니다.

602. The company's infrastructure consists of hundreds of Amazon EC2 instances using Amazon Elastic Block Store (Amazon EBS) storage. A solutions architect must ensure that all EC2 instances can be recovered after a disaster . What should the solutions architect do to meet this requirement with minimal effort?
① A. Take a snapshot of the EBS storage attached to each EC2 instance. Create an AWS CloudFormation template to launch a new EC2 instance from the EBS storage.
② B. Take a snapshot of the EBS storage attached to each EC2 instance. Use AWS Elastic Beanstalk to set up an environment based on an EC2 template and attach the EBS storage.
③ C. Set up a backup plan for the entire EC2 instance group using AWS Backup. Using the AWS Backup API or AWS CLI can speed up the restore process for multiple EC2 instances.
④ D. Create an AWS Lambda function that takes a snapshot of the EBS storage attached to each EC2 instance and copies the resulting Amazon Machine Image (AMI). Create another Lambda function that restores the copied AMI and connects the EBS storage.

  재해 발생 후 모든 EC2 인스턴스를 복구할 수 있도록 하기 위해서는 전체 인프라의 백업이 필요합니다. 이를 위해 AWS Backup을 사용하여 전체 EC2 인스턴스 그룹에 대한 백업 계획을 설정하는 방법이 가장 적합합니다. Это 방법은 자동화된 백업 및 복원 프로세스를 제공하여 솔루션 아키텍트의 노력을 최소화하며, 간단한 설정으로 모든 인프라를 복구할 수 있습니다. 다른 옵션은 더 많은 수동 조작이 필요하거나 도중에 수동으로 EC2 인스턴스를 생성하고 구성해야 하므로, 이 방법은 최소한의 노력으로 요구 사항을 충족하는 가장 효과적인 방법입니다.

  최소한의 노력으로 모든 EC2 인스턴스를 재해 복구할 수 있는 가장 효율적인 방법은 옵션 C입니다.

  AWS Backup을 활용하면 수백 개의 인스턴스를 한 번에 관리할 수 있는 중앙 집중식 백업 계획을 쉽게 설정할 수 있습니다. 이는 수동으로 각 인스턴스의 스냅샷을 관리하는 것보다 훨씬 효율적입니다 (옵션 A, D의 경우).

  AWS Elastic Beanstalk (옵션 B)는 배포와 관리를 간소화하지만, 재해 복구를 위한 자동화된 백업 및 복원 프로세스를 제공하는 데는 AWS Backup만큼 적합하지 않습니다. Backup API나 CLI를 사용하면 복원 프로세스를 자동화하여 시간과 노력을 크게 절감할 수 있습니다. 따라서, 대규모 EC2 인스턴스 그룹의 재해 복구를 위한 최적의 선택은 AWS Backup을 이용한 중앙 집중식 백업 및 복원 계획 구축입니다.

603. A company recently migrated to the AWS Cloud. The company wants a serverless solution for large-scale, parallel, on-demand processing of semi-structured data sets. The data consists of logs, media files, sales transactions, and IoT sensor data stored in Amazon S3. The company wants a solution that can process thousands of items in the data set in parallel
. What solution can meet these requirements with the most efficient operational efficiency?
① A. Process data in parallel using AWS Step Functions map states in inline mode.
② B. Process data in parallel using AWS Step Functions map states in distributed mode.
③ C. Process data in parallel using AWS Glue.
④ D. Process data in parallel using multiple AWS Lambda functions.

  정답은 2번입니다.

  회사가 요구하는 것은 대규모 데이터 세트의 수천 개의 항목을 효율적으로 병렬 처리하는 서버리스 솔루션입니다.

  AWS Lambda는 이벤트 트리거에 반응하여 코드 실행을 제공하는 서버리스 컴퓨팅 서비스로, 병렬 처리에 적합합니다. 하지만 여러 Lambda 함수를 직접 관리하고 조율하는 것은 복잡할 수 있습니다.

  AWS Step Functions는 복잡한 워크플로우를 구성하고 관리하기 위한 서비스입니다.

  분산 모드에서 맵 상태를 사용하면 하나의 작업을 여러 Lambda 함수 인스턴스로 분산 처리하여 대규모 병렬 처리를 효과적으로 수행할 수 있습니다.
  인라인 모드는 각 작업을 순차적으로 처리하므로 병렬 처리에 적합하지 않습니다.
  AWS Glue는 데이터 카탈로그링 및 ETL(추출, 변환, 로드) 작업에 특화된 서비스입니다. 병렬 처리 기능을 제공하지만, 본 문제에서 제시된 주문형 병렬 처리보다는 데이터 파이프라인 구축에 더 적합합니다.

  따라서, 수천 개의 항목을 효율적으로 병렬 처리하기 위해서는 AWS Step Functions의 분산 모드 맵 상태를 활용하여 여러 Lambda 함수를 오케스트레이션하는 방식 (2번)이 가장 적합합니다.

604. The company plans to migrate 10 petabytes of data to Amazon S3 within six weeks. Its current data center has a 500 Mbps uplink to the internet. Other on-premises applications share this uplink. The company can use 80% of its internet bandwidth for this one-time migration.
Which solution meets these requirements?
① A. Configure AWS DataSync to migrate data to Amazon S3 and automatically verify the data.
② B. Use rsync to transfer data directly to Amazon S3.
③ C. Send data directly to Amazon S3 using the AWS CLI and multiple copy processes.
④ D. Order multiple AWS Snowball devices. Copy your data to the devices. Ship the devices to AWS to copy the data to Amazon S3.

  이 문제는 데이터를 Amazon S3로 마이그레이션하는 가장 효과적인 방법을 찾는 것입니다. 회사는 6주 안에 10PB의 데이터를 마이그레이션해야 합니다. 현재 인터넷 업링크는 500Mbps이고 다른 온프레미스 애플리케이션과 공유됩니다. 회사는 이 마이그레이션 작업에 인터넷 대역폭의 80%를 사용할 수 있습니다.

  마이그레이션 속도와 데이터 양을 계산하면 인터넷을 통해 직접 전송하는 것은 시간적으로 불가능함을 알 수 있습니다. 따라서 대용량 데이터 전송을 지원하는 솔루션이 필요합니다.

  이 경우 AWS Snowball을 사용하는 것이 적절한 솔루션입니다. AWS Snowball은 대용량 데이터를 Amazon S3로 전송하는 데 사용할 수 있는 물리적 장치입니다. 데이터를 Snowball 장치에 복사하고 AWS로 보내면 데이터를 Amazon S3에 복사할 수 있습니다. 이는 인터넷 대역폭에 의존하지 않고 대용량 데이터를 효율적으로 전송할 수 있는 방법입니다.

  따라서 이 문제의 정답은 4번입니다. 여러 AWS Snowball 디바이스를 주문하여 데이터를 장치에 복사하고, 디바이스를 AWS로 보내 데이터를 Amazon S3에 복사하는 것입니다.

  회사의 요구 사항은 10PB의 대규모 데이터를 6주 이내에 안전하고 효율적으로 Amazon S3로 마이그레이션하는 것입니다. 주어진 인터넷 업링크 대역폭은 한정적이며, 그 중 80%만 사용할 수 있어 속도는 약 400Mbps로 제한됩니다.

  A 옵션: AWS DataSync는 효율적인 데이터 동기화를 제공하지만, 대용량 데이터의 일회성 대규모 마이그레이션에 필요한 대역폭과 시간 제약 조건을 충족시키기 어렵습니다.
  B 옵션: rsync를 통한 직접 전송은 인터넷 속도의 한계로 인해 시간이 지나치게 오래 걸릴 것입니다.
  C 옵션: AWS CLI를 활용한 병렬 처리도 대역폭 제약으로 인해 마이그레이션 기간 내 완료가 어려울 것으로 예상됩니다.
  정답인 D 옵션은 AWS Snowball 디바이스를 활용합니다. Snowball은 대용량 데이터를 안전하게 물리적으로 운송할 수 있는 고성능 저장 장치로, 인터넷 대역폭 제약을 극복할 수 있습니다. 데이터 센터에서 디바이스에 데이터를 복사한 후 AWS로 보내면, AWS가 이를 처리하여 Amazon S3에 안전하게 업로드합니다. 이 방법은 시간과 대역폭 제약을 효과적으로 관리하면서 대규모 데이터 마이그레이션을 가능하게 합니다. 따라서, 주어진 상황에서 가장 적합한 솔루션은 여러 AWS Snowball 디바이스를 사용하는 것입니다.

604. A company collects 10 GB of telemetry data daily from various systems . The company stores the data in an Amazon S3 bucket in its source data account. The company has hired several consulting firms to analyze this data. Each firm requires read access to the data for its analysts . The company needs to share the data in the source data account by selecting a solution that maximizes security and operational efficiency. Which solution meets these requirements?
① A. Configure an S3 global table to replicate data from each organization.
② B. Make the S3 bucket public for a limited time. Notify only your organization.
③ C. Configure cross-account access to S3 buckets for accounts owned by your organization.
④ D. Set up an IAM user for each analyst in the source data account. Grant each user access to the S3 bucket.

  정답은 C입니다.

  회사의 요구 사항은 원본 데이터의 보안을 유지하면서 여러 컨설팅 기관에 효율적으로 읽기 액세스를 제공하는 것입니다.

  옵션 A는 데이터 복제를 필요로 하며, 이는 저장 공간과 관리 복잡성을 증가시킵니다.
  옵션 B는 일시적으로 버킷을 공개하는 방식으로, 보안 위험이 크고 지속적인 관리가 어렵습니다.
  옵션 D는 각 분석가에게 개별 IAM 사용자를 생성하고 권한을 부여하는 방법으로, 사용자 수가 많아지면 관리가 복잡해질 수 있습니다.
  반면, 옵션 C는 교차 계정 액세스를 통해 원본 S3 버킷의 데이터에 대한 읽기 권한을 여러 기관 계정으로 안전하게 공유할 수 있습니다. 이 방법은 중앙 집중적인 권한 관리가 가능하며, 보안 정책을 일관되게 적용할 수 있어 효율성과 보안을 동시에 극대화합니다. 따라서 회사의 요구 사항을 가장 잘 충족하는 솔루션입니다.

605. The company has several on-premises Internet Small Computer Systems Interface (ISCSI) network storage servers. The company wants to reduce the number of these servers by moving to the AWS cloud . The solution architect needs to provide low-latency access to frequently accessed data and reduce dependency on on-premises servers with minimal infrastructure changes.
Which solution meets these requirements?
① A. Deploy an Amazon S3 File Gateway.
② B. Deploy Amazon Elastic Block Store (Amazon EBS) storage with backups to Amazon S3.
③ C. Deploy an AWS Storage Gateway volume gateway configured with stored volumes.
④ D. Deploy an AWS Storage Gateway volume gateway configured with cached volumes.

  주요 이유는 다음과 같습니다. Storage Gateway 볼륨 게이트웨이는 캐시된 볼륨을 사용하여 iSCSI 블록 스토리지를 제공합니다. 이를 통해 최소한의 변경으로 온프레미스 iSCSI 서버를 교체할 수 있습니다. 캐시된 볼륨은 짧은 대기 시간 액세스를 위해 자주 액세스하는 데이터를 로컬에 저장하는 반면, 자주 액세스하지 않는 데이터는 S3에 저장합니다. 이를 통해 온프레미스 서버 수를 줄이는 동시에 핫 데이터에 대한 짧은 대기 시간 액세스를 제공합니다. EBS는 기존 서버를 대체하기 위한 iSCSI 지원을 제공하지 않습니다. S3 파일 게이트웨이는 블록 스토리지가 아닌 파일 스토리지용입니다. 저장된 볼륨은 S3가 아닌 온프레미스에 모든 데이터를 저장합니다.

  온프레미스 ISCSI 네트워크 스토리지 서버를 AWS 클라우드로 이동하면서 서버의 수를 줄이고 최소한의 인프라 변경으로 온프레미스 서버에 대한 종속성을 сниęp.DataAccess를 제공하는 솔루션은 AWS Storage Gateway 볼륨 게이트웨이입니다.

  이중 캐시된 볼륨으로 구성된 AWS Storage Gateway 볼륨 게이트웨이를 배포하는 것이 적절합니다. 이는 자주 사용되는 데이터에 대한 짧은 대기 시간 액세스를 제공하고 온프레미스 서버에 대한 종속성을 줄일 수 있습니다.

  캐시된 볼륨 게이트웨이는 자주 사용하는 데이터를 캐싱하여 빠른 액세스를 가능하게 하고, LESS 자주 사용되는 데이터는 AWS 클라우드 스토리지에 저장합니다. 따라서 이를 통해 서버의 수를 줄이고 온프레미스 서버에 대한 종속성을 최소한으로 유지할 수 있습니다.

606. A solutions architect is designing an application that allows business users to upload objects to Amazon S3. The solution must maximize object durability. Furthermore, objects must be readily available at all times. Users frequently access objects within the first 30 days after they are uploaded , but they are much less likely to access objects older than 30 days.
What is the most cost-effective solution to meet these requirements?
① A. Use S3 lifecycle rules to store all objects in S3 Standard and transition objects to S3 Glacier after 30 days.
② B. Use S3 lifecycle rules to store all objects in S3 Standard, transitioning objects to S3 Standard-Infrequent Access (S3 Standard-IA) after 30 days.
③ C. Store all objects in S3 Standard using an S3 lifecycle rule that transitions objects to S3 One Zone-Infrequent Access (S3 One Zone-IA) after 30 days.
④ D. Use S3 lifecycle rules to store all objects in S3 Intelligent-Tiering, transitioning objects to S3 Standard-Infrequent Access (S3 Standard-IA) after 30 days.

  S3 Standard-IA 또는 S3 One Zone-IA로 전환하는 데 필요한 최소 일수 객체를 S3 Standard-IA 또는 S3 One Zone-IA로 전환하기 전에 해당 객체를 Amazon S3에 최소 30일 동안 저장해야 합니다. 예를 들어, 객체를 생성한 지 하루 만에 객체를 S3 Standard-IA 스토리지 클래스로 전환하는 수명 주기 규칙을 생성할 수 없습니다. 새로운 객체는 S3 Standard-IA 또는 S3 One Zone-IA 스토리지에 적합한 것보다 더 자주 액세스되거나 더 빨리 삭제되는 경우가 많기 때문에 Amazon S3는 처음 30일 이내에 이러한 전환을 지원하지 않습니다. 마찬가지로, 버전이 지정된 버킷에서 이전 객체를 전환하는 경우 최소 30일 이상 이전 버전인 객체만 S3 Standard-IA 또는 S3 One Zone-IA 스토리지로 전환할 수 있습니다.

 객체 내구성을 극대화하고 쉽게 사용할 수 있어야 하며 처음 30일 이내에 자주 액세스하지만 이후에는 액세ス할 가능성이 적다. S3 Standard의 경우 자주 액세스하는 데이터에 적합하다. 하지만 30일 이후로는 S3 Standard-Infrequent Access로 전환하는 것이成本을 절감하면서도 적절한 내구성과 액세스를 제공할 수 있다. 따라서 S3 수명 주기 규칙을 사용하여 객체를 초기에 S3 Standard에 저장하고 30일 후에 S3 Standard-Infrequent Access로 전환하는 것이 가장 비용 효율적이다. 이 방식은 초기에 자주 액세스하는 데이터에 대해 높은 성능을 제공하면서도 이후에는成本을 절감할 수 있다. 또한 S3 Standard-Infrequent Access는 S3 Standard와 비슷한 내구성을 제공하므로 객체 내구성을 극대화할 수 있다. 따라서 이러한 요구 사항을 가장 비용 효율적으로 충족하는 솔루션은 B번이다.

  비즈니스 요구 사항에 따라 객체의 초기 액세스 빈도가 높고 시간이 지나면 액세스 빈도가 크게 감소하는 패턴을 고려할 때, 비용 효율성을 극대화하는 최적의 솔루션은 보기 2번입니다.

  S3 Standard: 객체를 처음 30일 동안 S3 Standard에 저장하여 빠른 액세스를 보장합니다.
  S3 Standard-IA로 전환: 30일 후에는 객체를 S3 Standard-Infrequent Access로 이동하여 저장 비용을 절감합니다. 이 옵션은 자주 접근하지 않는 데이터에 대해 더 저렴한 가격으로 저장할 수 있게 해줍니다.
  다른 옵션들과 비교했을 때:

  A 옵션 (S3 Glacier로 전환): 액세스 시간이 길어지며, 액세스 비용이 상대적으로 높아집니다.
  C 옵션 (S3 One Zone-IA로 전환): 더 저렴하지만 가용성과 복원 시간이 제한적일 수 있어 자주 액세스하는 초기 30일 동안 적합하지 않습니다.
  D 옵션 (S3 Intelligent-Tiering 사용): 자동 티어 전환 기능이 있지만, 초기 설정과 관리 복잡성이 증가하고, 30일 후 S3 Standard-IA로만 전환하는 경우에는 보기 2번보다 비용 효율성이 떨어질 수 있습니다.
  따라서, 초기 액세스 빈도와 장기 보관 비용을 균형 있게 관리하는 측면에서 보기 2번이 가장 적합합니다.

607. A company migrated a two-tier application from its on-premises data center to the AWS Cloud. The data tier is a multi-AZ deployment of Amazon RDS for Oracle with 12 TB of general-purpose SSD Amazon Elastic Block Store (Amazon EBS) storage. The application was designed to process and store database documents as binary large objects (BLOBs), with an average document size of 6 MB . Over time, the database size grew, leading to degraded performance and increased storage costs. The company needs to improve database performance and requires a solution with high availability and resiliency.
What is the most cost-effective solution to meet these requirements?
① A. Reduce the size of the RDS DB instance. Increase the storage capacity to 24 TiB. Change the storage type to magnetic.
② B. Increase the size of your RDS DB instance. Increase the storage capacity to 24 Ti. Change the storage type to Provisioned IOPS.
③ C. Create an Amazon S3 bucket. Update your application to store documents in the S3 bucket. Store object metadata in your existing database.
④ D. Create an Amazon DynamoDB table. Update your application to use DynamoDB. Migrate data from the Oracle database to DynamoDB using AWS Database Migration Service (AWS DMS).

  회사가 직면한 주요 문제는 데이터베이스의 성능 저하와 스토리지 비용 증가입니다. 현재 구조는 대용량 BLOB 데이터를 데이터베이스 내에 저장하고 있어, 이는 일반 데이터베이스의 용량과 성능 제한에 부딪힙니다.

  1번 옵션은 스토리지 유형을 마그네틱으로 변경하면 IO 성능이 저하되어 성능 개선에 부적절합니다.

  2번 옵션은 인스턴스와 스토리지 용량을 늘리는 것이지만, 여전히 대용량 BLOB 데이터를 데이터베이스 내에 저장하므로 장기적으로 비용 효율성과 성능 개선에 한계가 있습니다.

  3번 옵션은 정답입니다. Amazon S3 버킷을 활용하여 대형 BLOB 데이터를 저장하면 다음과 같은 이점이 있습니다:

  성능 향상: 데이터베이스 부하를 줄여 성능 개선
  비용 효과성: S3는 대용량 스토리지에 대해 경제적입니다.
  확장성: S3는 간단하게 확장 가능하며, 데이터 관리와 비용 최적화를 용이하게 합니다.
  메타데이터 관리: 데이터베이스에 BLOB의 메타데이터만 저장하면, 핵심 데이터 처리는 효율적으로 분산 관리됩니다.
  4번 옵션은 DynamoDB로의 전환이 가능하지만, BLOB 데이터 저장에 최적화된 솔루션이 아니며 전환 과정에서 복잡성과 잠재적인 마이그레이션 비용이 추가될 수 있습니다. 따라서, 현재 상황에서 가장 비용 효율적이고 성능을 개선하는 방법은 S3를 활용하는 3번 옵션입니다.

608. A company has an application that serves clients deployed in over 20,000 retail locations worldwide. The application consists of a backend web service exposed via HTTPS on port 443. The application is hosted on Amazon EC2 instances behind an Application Load Balancer (ALB). The retail stores communicate with the web application over the public internet. The company allows each retail store to register an IP address assigned by its local ISP.
The company's security team recommends strengthening the security of the application endpoints by restricting access to only the IP addresses registered by the retail store.
What should a solution architect do to meet these requirements?
① A. Associate an AWS WAF web ACL with an ALB. Filter traffic using the ALB's IP rule set. Update the IP addresses in the rule to include the registered IP addresses.
② B. Deploy AWS Firewall Manager to manage ALConfigure firewall rules to restrict traffic to AL. Modify the firewall rules to include registered IP addresses.
③ C. Store IP addresses in an Amazon DynamoDB table. Configure AWS Lambda authentication in ALB to verify that incoming requests come from registered IP addresses.
④ D. Configure a network ACL on the subnet containing the ALB's public interface. Update the ingress rule in the network ACL with an entry for each registered IP address.

  정답은 1번입니다. AWS WAF 웹 ACL을 ALB와 연결하여 요구 사항을 효과적으로 충족할 수 있습니다.

  A 옵션은 적절합니다. AWS WAF의 웹 액세스 제어 목록(Web ACL)을 ALB와 연동하면, IP 기반 규칙을 설정하여 특정 IP 주소만 허용할 수 있습니다. 등록된 소매점 IP 주소만 허용하도록 이 규칙을 구성하면, 애플리케이션의 보안이 강화됩니다.
  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  B 옵션: AWS Firewall Manager는 주로 다수의 계정과 리소스에 걸쳐 보안 규칙을 관리하는 데 유용하지만, 개별 ALB의 세밀한 IP 제한에는 적합하지 않습니다.
  C 옵션: DynamoDB와 Lambda를 사용해 인증을 구현하는 방법은 복잡하며, ALB 자체의 기본 기능을 넘어서는 추가적인 개발과 관리 부담이 있습니다.
  D 옵션: 네트워크 ACL은 서브넷 수준에서 트래픽을 제어하는데 사용되며, ALB의 특정 엔드포인트에 대한 세밀한 IP 접근 제어에는 적합하지 않습니다. 또한, ALB의 공용 인터페이스가 여러 IP를 통해 분산될 수 있어 각 IP 주소에 대해 일일이 규칙을 설정하는 것은 관리상 비효율적입니다.
  따라서, 가장 간단하고 효과적인 방법은 AWS WAF를 활용하는 1번 옵션입니다.

609. A company is building a data analytics platform on AWS using AWS Lake Formation. The platform collects data from various sources, such as Amazon S3 and Amazon RDS. The company needs a security solution to prevent access to data containing sensitive information
. What solution can meet these requirements with minimal operational overhead?
① A. Create an IAM role with permissions to access the Lake Formation table.
② B. Create data filters to implement row-level security and cell-level security.
③ C. Create an AWS Lambda function that removes sensitive information before Lake Formation collects the data.
④ D. Create an AWS Lambda function that periodically queries and removes sensitive information from the Lake Formation table.

  Lake Formation 데이터 필터를 사용하면 조건에 따라 데이터 테이블의 행이나 셀에 대한 액세스를 제한할 수 있습니다. 이를 통해 민감한 데이터에 대한 액세스를 방지할 수 있습니다. 데이터 필터는 Lake Formation 내에서 구현되며 추가 코딩이나 Lambda 기능이 필요하지 않습니다. 데이터를 사전 처리하거나 테이블을 제거하는 Lambda 함수에는 지속적인 개발 및 유지 관리가 필요합니다. IAM 역할은 행 또는 셀 수준 보안이 아닌 사용자 수준 권한만 제공합니다. 데이터 필터는 복잡한 사용자 정의 코드를 피하면서 최소한의 구성으로 Lake Formation 데이터에 대한 세부적인 액세스 제어를 제공합니다.

  회사는 AWS Lake Formation을 사용하여 데이터 분석 플랫폼을 구축하고 있습니다. 이 플랫폼은 Amazon S3 및 Amazon RDS와 같은 다양한 소스에서 데이터를 수집합니다. companys에는 민감한 정보가 포함된 데이터 부분에 대한 액세스를 방지하기 위한 보안 솔루션이 필요합니다.

  Lake Formation 테이블에 대한 액세스를 제어하고 민감한 정보에 대한 액세스를 방지하기 위해 행 수준 보안과 셀 수준 보안을 구현하는 것이 좋습니다. 이를 위해 데이터 필터를 만들 수 있습니다. 이렇게하면 Lake Formation 테이블에 저장된 데이터에 대한 액세스를 제어할 수 있고 민감한 정보에 대한 액세스를 방지할 수 있습니다.

  이 방법은 운영 오버헤드를 최소화하면서도 보안 요구 사항을 충족할 수 있습니다. 따라서 데이터 필터를 만드는 것이 최소한의 운영 오버헤드로 보안 요구 사항을 충족하는 최선의 솔루션입니다.

  정답은 B입니다. 행 수준 보안과 셀 수준 보안을 구현하는 데이터 필터를 만드는 것이 최적의 솔루션입니다.

  이 접근법은 민감한 정보에 대한 접근을 효과적으로 제한합니다.

  행 수준 보안: 특정 사용자나 그룹이 특정 데이터 행에 접근하는 것을 차단합니다. 예를 들어, 특정 부서 직원이 다른 부서의 민감한 데이터에 접근하지 못하도록 할 수 있습니다.
  셀 수준 보안: 개별 데이터 셀 내의 민감한 정보에 대한 접근을 제어합니다. 이는 특정 필드 내의 민감한 데이터를 숨기거나 마스킹하는 데 유용합니다.
  다른 옵션들과 비교했을 때:

  A는 기본적인 권한 관리에 불과해 세부적인 민감 정보 보호에는 한계가 있습니다.
  C는 데이터 수집 전에 민감 정보를 제거하는 방법이지만, 이미 저장된 민감 데이터에 대한 실시간 보호는 제공하지 못합니다.
  D는 주기적인 쿼리와 제거 방식으로, 실시간 보안 요구와 지속적인 데이터 변경에 대응하기 어렵고 오버헤드가 커질 수 있습니다.
  따라서, 최소 운영 오버헤드로 민감 정보에 대한 정밀한 액세스 제어를 제공하는 B 옵션이 가장 적합합니다.

610. A company deploys an Amazon EC2 instance running in a VPC. The EC2 instance loads source data into an Amazon S3 bucket for later processing . Compliance laws require that the data not be transmitted over the public internet . Servers in the company's on-premises data center consume the output from the application running on the EC2 instance.
Which solution meets these requirements?
① A. Deploy an interface VPC endpoint for Amazon EC2. Create an AWS Site-to-Site VPN connection between your company and the VPC.
② B. Deploy a gateway VPC endpoint for Amazon S3. Establish an AWS Direct Connect connection between your on-premises network and your VPC.
③ C. Establish an AWS Transit Gateway connection from your VPC to an S3 bucket. Create an AWS Site-to-Site VPN connection between your company and your VPC.
④ D. Set up a proxy EC2 instance with a route to the NAT gateway. Configure the proxy EC2 instance to fetch S3 data and serve it to your application instances.

  이 문제는 AWS 솔루션을 통한 데이터 전송 및 보안 요구 사항을 충족하는 해결책을 찾는 것입니다.

  조건 1: 회사의 EC2 인스턴스에서 소스 데이터를 Amazon S3 버킷에 로드해야 합니다. 조건 2: 데이터는 공용 인터넷을 통해 전송되어서는 안 됩니다.

  위 조건을 충족하는 해결책은 B입니다. Amazon S3용 게이트웨이 VPC 엔드포인트를 배포하면 EC2 인스턴스와 S3 버킷 간에 비공개 연결을 생성하여 조건 1과 2를 충족합니다. 또한 온프레미스 네트워크와 VPC 간에 AWS Direct Connect 연결을 설정하면 공용 인터넷을 통해 데이터를 전송할 필요가 없어지므로 보안을 강화할 수 있습니다.

  정답은 2번입니다. 이 솔루션은 데이터 보안과 직접 연결성 요구 사항을 모두 충족합니다:

  Amazon S3용 게이트웨이 VPC 엔드포인트: 이 설정은 VPC 내 EC2 인스턴스가 공개 인터넷을 통하지 않고 S3 버킷에 안전하게 접근할 수 있게 합니다. 따라서 데이터 전송이 공용 인터넷을 거치지 않도록 규정 준수를 보장합니다.

  AWS Direct Connect: 이는 온프레미스 데이터 센터와 AWS VPC 간에 고가속, 저지연 연결을 제공합니다. 이 연결을 통해 EC2 인스턴스의 애플리케이션 출력이 온프레미스 서버로 안전하고 효율적으로 전송될 수 있습니다.

  다른 옵션들은 요구 사항을 완전히 충족하지 못합니다:

  1번은 VPN을 사용하여 연결성을 제공하지만, S3에 대한 직접적인 VPC 엔드포인트 설정이 누락되어 인터넷을 통한 데이터 전송 위험이 남아 있습니다.
  3번의 Transit Gateway와 Site-to-Site VPN은 연결성을 제공하지만, S3에 대한 직접적인 보안 엔드포인트 설정이 부족합니다.
  4번의 NAT 게이트웨이와 프록시 EC2 인스턴스는 인터넷을 통한 아웃바운드 트래픽 관리에 도움이 될 수 있지만, S3와의 안전한 내부 통신과 온프레미스와의 고속 연결을 동시에 만족시키지 못합니다.
  따라서 요구 사항을 가장 효과적으로 충족하는 옵션은 2번입니다.

611. My company has an application with a REST-based interface that receives data from a third-party vendor in near real-time. Once received, the application processes and stores the data for further analysis. The application is running on an Amazon EC2 instance.
When the third-party vendor sends data to the application, it receives numerous 503 Service Unavailable errors. As data volumes surge, computing capacity is maxed out, and the application cannot handle all requests.
What design should a solution architect recommend to ensure a more scalable solution?
① A. Collect data using Amazon Kinesis Data Streams. Process the data using an AWS Lambda function.
② B. Use Amazon API Gateway on top of your existing applications. Create a usage plan with quota limits for third-party providers.
③ C. Collect data using Amazon Simple Notification Service (Amazon SNS). Deploy EC2 instances in an Auto Scaling group behind an Application Load Balancer.
④ D. Repackage the application into a container. Deploy the application using Amazon Elastic Container Service (Amazon ECS) with the EC2 launch type and an Auto Scaling group.

  Kinesis Data Streams는 대량의 스트리밍 데이터 수집 및 처리량을 처리할 수 있는 자동 조정 스트림을 제공합니다. 이렇게 하면 데이터 수신과 관련된 병목 현상이 제거됩니다.
  AWS Lambda는 EC2 용량 제한을 피하면서 확장 가능한 서버리스 방식으로 데이터를 처리하고 저장할 수 있습니다.
  API 게이트웨이는 API 관리 기능을 추가하지만 EC2 애플리케이션의 기본 확장성을 개선하지는 않습니다.
  SNS는 대규모 데이터 수집이 아닌 이벤트 게시/알림을 위한 것입니다. ECS는 여전히 EC2 용량에 의존합니다.

  이 상황은 데이터가 급격히 증가하여 컴퓨팅 용량이 최대치에 도달하면서 발생하는 문제입니다. 503 서비스 사용할 수 없음 오류가 자주 발생하는데, 이는 애플리케이션이 데이터 처리를 못해 생기는 문제입니다. 해결 방법은 데이터 처리能力을 향상시켜야 합니다.

  권장하는 디자인은 Amazon Kinesis Data Streams를 사용하여 데이터를 수집하고 AWS Lambda 함수를 사용하여 데이터를 처리하는 것입니다. Amazon Kinesis Data Streams는 대량의 데이터를 수집하고 처리할 수 있는 서비스로, 데이터가 많아지면 자동으로 처리 용량을 확장하여 데이터를 빠르게 처리할 수 있습니다. AWS Lambda 함수는 이벤트 驅動 방식으로 작동하여 데이터가 들어오면 자동으로 실행되어 데이터를 처리할 수 있습니다.

  이러한 설계로 인해 애플리케이션의 확장성을 향상시킬 수 있으며, 데이터 처리 능력을 향상시켜 503 오류를 줄일 수 있습니다. 또한, Amazon Kinesis Data Streams와 AWS Lambda 함수는 서버리스 아키텍처로, 컴퓨팅 용량을 관리할 필요가 없어 더욱 효율적입니다.

612. My company has an application running on an Amazon EC2 instance in a private subnet. The application needs to handle sensitive information in an Amazon S3 bucket. The application should not use the internet to connect to the S3 bucket. What solution meets this requirement?
① A. Configure an Internet gateway. Update your S3 bucket policy to allow access from the Internet gateway. Update your applications to use the new Internet gateway.
② B. Configure a VPN connection. Update your S3 bucket policy to allow access from the VPN connection. Update your applications to use the new VPN connection.
③ C. Configure a NAT gateway. Update your S3 bucket policy to allow access from the NAT gateway. Update your application to use the new NAT gateway.
④ D. Configure the VPC endpoint. Update the S3 bucket policy to allow access from the VPC endpoint. Update your application to use the new VPC endpoint.

  VPC 엔드포인트는 VPC로부터의 프라이빗 연결을 허용합니다. 인터넷 게이트웨이를 사용하지 않고 S3와 같은 AWS 서비스에 연결됩니다. 애플리케이션은 인터넷 액세스 없이 프라이빗 서브넷에 남아 있는 동안 VPC 엔드포인트를 통해 S3에 연결할 수 있습니다.

  올바른 솔루션은 D입니다. 이 방법은 프라이빗 서브넷의 애플리케이션이 인터넷을 통해 연결하지 않고도 Amazon S3 버킷에 연결할 수 있도록 해줍니다. VPC 엔드포인트를 사용하면 VPC 내에서 AWS 서비스에 대한 프라이빗 연결을 생성할 수 있습니다.

  이렇게하면 애플리케이션이 S3 버킷과 통신할 수 있게 됩니다. 또한, VPC 엔드포인트에서 액세스를 허용하도록 S3 버킷 정책을 업데이트하여 보안을 강화할 수 있습니다.

  인터넷 게이트웨이, VPN 연결, NAT 게이트웨이를 사용하는 다른 방법은 모두 인터넷을 통해 연결해야 하기 때문에 요구 사항을 충족하지 못합니다. 따라서, VPC 엔드포인트를 구성하는 것이 가장 적합한 솔루션입니다.

  프라이빗 서브넷 내의 EC2 인스턴스는 인터넷에 직접 노출되지 않도록 설계되어 민감한 정보에 대한 직접적인 인터넷 접근을 피해야 합니다. 옵션들을 살펴보면:

  옵션 1은 인터넷 게이트웨이를 사용하므로 인스턴스가 인터넷을 통해 S3에 접근하게 되어 요구 사항을 충족하지 못합니다.
  옵션 2의 VPN 연결은 보안을 강화하지만, 이 역시 인터넷을 통해 통신이 이루어지므로 적합하지 않습니다.
  옵션 3의 NAT 게이트웨이는 주로 프라이빗 서브넷의 인스턴스가 인터넷을 통해 아웃바운드 트래픽을 보내는 데 사용되며, S3와 같은 서비스에 대한 안전한 사설 네트워크 액세스를 직접 제공하지 않습니다.
  옵션 4의 VPC 엔드포인트는 VPC 내부에서 직접 S3에 연결할 수 있는 사설 경로를 제공합니다. 이를 통해 인스턴스는 인터넷을 통하지 않고 S3 버킷에 안전하게 접근할 수 있습니다. 따라서 VPC 엔드포인트 구성이 요구 사항을 완벽하게 충족합니다.

613. A company uses Amazon Elastic Kubernetes Service ( Amazon EKS ) to run container applications. The EKS cluster stores sensitive information in Kubernetes secret objects. The company wants to ensure that the information is encrypted
. What solution meets these requirements with minimal operational overhead?
① A. Encrypt information using AWS Key Management Service (AWS KMS) using container applications.
② B. Enable secret encryption on your EKS cluster using AWS Key Management Service (AWS KMS).
③ C. Implement an AWS Lambda function that encrypts information using AWS Key Management Service (AWS KMS).
④ D. Use AWS Systems Manager Parameter Store to encrypt information using AWS Key Management Service (AWS KMS).

  EKS는 AWS KMS 키를 사용하여 클러스터 수준에서 Kubernetes 암호 암호화를 지원합니다. 이는 비밀을 암호화하는 자동화된 방법을 제공합니다. 이 기능을 활성화하려면 EKS 클러스터에 대한 구성 변경을 최소화해야 하며 코드는 변경하지 않아도 됩니다. Lambda 함수를 사용하거나 애플리케이션 코드를 수정하여 비밀을 암호화하는 등의 다른 옵션에는 추가 개발 노력과 오버헤드가 필요합니다. Systems Manager Parameter Store는 암호화된 매개변수를 저장할 수 있지만 Kubernetes 비밀을 암호화하기 위해 기본적으로 EKS와 통합되지는 않습니다. EKS 비밀 암호화 기능은 애플리케이션에서 KMS API를 직접 호출할 필요 없이 AWS KMS를 활용합니다.

  회사의 요구사항은 EKS 클러스터에서 민감한 정보가 암호화되었는지 확인하는 것입니다. 이 요구사항을 충족하면서 최소한의 운영 오버헤드를 갖는 솔루션은 AWS Key Management Service(AWS KMS)를 사용하여 EKS 클러스터에서 비밀 암호화를 활성화하는 것입니다. 이 방법은 EKS 클러스터에서 직접 비밀을 암호화할 수 있으므로 추가적인 리소스나 구성이 필요 없습니다. 또한 AWS KMS는 키를 안전하게 관리하고 암호화를 제공하는 AWS 관리형 서비스이므로 운영 오버헤드가 줄어듭니다.

614. A company is designing a new multi-tier web application consisting of the following components:

• Web and application servers running on Amazon EC2 instances as part of an Auto Scaling group
• Amazon RDS DB instances for data storage

The solution architect needs to restrict access to the application servers so that only the web servers can access them.
Which solution meets these requirements?
① A. Deploy AWS PrivateLink in front of your application servers. Configure network ACLs to ensure that only web servers can access the application servers.
② B. Deploy a VPC endpoint in front of the application server. Configure a security group to ensure that only the web server can access the application server.
③ C. Deploy a Network Load Balancer to a target group that includes the Auto Scaling group for the application servers. Configure a network ACL to ensure that only the web servers can access the application servers.
④ D. Deploy an Application Load Balancer with a target group that includes the Auto Scaling group for the application servers. Configure a security group to ensure that only the web servers can access the application servers.

  ALB(Application Load Balancer)는 트래픽을 애플리케이션 서버로 전달하고 보안 그룹을 통해 액세스 제어를 제공합니다. 보안 그룹은 인스턴스 수준에서 방화벽 역할을 하며 웹 서버에서 애플리케이션 서버에 대한 액세스를 제어할 수 있습니다. 네트워크 ACL은 서브넷 수준에서 작동하며 인스턴스 수준 액세스 제어를 위한 보안 그룹의 경우 유연성이 떨어집니다. VPC 엔드포인트는 EC2 인스턴스 간 액세스가 아닌 AWS 서비스에 대한 프라이빗 액세스를 제공하는 데 사용됩니다. AWS PrivateLink는 이 단일 VPC 시나리오에서는 필요하지 않은 VPC 간의 프라이빗 연결을 제공합니다.

  해당 요구 사항을 충족하는 솔루션은 애플리케이션 서버의 Auto Scaling 그룹이 포함된 대상 그룹과 함께 Application Load Balancer를 배포하는 것입니다. 이를 통해 웹 서버만 애플리케이션 서버에 액세스할 수 있도록 보안 그룹을 구성할 수 있습니다. Application Load Balancer는 들어오는 요청을 웹 서버로 분산시키고, 웹 서버만 애플리케이션 서버에 액세스할 수 있도록 하는 데 도움이 됩니다. 보안 그룹을 구성하여 애플리케이션 서버에 대한 액세스를 제한할 수 있습니다.

  정답은 4번입니다. 애플리케이션 서버에 대한 접근을 웹 서버로만 제한하려면 다음과 같은 접근이 필요합니다:

  Application Load Balancer (ALB): 웹 서버와 애플리케이션 서버 간의 트래픽을 관리하며, 세밀한 라우팅 규칙을 설정할 수 있어요. 특정 서버나 그룹으로의 접근을 제어하는 데 유용합니다.

  보안 그룹 구성: ALB의 후단에 위치한 애플리케이션 서버의 보안 그룹을 적절히 설정하여 웹 서버 IP 주소만 애플리케이션 서버에 접근할 수 있도록 제한합니다. 이를 통해 외부 또는 다른 내부 서버의 무단 접근을 방지합니다.

  1번과 2번 옵션들은 주로 프라이빗 연결이나 VPC 엔드포인트를 강조하지만, 애플리케이션 서버에 대한 세밀한 액세스 제어가 필요한 이 문제에서는 직접적인 트래픽 관리와 보안 설정이 더 적합합니다. 3번은 Network Load Balancer를 사용하며, 주로 TCP/UDP 트래픽에 최적화되어 있어 애플리케이션 레벨의 세밀한 라우팅 제어가 부족합니다. 따라서, 애플리케이션 서버의 접근 제어를 효과적으로 구현하려면 4번의 Application Load Balancer와 보안 그룹 조합이 가장 적합합니다.

615. A company runs a critical customer-facing application on Amazon Elastic Kubernetes Service (Amazon EKS). The application has a microservices architecture. The company needs to implement a solution to collect, aggregate, and summarize application metrics and logs in a central location.
What solution meets these requirements?
① A. Run the Amazon CloudWatch agent on your existing EKS cluster. View metrics and logs in the CloudWatch console.
② B. Run AWS App Mesh on your existing EKS cluster. Check metrics and logs in the App Mesh console.
③ C. Configure AWS CloudTrail to capture data events. Query CloudTrail using Amazon OpenSearch Service.
④ D. Configure Amazon CloudWatch Container Insights on your existing EKS cluster. View metrics and logs in the CloudWatch console.

CloudWatch Container Insights를 사용하여 컨테이너화된 애플리케이션 및 마이크로서비스에서 지표와 로그를 수집, 집계 및 요약합니다

  한 회사가 Amazon Elastic Kubernetes Service(Amazon EKS)에서 고객을 대상으로 하는 중요한 애플리케이션을 실행하고 있고, 이 애플리케이션에는 마이크로서비스 아키텍처가 있습니다. 회사는 중앙 위치에서 애플리케이션의 측정항목과 로그를 수집, 집계, 요약하는 솔루션을 구현해야 합니다.

  이러한 요구 사항을 충족하는 솔루션은 기존 EKS 클러스터에 Amazon CloudWatch Container Insights를 구성하는 것입니다. CloudWatch Container Insights를 사용하면 컨테이너의 성능을 모니터링하고, 로그를 수집하며, 문제를 진단할 수 있습니다. CloudWatch 콘솔에서 지표와 로그를 확인하면 중앙 위치에서 애플리케이션의 성능과 문제를 모니터링할 수 있습니다.

  따라서 정답은 D입니다. Amazon CloudWatch Container Insights는 컨테이너화된 애플리케이션의 성능 모니터링과 문제 진단을 위한 강력한 도구입니다.

  정답은 4번입니다.

  Amazon CloudWatch Container Insights는 EKS 환경 내의 컨테이너화된 애플리케이션에 특화된 모니터링 솔루션입니다. 이 솔루션은 마이크로서비스 아키텍처를 갖춘 애플리케이션의 성능 지표와 로그를 자동으로 수집, 집계, 요약합니다. 특히, Kubernetes 환경의 복잡성을 고려해 최적화되어 있어, 중앙 집중식 관리와 분석이 용이합니다.

  1번 옵션의 Amazon CloudWatch 에이전트는 일반적인 모니터링에 효과적이나, 컨테이너와 EKS 환경에 특화된 심층적인 인사이트를 제공하는 데는 한계가 있습니다.

  2번의 AWS App Mesh는 주로 서비스 간 통신 관리와 모니터링에 초점을 맞추고 있어, 전체 애플리케이션의 측정항목과 로그의 종합적인 수집과 집계에는 적합하지 않습니다.

  3번의 AWS CloudTrail은 주로 API 호출 활동을 기록하는 데 사용되며, 애플리케이션의 실시간 성능 지표와 로그 수집에 최적화되어 있지 않습니다. Amazon OpenSearch Service는 로그 분석에 유용하지만, CloudTrail 데이터를 직접적으로 활용하는 것은 주어진 문제의 요구사항에 완벽하게 부합하지 않습니다.

  따라서, 마이크로서비스 기반 애플리케이션의 측정항목과 로그를 효과적으로 중앙에서 수집하고 분석하려면 Amazon CloudWatch Container Insights가 가장 적합합니다.

616. A company has deployed its latest product to AWS. The product runs in an Auto Scaling group behind a Network Load Balancer. The company stores the product's objects in an Amazon S3 bucket.
The company recently experienced a malicious attack on its systems. The company needs a solution that continuously monitors its AWS account for malicious activity, workloads, and access patterns to the S3 bucket . The solution also needs to report suspicious activity and display the information on a dashboard.
Which solution meets these requirements?
① A. Configure Amazon Macie to monitor results and report them to AWS Config.
② B. Configure Amazon Inspector to monitor results and report them to AWS CloudTrail.
③ C. Configure Amazon GuardDuty to monitor findings and report to AWS Security Hub.
④ D. Configure AWS Config to monitor results and report to Amazon EventBridge.

  Amazon GuardDuty는 악의적인 활동과 무단 행동을 지속적으로 모니터링하는 위협 탐지 서비스입니다. AWS CloudTrail, VPC 흐름 로그 및 DNS 로그를 분석합니다. GuardDuty는 인스턴스 또는 S3 버킷 손상, 악성 IP 주소 또는 비정상적인 API 호출과 같은 위협을 탐지할 수 있습니다. 중앙 집중식 보안 대시보드와 알림을 제공하는 AWS Security Hub로 결과를 전송할 수 있습니다. Amazon Macie 및 Amazon Inspector는 GuardDuty가 수행하는 활동의 범위를 모니터링하지 않습니다. 그들은 각각 데이터 보안과 애플리케이션 취약성에 더 중점을 둡니다. AWS Config는 악의적인 활동이 아닌 리소스 구성 변경을 모니터링합니다.

  정답은 3번입니다.

  Amazon GuardDuty는 AWS 계정, 네트워크, 워크로드, 그리고 서비스 활동을 지속적으로 모니터링하여 악의적인 행동이나 보안 위협을 탐지합니다. 이 솔루션은 다음과 같은 특징을 가지고 있어 문제의 요구 사항을 완벽하게 충족합니다:

  다양한 위협 감지: 네트워크 트래픽, S3 버킷 액세스 패턴, 그리고 Auto Scaling 그룹 내의 워크로드 활동 등을 포함한 다양한 영역에서 의심스러운 활동을 감지합니다.
  자동 보고 및 통합: 감지된 위협과 이상 징후를 AWS Security Hub에 자동으로 보고합니다. Security Hub는 이러한 정보를 통합된 대시보드에서 제공하여 관리자가 쉽게 모니터링하고 대응할 수 있게 합니다.
  실시간 모니터링: 지속적인 모니터링 기능으로 실시간으로 위협을 감지하고 알림을 제공하여 신속한 대응이 가능합니다.
  다른 옵션들과 비교했을 때:

  Amazon Macie는 주로 S3 버킷 내의 민감 데이터 보호에 초점을 맞춥니다.
  Amazon Inspector는 주로 EC2 인스턴스의 보안 취약성을 스캔하는 데 중점을 둡니다.
  AWS Config는 리소스의 구성 변경을 추적하고 감사하는데 유용하지만, 직접적인 위협 감지와 실시간 모니터링 기능은 제한적입니다.
  따라서, 종합적인 보안 모니터링과 의심스러운 활동의 자동 보고 및 대시보드 표시를 요구하는 상황에서는 Amazon GuardDuty가 가장 적합합니다.

617. A company is planning to migrate its on-premises data center to AWS. The data center hosts storage servers that store data on an NFS-based file system. The storage servers contain 200 GB of data. The company needs to migrate the data without disrupting existing services. Multiple resources in AWS must be able to access the data using the NFS protocol.
What combination of steps will most cost-effectively meet these requirements? (Choose two.)
① A. Create an Amazon FSx for Lustre file system.
② B. Create an Amazon Elastic File System (Amazon EFS) file system.
③ C. Create an Amazon S3 bucket to receive the data.
④ E. Install the AWS DataSync agent in your on-premises data center. Use DataSync tasks between your on-premises location and AWS.
⑤ E. Install the AWS DataSync agent in your on-premises data center. Use DataSync tasks between your on-premises location and AWS.

  Amazon EFS는 AWS의 여러 리소스에서 액세스할 수 있는 확장 가능한 고성능 NFS 파일 시스템을 제공합니다. AWS DataSync는 기존 서비스를 중단하지 않고 온프레미스 NFS 서버에서 EFS로 마이그레이션을 수행할 수 있습니다. 이렇게 하면 가동 중지 시간을 유발할 수 있는 데이터를 수동으로 이동할 필요가 없습니다. DataSync는 변경된 데이터를 점진적으로 동기화합니다. EFS와 DataSync는 요구 사항을 충족하면서도 S3 또는 FSx를 사용할 때보다 비용 최적화된 접근 방식을 제공합니다. 200GB의 데이터를 AWS에 수동으로 복사하는 것은 DataSync를 사용하는 것에 비해 느리고 위험합니다.

  AWS에서 제공하는 서비스 중에 NFS 프로토콜을 지원하는 서비스는 Amazon Elastic File System(Amazon EFS)와 Lustre 파일 시스템용 Amazon FSx가 있습니다. 하지만 데이터를 마이그레이션할 때 기존 서비스를 중단하지 않고 데이터를 마이그레이션해야 하므로 AWS DataSync를 사용하여 온프레미스 데이터 센터와 AWS 간에 데이터를 마이그레이션할 수 있습니다. 비용 효율적으로 데이터를 마이그레이션하기 위해서는 AWS DataSync 에이전트를 설치하여 데이터를 마이그레이션하고 NFS 프로토콜을 지원하는 Amazon EFS 파일 시스템을 생성하여 AWS의 여러 리소스가 데이터에 액세스할 수 있게끔 하면 됩니다. 따라서 2번과 4번이 정답입니다.

  주어진 요구 사항을 충족하면서 비용 효율성을 고려할 때, 주요 포인트는 서비스 중단 없이 데이터를 마이그레이션하고 NFS 프로토콜을 통한 접근성을 유지하는 것입니다.

  Amazon FSx (Lustre or EFS) 옵션들은 NFS 지원은 가능하지만, 기존 스토리지를 운영 중인 상태에서의 점진적 마이그레이션 과정에서 추가 장애 포인트를 유발하거나 초기 설정 비용이 높을 수 있습니다. 특히, 기존 NFS 환경을 그대로 유지하면서 바로 접근 가능하게 전환하기에는 제약이 있을 수 있습니다.

  Amazon S3는 고성능 파일 시스템 액세스에 최적화되지 않았으며, NFS 프로토콜을 직접 지원하지 않기 때문에, 기존 시스템이 직접 접근하기 위한 중간 계층 도입이 필요해지면 복잡성과 추가 비용이 발생할 수 있습니다.

  AWS DataSync (보기 4와 중복)를 사용하면, 온프레미스 환경에서 AWS로의 안정적이고 효율적인 데이터 마이그레이션이 가능합니다. 특히, DataSync는 중단 없이 데이터를 전송하고, 후에 Amazon EFS와 같은 서비스로의 전환을 용이하게 할 수 있습니다. 초기 설정 후에는 비용 효율적인 지속적인 데이터 동기화를 제공하며, NFS 프로토콜에 의존하는 시스템의 요구사항을 충족시키는 후속 단계로 EFS 등의 파일 시스템 도입이 가능합니다.

  따라서, 정답은 4번 (AWS DataSync 에이전트 설치 및 DataSync 작업 사용)이 가장 적합합니다. 이 방법은 초기 안정적인 마이그레이션과 유지 비용 효율성을 동시에 고려한 접근법을 제공합니다. 필요에 따라 EFS와 같은 다른 AWS 파일 서비스로의 후속 전환도 용이합니다. 다른 정답으로는 직접 NFS 지원과 마이그레이션 후의 파일 시스템 관리 측면에서 비슷한 유연성을 제공하지 않는 옵션들이므로, 주어진 조건에서는 4번 외의 추가 선택은 적절하지 않습니다.

618. A company wants to use Amazon FSx for Windows File Server on an Amazon EC2 instance with an SMB file share mounted as a volume in the us-east-1 region. The company has a recovery point objective (RPO) of 5 minutes for planned system maintenance or unplanned service outages. The company needs to replicate the file system to the us-west-2 region. The replicated data must not be deleted by any user for 5 years.
Which solution meets these requirements?
① A. Create an FSx for Windows File Server file system in us-east-1 using the Single-AZ 2 deployment type. Create a daily backup plan with a backup rule that copies backups to us-west-2 using AWS Backup. Configure AWS Backup Vault Lock in compliance mode for the target vault in us-west-2. Set the minimum duration to 5 years.
② B. Create an FSx for Windows File Server file system in us-east-1 with a multi-AZ deployment type. Create a daily backup plan with a backup rule that copies backups to us-west-2 using AWS Backup. Configure AWS Backup Vault Lock in governance mode for the destination vault in us-west-2. Set the minimum duration to 5 years.
③ C. Create an FSx for Windows File Server file system in us-east-1 with a multi-AZ deployment type. Create a daily backup plan with a backup rule that uses AWS Backup to copy backups to us-west-2. Configure AWS Backup Vault Lock in compliance mode for the target vault in us-west-2. Set the minimum duration to 5 years.
④ D. Create an FSx for Windows File Server file system in us-east-1 with a single-AZ 2 deployment type. Create a daily backup plan with a backup rule that uses AWS Backup to copy backups to us-west-2. Configure AWS Backup Vault Lock in governance mode for the destination vault in us-west-2. Set the minimum duration to 5 years.

us-east-1 리전에 Amazon FSx for Windows File Server를 사용하여 데이터를 저장하고 us-west-2 리전에 복제해야 합니다. 또한 5년 동안 삭제되지 않도록 모든 복제된 데이터를 보존해야 합니다.

  이 요구 사항을 충족하려면 다중 AZ 배포 유형을 사용하여 높은 가용성과 내구성을 제공하는 us-east-1에 FSx for Windows File Server 파일 시스템을 생성해야 합니다. AWS Backup을 사용하면 일일 백업 계획을 생성하여 us-west-2에 백업을 복사할 수 있습니다.

  중요한 점은 us-west-2의 대상 볼트에 대해 규정 준수 모드로 AWS Backup Vault Lock을 구성하여 백업 데이터의 무단 삭제나 수정을 방지하는 것입니다. 규정 준수 모드는 최소 보존 기간을 5년으로 설정하여 필요한 기간 동안 데이터를 보존하도록 합니다.

  따라서 다중 AZ 배포 유형을 사용하여 us-east-1에 FSx for Windows File Server 파일 시스템을 생성하고, AWS Backup을 사용하여 일일 백업 계획을 생성하여 us-west-2에 백업을 복사하고, us-west-2의 대상 볼트에 대해 규정 준수 모드로 AWS Backup Vault Lock을 구성하며, 최소 기간을 5년으로 구성하는 것이 적절한 솔루션입니다.

  회사의 요구 사항을 충족하기 위해서는 다음과 같은 요소들이 필요합니다:

  안정적인 다중 AZ 배포로 인한 가용성 향상
  5분 이하의 RPO를 지원하는 꾸준한 백업 및 복제 시스템
  5년 동안 데이터 삭제 방지를 위한 엄격한 규정 준수

  다중 AZ 배포는 시스템의 가용성을 높이고 중대한 중단 시에도 복구를 용이하게 하므로, 단일 AZ 배포보다 다중 AZ 배포 유형 (보기 1, 4 제외)이 적합하다.
  백업 및 복제: AWS Backup을 활용해 us-east-1의 FSx 파일 시스템을 us-west-2로 일일 백업을 복사하는 것은 요구 사항을 만족시킨다. 모든 옵션이 이 부분을 충족한다.
  데이터 보존 정책: 5년 동안 데이터 삭제를 방지하기 위해 규정 준수 모드가 필요하며, 이는 보다 엄격한 데이터 보호를 제공한다. 거버넌스 모드보다 규정 준수 모드가 더 강력한 보호를 제공하므로, 옵션 3만이 모든 요구 사항을 충족한다.
  따라서, 정답은 3번입니다. 이 옵션은 다중 AZ 배포를 통해 가용성을 확보하고, AWS Backup과 규정 준수 모드의 AWS Backup Vault Lock을 통해 RPO와 데이터 보존 기간을 충족시킵니다.

619. A solutions architect is designing a security solution for a company that wants to provide individual AWS accounts to developers through AWS Organizations while maintaining standard security controls. Because individual developers will have root user-level access to their accounts, the solutions architect wants to ensure that the required AWS CloudTrail configurations applied to the new developer accounts are not modified.
What actions can be taken to meet this requirement?
① A. Create an IAM policy that prohibits changes to CloudTrail. Attach it to the root user.
② B. Create a new trail in CloudTrail within your developer account with the Organizational Trail option enabled.
③ C. Create a Service Control Policy (SCP) that prohibits changes to CloudTrail and attach it to your developer account.
④ D. Create a service-linked role for CloudTrail with a policy condition that allows changes only to the Amazon Resource Name (ARN) of the master account.

  새 개발자 계정에 적용되는 필수 AWS CloudTrail 구성이 수정되지 않았는지 확인하려면 CloudTrail 변경을 금지하는 서비스 제어 정책(SCP)을 생성하고 이를 개발자 계정에 연결해야 합니다.这样하면 개발자가 자신의 계정에서 CloudTrail 설정을 수정할 수 없게 됩니다. 이것이 SCP의 목적이기 때문입니다. 따라서 서비스 제어 정책을 생성하고 이를 개발자 계정에 연결함으로써 솔루션 아키텍트의 요구 사항을 충족할 수 있습니다.

  정답은 3번입니다. 회사가 AWS Organizations를 통해 여러 개발자 계정을 관리하며, 각 개발자에게 루트 사용자 수준의 액세스 권한을 부여하면서도 CloudTrail 설정이 무단으로 변경되지 않도록 보호하려면, 서비스 제어 정책 (SCP)을 활용하는 것이 가장 효과적입니다.

  보기 1은 IAM 정책을 사용하나, 이는 개별 계정 수준에서만 적용되어 조직 전체의 일관된 제어를 제공하지 못합니다.
  보기 2는 개발자 계정 내에서 CloudTrail 추적을 설정하는 것으로, 변경 방지 기능이 포함되어 있지 않습니다.
  보기 4는 특정 ARN에 대한 접근 제한을 설정하지만, 서비스 제어와 조직 전체의 정책 적용 측면에서 SCP보다 제한적입니다.
  따라서 3번의 SCP는 AWS Organizations 수준에서 강력한 정책 통제를 제공하여, 조직 내 모든 개발자 계정에서 CloudTrail 설정이 무단으로 수정되지 않도록 강제할 수 있습니다. 이는 요구사항에 가장 적합한 접근 방식입니다.

620. A company plans to deploy a business-critical application on the AWS Cloud. The application requires durable storage with consistent, low-latency performance . What type of storage should the solution architect recommend to meet these requirements?
① A. Instance Store Volume
② B. Amazon ElastiCache for Memcached Clusters
③ C. Provisioned IOPS SSD Amazon Elastic Block Store (Amazon EBS) volume
④ D. Throughput-optimized HDD Amazon Elastic Block Store (Amazon EBS) volumes

  프로비저닝된 IOPS SSD — 미션 크리티컬하고 지연 시간이 짧으며 처리량이 높은 워크로드에 고성능을 제공합니다.
  처리량 최적화 HDD — 자주 액세스하고 처리량 집약적인 워크로드를 위해 설계된 저가형 HDD입니다.

  빌드업된 IOPS SSD 아마존 탄력적 블록 저장소 볼륨은 일관되고 지연 시간이 짧은 성능을 제공하여 비즈니스에 중요한 애플리케이션에 적합합니다. 이 유형의 저장소는 높은 성능과 내구성을 제공하므로 애플리케이션에 필요한 일관된 성능을 보장합니다. 또한 IOPS를 프로비저닝하여 특정 워크로드에 필요한 성능을 미리 설정할 수 있습니다. 따라서, 이 유형의 저장소가 최선의 선택입니다. 다른 옵션은 일관된 성능과 내구성을 제공하지 않거나, 추가 구성이 필요하여 적합하지 않습니다.

  해당 애플리케이션은 일관되고 짧은 지연 시간과 높은 내구성을 필요로 합니다. 각 옵션을 분석해보면:

  인스턴스 스토어 볼륨: 빠른 성능을 제공하지만, 내구성이 낮고 가용성 문제가 있을 수 있으며, 호스트 인스턴스의 수명에 종속됩니다.
  Memcached 클러스터용 Amazon ElastiCache: 주로 캐싱 용도로 사용되며, 데이터의 영구 저장보다는 빠른 읽기/쓰기 성능에 초점을 맞춥니다.
  프로비저닝된 IOPS SSD Amazon Elastic Block Store (EBS) 볼륨: SSD 기반으로 매우 낮은 지연 시간과 높은 IOPS(Input/Output Operations Per Second)를 제공하여 고성능과 내구성을 동시에 충족합니다.
  처리량 최적화 HDD EBS 볼륨: 비용 효율적이지만, SSD에 비해 지연 시간이 길고 IOPS가 낮아 성능 요구사항을 충족시키기 어렵습니다.
  특별시된 성능 요구사항, 특히 짧은 지연 시간과 내구성을 고려할 때, C. 프로비저닝된 IOPS SSD Amazon EBS 볼륨이 가장 적합합니다. 이를 통해 애플리케이션은 요구되는 고성능과 신뢰성을 확보할 수 있습니다.

621. An online photo-sharing company stores photos in an Amazon S3 bucket in the us-west-1 region. The company needs to store a copy of all new photos in the us-east-1 region. What solution can meet this requirement with minimal operational effort?
① A. Create a second S3 bucket in us-east-1. Use S3 cross-region replication to copy photos from the existing S3 bucket to the second S3 bucket.
② B. Create a CORS (Cross-Origin Resource Sharing) configuration for your existing S3 bucket. Specify us-east-1 in the AllowedOrigin element of the CORS rule.
③ C. Create a second S3 bucket in us-east-1 across multiple Availability Zones. Create an S3 lifecycle rule to store photos in the second S3 bucket.
④ D. Create a second S3 bucket in us-east-1. Configure S3 event notifications for object creation and update events to invoke an AWS Lambda function that copies photos from the existing S3 bucket to the second S3 bucket.

  S3 교차 리전 복제는 원본 버킷에 추가된 새 객체를 다른 리전의 대상 버킷에 자동으로 복사하는 작업을 처리합니다. 파일을 수동으로 복사하거나 Lambda 트리거를 설정할 필요 없이 새 사진을 지속적으로 복제합니다. CORS는 원본 간 액세스만 허용하며 객체를 복사하지 않습니다. 수명 주기 규칙 또는 Lambda 함수를 사용하려면 복사를 처리하기 위한 사용자 지정 코드와 논리가 필요합니다. S3 교차 리전 복제는 운영 오버헤드를 최소화하는 자동화된 복제를 제공합니다.

  온라인 사진 공유 회사의 요구 사항을滿足하는 솔루션은 us-west-1 지역의 기존 S3 버킷에서 us-east-1 지역의 두 번째 S3 버킷으로 새 사진을 복사해야 합니다. 이 요구 사항을 최소한의 운영 노력으로 충족할 수 있는 솔루션은 us-east-1 지역에 두 번째 S3 버킷을 생성하고 S3 교차 리전 복제를 사용하여 기존 S3 버킷의 사진을 두 번째 S3 버킷으로 복사하는 것입니다. 이 방법은 자동으로 새 사진을 복사하여 추가적인 운영 노력이 필요 없습니다.

  정답은 1번입니다. 이 솔루션이 가장 효과적인 이유는 다음과 같습니다:

  S3 교차 리전 복제: 옵션 A는 S3의 내장 기능인 교차 리전 복제를 활용합니다. 이를 통해 us-west-1에 있는 원본 S3 버킷의 새 객체가 생성되거나 업데이트될 때 자동으로 us-east-1의 복제본 버킷에 사본이 생성됩니다. 이 방법은 수동 개입 없이 실시간으로 데이터 일관성을 유지할 수 있습니다.
  다른 옵션들의 한계점:

  옵션 B: CORS는 클라이언트 간의 교차 도메인 요청을 처리하는 데 사용되며, 데이터의 자동 복제와는 무관합니다. 따라서 이 요구 사항을 충족시키지 못합니다.
  옵션 C: 가용 영역 내에서의 복제나 수명 주기 규칙은 데이터의 자동 교차 리전 복제를 지원하지 않습니다. 수명 주기 규칙은 주로 데이터의 보관 및 삭제 정책에 초점을 맞추고 있습니다.
  옵션 D: AWS Lambda와 이벤트 알림을 사용하는 방식은 작동하지만, 수동 구성과 람다 함수의 실행 오버헤드가 필요합니다. 이는 실시간 자동화와 최소한의 운영 노력이라는 요구사항에 비해 비효율적입니다.
  따라서, 최소한의 운영 노력과 자동화된 복제를 위해 옵션 1이 가장 적합합니다.

622. A company is building a new web application for its subscribers. The application consists of a static single page and a persistent database layer. During the first four hours of the morning, the application receives millions of users, but during the rest of the day, it receives only a few thousand users. The company's data architects have requested the ability to rapidly evolve the schema.
Which solution meets these requirements and offers the greatest scalability? (Choose two.)
① A. Deploy Amazon DynamoDB as your database solution. Provision on-demand capacity.
② B. Deploy Amazon Aurora as your database solution. Choose the serverless DB engine mode.
③ C. Deploy Amazon DynamoDB as your database solution. Ensure DynamoDB Auto Scaling is enabled.
④ D. Deploy static content to an Amazon S3 bucket. Provision an Amazon CloudFront distribution using the S3 bucket as the origin.
⑤ E. Deploy a web server for static content across Amazon EC2 instances in an Auto Scaling group. Configure the instances to periodically refresh content from the Amazon Elastic File System (Amazon EFS) volume.

A: 수요가 급격히 변하기 때문에 온디맨드 확장(수백만에서 수천)
D: 단일 페이지 앱에 대한 대부분의 확장성 = S3 버킷을 원본으로 사용하는 Amazon CloudFront 배포

  정답은 4번입니다. 회사는 정적 단일 페이지와 영구 데이터베이스 계층으로 구성된 새로운 웹 애플리케이션을 만들고 있습니다. 애플리케이션의 사용자는 아침에만 수백만 명에 달하고 나머지 시간에는 수천 명에 불과합니다. 따라서, 데이터 설계자는 스키마를 빠르게 발전시킬 수 있는 기능을 요청했습니다. 이러한 요구 사항을 충족하고 가장 뛰어난 확장성을 제공하는 솔루션은 정적 콘텐츠를 Amazon S3 버킷에 배포하고 S3 버킷을 원본으로 사용하여 Amazon CloudFront 배포를 프로비저닝하는 것입니다. Amazon S3는 확장성이 뛰어난 정적 콘텐츠 저장소로, 높은 요청률에도 대응할 수 있습니다. Amazon CloudFront는 캐싱 및 콘텐츠 전송 네트워크 서비스로, 사용자에게 더 빠르고 안전하게 콘텐츠를 제공할 수 있습니다. 이러한 솔루션은 회사에 필요한 확장성과 성능을 제공할 수 있습니다.

  정답은 4번과 1번입니다.

  4번 (Amazon S3 + CloudFront): 정적 단일 페이지 애플리케이션의 경우, Amazon S3는 저렴하고 확장성이 뛰어난 객체 스토리지를 제공하여 콘텐츠를 효율적으로 저장하고 관리할 수 있습니다. CloudFront는 S3 버킷을 원본으로 사용하여 글로벌 콘텐츠 배송 네트워크를 통해 사용자에게 빠르게 콘텐츠를 제공하여 성능을 극대화합니다. 특히 사용자 수 변동이 큰 상황에서 S3와 CloudFront의 자동 확장 기능은 비용 효율적인 솔루션을 제공합니다.

  1번 (DynamoDB + 온디맨드 용량): 영구 데이터베이스 계층의 경우, DynamoDB는 NoSQL 데이터베이스로 스키마 유연성과 빠른 성능이 강점입니다. 온디맨드 용량 프로비저닝은 사용자 트래픽 변동에 따라 자동으로 리소스를 조절하여 아침 시간대의 수백만 명의 사용자 부하에도 안정적으로 대응합니다. 이는 데이터 설계자의 요구사항인 빠른 스키마 발전을 지원하며, 확장성을 최우선으로 고려하는 상황에 적합합니다.

  2번, 3번, 5번은 적합하지 않습니다.

  2번 (Amazon Aurora 서버리스): 서버리스 모드는 비용 효율적이지만, 데이터베이스 워크로드에 대한 예측 가능한 패턴이 없는 경우 비용 예측이 어려울 수 있습니다. 또한, 서버리스 모드는 장기적인 쿼리 실행 등 특정 작업에 제약이 있을 수 있습니다.
  3번 (DynamoDB Auto Scaling): Auto Scaling은 유용하지만, 온디맨드 용량 프로비저닝과 비교했을 때 예측 불가능한 트래픽 변동에 대한 즉각적인 대응력이 다소 떨어질 수 있습니다.
  5번 (EC2 + EFS): 정적 콘텐츠 배포에는 EC2 인스턴스와 EFS가 과도한 리소스 사용을 초래하며, 확장성과 비용 효율성 측면에서 S3와 CloudFront 조합보다 불리합니다.

623. The company uses Amazon API Gateway to manage a REST API accessed by third-party service providers. The company needs to protect the REST API from SQL injection and cross-site scripting attacks. What is the most operationally efficient solution to meet these requirements?
① A. Configure AWS Shield.
② B. Configure AWS WAF.
③ C. Set up API Gateway using an Amazon CloudFront distribution. Configure AWS Shield on CloudFront.
④ D. Set up API Gateway with an Amazon CloudFront distribution. Configure AWS WAF on CloudFront.

  SQL 주입 및 교차 사이트 스크립팅 = WAF이므로 B 또는 D입니다. B와 D는 모두 유효한 옵션이지만 질문은 CloudFront가 실제로 필요함을 나타내지 않으므로 API 게이트웨이와 함께 WAF를 사용하면 됩니다.

  AWS WAF를 구성하면 REST API를 SQL 주입 및 크로스 사이트 스크립팅 공격으로부터 보호할 수 있습니다. AWS WAF는 웹 애플리케이션 방화벽 서비스로,悪의적인 요청을 필터링하고 웹 애플리케이션을 보호하는 기능을 제공합니다. 다른 보안 서비스와 통합하여 사용할 수 있습니다. 따라서 AWS WAF를 구성하는 것이 가장 운영 효율적인 솔루션입니다. 다른 옵션은 부가적인 보안을 제공할 수 있으나, SQL 주입 및 크로스 사이트 스크립팅 공격으로부터 REST API를 보호하는 데에는 AWS WAF가 가장 적합합니다.

  정답은 B입니다. AWS WAF(Web Application Firewall)를 구성하는 것이 가장 효과적입니다.

  SQL 주입과 크로스 사이트 스크립팅(XSS) 공격을 방어하기 위해서는 웹 애플리케이션 수준에서의 세밀한 트래픽 필터링이 필요합니다. AWS WAF는 이러한 특정 유형의 공격을 차단하도록 설계된 규칙을 적용할 수 있는 기능을 제공합니다.

  AWS Shield는 주로 DDoS 공격에 대한 보호를 제공하므로, SQL 주입이나 XSS와 같은 애플리케이션 수준의 위협에는 적합하지 않습니다.
  CloudFront와의 조합 (C와 D 옵션)은 캐싱 및 콘텐츠 배포의 효율성을 높일 수 있지만, 기본적으로 애플리케이션 수준의 보안 위협을 직접적으로 방어하는 기능은 제공하지 않습니다. CloudFront 자체에는 이러한 보안 규칙이 내장되어 있지 않기 때문에, AWS WAF를 CloudFront와 함께 사용하는 D 옵션도 부분적으로 올바르지만, 가장 직접적이고 효과적인 해결책은 AWS WAF를 API Gateway 바로 앞에 직접 적용하는 것입니다.
  따라서, SQL 주입과 XSS 공격으로부터 REST API를 효과적으로 보호하려면 AWS WAF를 직접 구성하는 것이 가장 적합합니다.

624. A company wants to provide users with access to AWS resources. The company has 1,500 users and manages access to on-premises resources through Active Directory user groups on the company network. However, the company doesn't want users to have to maintain different identities to access resources. The solutions architect needs to manage user access to AWS resources while maintaining access to on-premises resources.
What should the solutions architect do to meet these requirements?
① A. Create an IAM user for each user in your company. Attach the appropriate policy to each user.
② B. Use Amazon Cognito with an Active Directory user pool. Create a role with the appropriate policies attached.
③ C. Define cross-account roles with appropriate policies attached. Map the roles to Active Directory groups.
④ D. Configure a federation based on Security Assertion Markup Language (SAML) 2.0. Create roles with appropriate policies attached. Map the roles to Active Directory groups.

  SAML 통합을 통해 Amazon Cognito를 사용하세요. (SAML)은 자격 증명 공급자(이 경우 온프레미스 AD)가 사용자를 인증하고 사용자에 대한 자격 증명 및 보안 정보를 서비스 공급자(이 경우 AWS)에 전달할 수 있도록 허용하는 개방형 페더레이션 표준입니다

  회사에서는 온프레미스 리소스에 대한 액세스를 유지하면서 AWS 리소스에 대한 사용자 액세스를 관리하려고 합니다. 이를 위해서는 사용자가 리소스에 액세스하기 위해 다른 ID를 유지하지 않도록 해야 합니다.

  솔루션 설계자는 SAML 2.0 기반 페더레이션을 구성하여 Active Directory 사용자 그룹을 AWS 리소스와 통합할 수 있습니다. 이를 통해 Active Directory를 통해 액세스 권한을 관리할 수 있습니다.

  즉, 사용자는 온프레미스 리소스에 액세스하는 것과 동일한 자격 증명을 사용하여 AWS 리소스에 액세스할 수 있습니다. 적절한 정책이 연결된 역할을 생성하여 사용자 그룹에 매핑하면 사용자가 온프레미스 리소스와 AWS 리소스에 액세스할 수 있도록 할 수 있습니다.

  따라서 솔루션 설계자는 SAML 2.0 기반 페더레이션을 구성하고 적절한 정책이 연결된 역할을 생성하여 Active Directory 그룹에 매핑해야 합니다.

  정답은 4번입니다. 이 솔루션은 온프레미스 Active Directory와 AWS 간의 원활한 싱글 사인온(SSO) 환경을 구축하는 데 적합합니다. 구체적으로 

  SAML 2.0 기반 페더레이션을 사용하면, 회사의 온프레미스 Active Directory와 AWS 간에 보안 인증 정보를 교환할 수 있습니다. 이를 통해 사용자들은 이미 Active Directory를 통해 인증받은 상태에서 추가적인 로그인 없이 AWS 리소스에 안전하게 접근할 수 있습니다.

  역할 생성 및 그룹 매핑: 각 Active Directory 그룹에 맞는 AWS 역할을 생성하고, 이 역할들에 필요한 액세스 권한을 부여하는 정책을 연결합니다. 그런 다음, Active Directory 그룹과 AWS 역할을 매핑하여 해당 그룹의 모든 사용자가 자동으로 해당 역할의 권한을 갖게 됩니다.

  이 방법은 사용자가 별도의 AWS ID를 관리할 필요 없이 기존의 온프레미스 인증 시스템을 활용할 수 있게 해주므로, 회사의 요구 사항을 완벽하게 충족시킵니다. 다른 옵션들은 부분적인 해결책을 제공하거나 복잡성을 증가시킬 수 있어 적합하지 않습니다.

625. A company hosts its website behind multiple Application Load Balancers. The company has varying distribution rights for content globally. The solution architect must ensure that users receive the correct content without violating distribution rights. What configuration should the solution architect choose to meet these requirements?
① A. Configure Amazon CloudFront with AWS WAF.
② B. Configuring Application Load Balancer with AWS WAF
③ C. Configuring Amazon Route 53 with a Geolocation Policy
④ D. Configuring Amazon Route 53 with a Geoproximity Routing Policy

  지리적 위치 정책으로 Amazon Route 53을 구성합니다. 지리적 위치 정책으로 Amazon Route 53을 구성함으로써 솔루션 아키텍트는 지리적 위치에 따라 사용자를 다양한 Application Load Balancer로 안내할 수 있습니다. 이를 통해 회사는 배포권을 침해하지 않고 다양한 지역의 사용자에게 올바른 콘텐츠를 제공할 수 있습니다. 지리적 위치 라우팅 정책을 사용하면 사용자의 지리적 위치를 기반으로 트래픽을 라우팅하여 사용자가 위치를 기반으로 가장 가깝거나 가장 적절한 엔드포인트로 연결되도록 할 수 있습니다. 이 솔루션은 콘텐츠 배포 권한이 지역에 따라 다르며 이에 따라 시행되어야 하는 시나리오에 적합합니다.

  콘텐츠에 대한 다양한 배포권한을 지키면서 사용자에게 올바른 콘텐츠를 제공하기 위해서는 사용자들이 접근하는 지리적인 위치를 기준으로 콘텐츠를 제공해야 합니다. 이를 위해서는 지리적인 위치에 따라 콘텐츠를 분기해 줄 수 있는 기능이 필요합니다.

  Amazon Route 53의 지리적 위치 정책을 사용하면 사용자의 지리적 위치에 따라 다르게 콘텐츠를 제공할 수 있습니다. 따라서, 지리적으로 다른 위치에서 접근하는 사용자들에게 각기 다른 콘텐츠를 제공하여 배포 권한을 지킬 수 있습니다.

  따라서, 이러한 요구 사항을 충족하기 위해 지리적 위치 정책으로 Amazon Route 53를 구성하는 것이 적절합니다.

626. A company stores its data on-premises. The volume of data is growing beyond the company's available capacity. The company wants to migrate the data from its on-premises location to an Amazon S3 bucket . The company needs a solution that automatically verifies the integrity of the data after transfer. Which solution meets these requirements?
① A. Order an AWS Snowball Edge device. Configure the Snowball Edge device to perform online data transfers to S3 buckets.
② B. Deploy the AWS DataSync agent on-premises. Configure the DataSync agent to perform online data transfers to S3 buckets.
③ C. Create an Amazon S3 File Gateway on-premises and configure the S3 File Gateway to perform online data transfers to an S3 bucket.
④ D. Configure an accelerator for Amazon S3 Transfer Acceleration on-premises. Configure the accelerator to perform online data transfers to S3 buckets.

  전송 중에 AWS DataSync는 항상 데이터의 무결성을 확인하지만 다음 옵션을 사용하여 이 확인이 수행되는 방법과 시기를 지정할 수 있습니다. 전송된 데이터만 확인(권장) - DataSync는 소스에서 전송된 파일 및 메타데이터의 체크섬을 계산합니다. 위치.

  회사의 요구 사항은 데이터 마이그레이션의 효율성과 전송 후 데이터 무결성 검증에 있다. AWS DataSync (보기 2)는 온프레미스 시스템과 클라우드 스토리지 간의 안정적이고 자동화된 데이터 전송을 지원한다. 특히, DataSync는 전송 프로세스 내에서 데이터의 일관성과 무결성을 자동으로 검증하는 기능을 제공한다. 이는 회사가 필요로 하는 핵심 요소이다.

  다른 옵션들을 살펴보면:

  AWS Snowball Edge (보기 1)는 대용량 데이터의 물리적 운송에 효과적이나, 주로 오프라인 환경에서 활용되고 자동 무결성 검증 기능은 제한적일 수 있다.
  S3 파일 게이트웨이 (보기 3)는 주로 가상화된 환경에서 파일 시스템 인터페이스로 S3에 액세스할 수 있게 해주지만, 대규모 마이그레이션 시의 자동 무결성 검증 기능은 포함되지 않는다.
  S3 Transfer Acceleration (보기 4)는 주로 전송 속도 향상에 초점을 맞추고 있어, 데이터 무결성 검증을 위한 내장 기능은 제공하지 않는다.
  따라서, 데이터의 자동 무결성 검증을 포함한 원활한 온라인 마이그레이션을 위해 가장 적합한 솔루션은 AWS DataSync (보기 2)이다.

627. A company is planning to migrate two DNS servers to AWS. The servers host approximately 200 zones and receive an average of 1 million requests per day. The company wants to maximize availability while minimizing the operational overhead associated with managing the two servers. What should the solution architect recommend to meet these requirements?
① A. Create 200 new hosted zones from the Amazon Route 53 console import zone file.
② B. Launch the Import a Single Large Amazon EC2 Instance area tile. Configure Amazon CloudWatch alarms and notifications to notify your company of any downtime.
③ C. Migrate servers to AWS using AWS Server Migration Service (AWS SMS). Configure Amazon CloudWatch alarms and notifications to notify your company of any downtime.
④ D. Launch Amazon EC2 instances in an Auto Scaling group across two Availability Zones. Import the zone file. Set the desired capacity for the Auto Scaling group to 1 and the maximum capacity to 3. Configure a scaling alarm to adjust based on CPU utilization.

  회사에서 두 개의 DNS 서버를 AWS로 마이그레이션하려는 경우를 생각해 보자. 두 서버의 관리와 관련된 운영 오버헤드를 최소화하면서 가용성을 극대화하는 것이 목표다.

  이러한 요구 사항을 충족하기 위해 Amazon Route 53을 사용하는 것이 좋다. Amazon Route 53은 클라우드 기반의 Domain Name System(DNS) 서비스로, 높은 가용성과 확장성을 제공한다.

  Amazon Route 53을 사용하면 200개의 새로운 호스팅 영역을 생성할 수 있다. 이는 회사의 DNS 요구 사항을 충족하는 데 도움이 되며, 운영 오버헤드를 최소화할 수 있다. 또한, Amazon Route 53은 자동으로 트래픽을 여러 영역에 분산시켜 가용성을 극대화할 수 있다.

  따라서, 정답은 1번인 Amazon Route 53 콘솔에서 200개의 새로운 호스팅 영역을 생성하는 것이다. 다른 옵션들은 DNS 서버를 마이그레이션하고 운영 오버헤드를 최소화하는 데 필요한 요구 사항을 충족하지 않는다.

  정답은 1번입니다.

  회사의 주요 목표는 DNS 서버의 운영 오버헤드를 줄이고 가용성을 극대화하는 것입니다.

  1번 옵션은 Amazon Route 53을 활용하는 방법입니다. Route 53은 AWS의 고성능 DNS 서비스로, 자동 복제 및 고가용성을 제공하여 관리 부담을 최소화하고 안정성을 높입니다. 200개의 도메인을 효율적으로 관리하고 높은 요청 처리량을 지원할 수 있습니다. 또한, Route 53 자체적으로 레드언던시 기능을 갖추고 있어 별도의 복잡한 설정 없이도 높은 가용성을 보장합니다.
  다른 옵션들은 문제 상황에 적합하지 않습니다.

  2번은 단일 EC2 인스턴스에 의존하여 가용성을 확보하기 어렵고, 가동 중지 시간 발생 가능성이 높습니다.
  3번은 서버 마이그레이션 솔루션인 AWS SMS를 사용하는 것이지만, DNS 서비스 자체의 고가용성과 관리 효율성을 직접적으로 해결하지 않습니다.
  4번은 EC2 인스턴스 기반으로 DNS 서비스를 구현하려는 접근으로, 운영 오버헤드가 증가하고 Route 53 대비 가용성 및 확장성 측면에서 불리합니다.
  따라서, 최적의 솔루션은 AWS의 전문 DNS 서비스인 Amazon Route 53을 활용하는 1번 옵션입니다.

628. A global company runs applications across multiple AWS accounts in AWS Organizations. The company's applications use multipart uploads to upload data to multiple Amazon S3 buckets in AWS Regions. The company wants to report on incomplete multipart uploads for cost compliance purposes.
What solution can meet these requirements with minimal operational overhead?
① A. Configure AWS Config with a rule that reports the number of incomplete multipart upload objects.
② B. Create a Service Control Policy (SCP) that reports the number of incomplete multipart upload objects.
③ C. Configure the S3 storage lens to report the number of incomplete multipart upload objects.
④ D. Create an S3 multi-region access point to report the number of incomplete multipart upload objects.

  3번. 불완전한 멀티파트 업로드 객체 수를 보고하도록 S3 스토리지 렌즈를 구성합니다.
  S3 스토리지 렌즈는 S3 버킷의 불완전한 멀티파트 업로드를 분석하기 위한 4가지 비용 효율성 지표를 제공합니다. 이러한 지표는 무료이며 모든 S3 Storage Lens 대시보드에 대해 자동으로 구성됩니다. 불완전 멀티파트 업로드 스토리지 바이트 – 불완전 멀티파트 업로드가 포함된 범위 내 총 바이트 % 불완전 MPU 바이트 – 불완전 멀티파트 업로드의 결과인 범위 내 바이트 비율 불완전 멀티파트 업로드 객체 수 – 불완전 멀티파트 업로드인 범위 내 객체 수 불완전한 MPU 객체 % – 범위 내에서 불완전한 멀티파트 업로드인 객체의 비율

629. A company runs a production database on Amazon RDS for MySQL. The company wants to upgrade the database version to comply with security regulations. Because the database contains sensitive data, the company wants a quick solution that allows them to upgrade and test features without data loss.
What solution can meet these requirements with minimal operational overhead?
① A. Create a manual snapshot of RDS. Upgrade to a new version of Amazon RDS for MySQL.
② B. Use native backup and restore. Restore data to the upgraded version of Amazon RDS for MySQL.
③ C. Replicate data to the new, upgraded version of Amazon RDS for MySQL using AWS Database Migration Service (AWS DMS).
④ D. Deploy and test production changes using Amazon RDS blue/green deployments.

  정답은 4. Amazon RDS 블루/그린 배포입니다.

  블루/그린 배포는 최소 가동 중단과 위험 감소를 위해 프로덕션 환경을 복제하는 방법입니다. MySQL 버전 업그레이드 시에는 다음과 같은 이점이 있습니다:

  데이터 손실 방지: 기존 프로덕션 데이터베이스(블루 환경)를 유지한 채, 새로운 환경(그린 환경)에서 업그레이드된 버전으로 데이터베이스를 설정할 수 있습니다.
  빠른 테스트: 업그레이드된 버전에서의 기능 테스트를 그린 환경에서 진행 가능하여 실제 프로덕션에 영향을 주지 않고 검증할 수 있습니다.
  원활한 전환: 모든 테스트가 완료되면 트래픽을 그린 환경으로 스위칭하여 운영 오버헤드를 최소화합니다.
  다른 옵션들과 비교했을 때,

  옵션 1과 2는 스냅샷이나 기본 백업 복원을 사용하므로 업그레이드 과정에서 일시적인 중단이 발생하거나 데이터 일관성 문제가 생길 위험이 있습니다.
  옵션 3 (AWS DMS)는 데이터 복제에 효과적이지만, 데이터베이스 버전 업그레이드 자체의 관리와 테스트 환경 구축에는 추가적인 복잡성이 따르게 됩니다.
  따라서 데이터 손실 없이 안전하고 빠르게 업그레이드와 테스트를 진행할 수 있는 최적의 선택은 Amazon RDS 블루/그린 배포입니다.

630. A solution architect is creating a data processing job that runs once a day and can take up to two hours to complete . If the job is interrupted, it must be restarted from the beginning. How can the solution architect solve this problem in the most cost-effective way?
① A. Create a script that runs locally on an Amazon EC2 Reserved Instance triggered by a cron job.
② B. Create an AWS Lambda function that is triggered by an Amazon EventBridge scheduled event.
③ C. Use Amazon Elastic Container Service (Amazon ECS) Fargate tasks triggered by Amazon EventBridge scheduled events.
④ D. Use Amazon Elastic Container Service (Amazon ECS) tasks running on Amazon EC2 that are triggered by Amazon EventBridge scheduled events.

  솔루션 설계자는 비용 효과적인 방식으로 데이터 처리 작업을 수행해야 한다. 작업이 하루에 한 번 실행되고 최대 2시간이 걸릴 수 있으며 중단되면 처음부터 다시 시작해야 한다.

  Amazon Elastic Container Service(Amazon ECS) Fargate 작업을 사용하면 서버 관리 부담 없이 작업을 실행할 수 있다. 또한 Fargate를 사용하면 작업이 완료된 후에 자동으로 종료되므로 비용을 절약할 수 있다.

  이 경우 Amazon EventBridge 예약 이벤트에 의해 트리거되는 Amazon ECS Fargate 작업을 사용하는 것이 가장 비용 효과적인 방식이다. 이 방법을 사용하면 작업을 실행하는 데 필요한 인프라를 관리할 필요가 없으며, 작업이 완료된 후에 자동으로 종료되므로 비용을 절약할 수 있다.

  따라서 가장 비용 효과적인 방식은 Amazon EventBridge 예약 이벤트에 의해 트리거되는 Amazon ECS Fargate 작업을 사용하는 것이다.

  이 문제는 매일 하루에 한 번 실행되고 최대 2시간 동안 완료될 수 있는 데이터 처리 작업을 만들 때 가장 비용 효과적인 방법을 찾는 것입니다.

  정답은 3번입니다. Amazon EventBridge 예약 이벤트에 의해 트리거되는 Amazon ECS Fargate 작업을 사용합니다. Fargate는 서버 관리가 필요 없고, 작업 완료 후 리소스를 자동으로 Scaling down 시켜 비용을 절감합니다. 작업이 완료되면 자동으로 종료되기 때문에 유휴 리소스를 방지합니다.

  다른 옵션은 다음과 같습니다.

  EC2 예약 인스턴스: 작업이 완료된 후에도 인스턴스가 계속 실행되기 때문에 비용 효율성이 떨어집니다.
  Lambda: 작업이 15분을 넘기 때문에 Lambda의 실행 시간 제한으로 인해 사용할 수 없습니다.
  ECS on EC2: EC2 인스턴스를 관리해야 하기 때문에 운영 부담과 비용이 증가합니다.
  Fargate는 작업 스케줄링과 리소스 관리를 자동화하고 비용을 가장 효율적으로 절감할 수 있습니다.

631. A social media company wants to store a database of user profiles, relationships, and interactions in the AWS Cloud. The company needs an application that monitors changes to the database. The application must analyze relationships between data entities and provide recommendations to users.
What solution can meet these requirements with minimal operational overhead?
① A. Use Amazon Neptune to store information. Use Amazon Kinesis Data Streams to process changes in the database.
② B. Store information using Amazon Neptune. Process database changes using Neptune Streams.
③ C. Store information using Amazon Quantum Ledger Database (Amazon QLDB). Process database changes using Amazon Kinesis Data Streams.
④ D. Store information using Amazon Quantum Ledger Database (Amazon QLDB). Process database changes using Neptune Streams.

  Amazon Neptune 데이터베이스는 뛰어난 확장성과 가용성을 위해 설계된 서버리스 그래프 데이터베이스입니다. Neptune 데이터베이스는 내장된 보안, 지속적인 백업 및 다른 AWS 서비스와의 통합을 제공합니다. 소셜 미디어에 적합합니다.
  Neptune Streams 기능을 사용하면 그래프 데이터에 발생한 모든 변경 사항을 실시간으로 기록하는 변경 로그 항목의 전체 시퀀스를 생성할 수 있습니다.

  이 문제에서는 소셜 미디어 회사의 요구 사항을 충족하기 위해 적합한 솔루션을 찾는 데 초점이 맞춰져 있습니다. 요약하면 회사는 사용자 프로필, 관계 및 상호 작용에 대한 데이터베이스를 저장하고, 데이터베이스의 변경 사항을 모니터링하고 분석하여 사용자에게 추천 사항을 제공해야 합니다.

  권장되는 솔루션은 Amazon Neptune을 사용하여 정보를 저장하고 Neptune Streams를 사용하여 데이터베이스의 변경 사항을 처리하는 것입니다. Amazon Neptune은 그래프 데이터베이스로 데이터 엔터티 간의 복잡한 관계를 저장하고 查询하는 데 적합합니다. Neptune Streams를 사용하면 데이터베이스의 사항을 스트리밍할 수 있어 애플리케이션이 실시간으로 분석과 추천을 제공할 수 있습니다. 이 솔루션은 최소한의 운영 오버헤드로 이러한 요구 사항을 충족할 수 있습니다.

632. A company is developing a new application that will store large amounts of data. The data will be analyzed hourly and modified by multiple Amazon EC2 Linux instances deployed across multiple Availability Zones. The amount of storage required will continue to increase over the next six months. What storage solution should the solution architect recommend to meet these requirements?
① A. Store your data in Amazon S3 Glacier. Update the S3 Glacier vault policy to allow access to your application instances.
② B. Store data on Amazon Elastic Block Store (Amazon EBS) volumes. Mount the EBS volumes on your application instances.
③ C. Store data in an Amazon Elastic File System (Amazon EFS) file system. Mount the file system on your application instances.
④ D. Store data on Amazon Elastic Block Store (Amazon EBS) provisioned IOPS volumes shared between application instances.

  Amazon EFS에서는 여러 Amazon EC2 인스턴스가 동일한 파일 시스템을 동시에 탑재할 수 있으므로 여러 인스턴스가 동시에 데이터에 쉽게 액세스하고 수정할 수 있습니다.

  이 문제는 AWS에서 대량의 데이터를 저장하고 처리하는 솔루션을 찾는 것입니다. 데이터는 여러 가용 영역에 배포된 여러 Amazon EC2 인스턴스에 의해 수정되며, 저장 공간의 양은 계속 증가할 예정입니다.

  솔루션 설계자는 여러 인스턴스에서 데이터에 액세스할 수 있도록 지원하는 스토리지 솔루션을 선택해야 합니다. Amazon S3 Glacier는 물리적인 보관을 위한 것으로, 데이터에 즉시 액세스할 수 없습니다. Amazon Elastic Block Store(Amazon EBS)는 인스턴스에_attach된 볼륨을 지원하지만 여러 인스턴스에서 동시에 액세스할 수는 없습니다.

  Amazon Elastic File System(Amazon EFS)은 여러 인스턴스에서 공유할 수 있는 파일 시스템을 제공하므로 데이터를 저장하고 처리하는 데 적합합니다. 또한 저장 공간의 양이 증가할 수록 자동으로 확장되므로 향후 늘어날 스토리지 양에도 대응할 수 있습니다.

  따라서 필요한 스토리지 솔루션은 Amazon Elastic File System(Amazon EFS) 파일 시스템에 데이터를 저장하고 애플리케이션 인스턴스에 파일 시스템을 마운트하는 것입니다.

633. A company manages an application that stores data on an Amazon RDS for PostgreSQL multi-AZ DB instance. Performance issues are occurring due to increased traffic. The company determines that database queries are the primary cause of the performance degradation. What should a solutions architect do to improve application performance?
① A. Serve read traffic from multi-AZ standby replicas.
② B. Configure the DB instance to use Transfer Acceleration.
③ C. Create a read-only replica from the source DB instance. Serve read traffic from the read replica.
④ D. Use Amazon Kinesis Data Firehose between your application and Amazon RDS to increase the concurrency of database requests.

  읽기 트래픽에 대한 읽기 복제본 분할은 전체 로드를 분산하고 성능을 향상시킵니다.
  A: 대기 복제본은 트래픽을 처리할 수 없습니다. (틀렸다면 정정해 주십시오.)
  B: Transfer Accelerator는 S3 트래픽 속도를 높이는 것입니다.
  D: Kiensis는 동시성을 높이지만 DB 성능 문제는 해결하지 않습니다.

  정답은 3번입니다. 성능 저하의 주요 원인이 쿼리 처리에 있다고 판단되는 상황에서, 읽기 전용 복제본을 생성하는 것이 가장 효과적입니다. 이 방법은 다음과 같은 이점을 제공합니다:

  읽기 부하 분산: 원본 DB 인스턴스는 주로 쓰기 작업에 집중하게 되며, 읽기 트래픽은 복제본으로 분산됩니다. 이로 인해 원본 데이터베이스의 부하가 줄어들어 쓰기 및 복합적인 쿼리 성능이 향상됩니다.
  고가용성 유지: 복제본은 읽기 작업을 처리함으로써 추가적인 가용성을 제공할 수 있으며, 필요 시 주 데이터베이스를 대체할 수 있는 옵션으로도 활용 가능합니다.
  다른 옵션들에 대한 간단한 설명:

  1번: 다중 AZ 대기 복제본은 주로 고가용성과 재해 복구를 위해 사용되며, 읽기 부하 분산에는 적합하지 않습니다.
  2번: Transfer Acceleration은 주로 S3와 같은 스토리지 서비스에서 데이터 전송 속도를 높이는 데 효과적이지만, 데이터베이스 쿼리 성능 향상에는 직접적인 영향을 미치지 않습니다.
  4번: Amazon Kinesis Data Firehose는 주로 스트리밍 데이터를 처리하고 저장하는데 사용되며, 데이터베이스 요청의 동시성을 높이는 데는 적합하지 않습니다.
  따라서, 성능 문제 해결과 읽기 부하 분산을 위해 3번이 최적의 선택입니다.

635. A company uses Amazon FSx for NetApp ONTAP in its primary AWS region for CIFS and NFS file sharing. Applications running on Amazon EC2 instances access the file shares. The company needs a storage disaster recovery (DR) solution in a secondary region. Data replicated to the secondary region must be accessed using the same protocols as the primary region.
What solution meets these requirements with minimal operational overhead?
① A. Create an AWS Lambda function to copy data to an Amazon S3 bucket. Replicate the S3 bucket to a secondary region.
② B. Create a backup of your FSx for ONTAP volume using AWS Backup. Copy the volume to a secondary region. Create a new FSx for ONTAP instance from the backup.
③ C. Create an FSx for ONTAP instance in the secondary region. Replicate data from the primary region to the secondary region using NetApp SnapMirror.
④ D. Create an Amazon Elastic File System (Amazon EFS) volume. Migrate your current data to the volume. Replicate the volume to a secondary region.

  NetApp SnapMirror를 사용하여 FSx for ONTAP 파일 시스템과 두 번째 파일 시스템 간의 주기적인 복제를 예약할 수 있습니다. 이 기능은 지역 내 배포와 지역 간 배포 모두에 사용할 수 있습니다.

  필요한 요구 사항을 충족하는 솔루션은 NetApp SnapMirror를 사용하여 기본 지역에서 보조 지역으로 데이터를 복제하는 것입니다. 이렇게 하면 기본 지역과 동일한 프로토콜을 사용하여 복제된 데이터에 액세스할 수 있습니다. 운영 오버헤드를 최소화하면서 스토리지 재해 복구 솔루션을 제공하기 때문에 적절한 선택입니다. 다른 옵션은 동일한 프로토콜을 사용하여 데이터에 액세스할 수 없거나 너무 복잡하여 운영 오버헤드가 증가합니다. 따라서 NetApp SnapMirror를 사용하는 것이 가장 적합한 솔루션입니다.

  정답은 C입니다.

  NetApp SnapMirror 기술을 활용하면 기본 리전의 FSx for ONTAP에서 보조 리전의 FSx for ONTAP로 실시간 또는 증분 데이터 복제가 가능합니다. 이 방법은 다음과 같은 이점을 제공합니다:

  동일 프로토콜 유지: CIFS와 NFS 프로토콜을 그대로 유지하며, 애플리케이션 측의 변경 없이 보조 리전에서도 동일한 방식으로 파일 공유에 접근 가능.
  최소 운영 오버헤드: 자동화된 복제 프로세스 덕분에 수동 관리가 최소화되어 운영 부담이 적습니다.
  재해 복구 효과: 빠르고 안정적인 데이터 복제로 재해 복구 요구 사항을 충족시킵니다.
  다른 옵션들은 다음과 같은 한계가 있습니다:

  A, D 옵션: S3나 EFS는 파일 시스템의 네이티브 프로토콜(CIFS/NFS)을 직접 지원하지 않으므로, 애플리케이션 변경이 필요합니다.
  B 옵션: AWS Backup을 사용한 백업 및 복원 과정은 시간이 오래 걸리고, 재해 복구 시 수동 개입이 필요하며, 실시간 복제보다는 비효율적입니다.

636. The development team is building an event-driven application using AWS Lambda functions. An event is generated when a file is added to an Amazon S3 bucket. The development team currently has an Amazon Simple Notification Service (Amazon SNS) configured as an event target for Amazon S3.
What should a solutions architect do to handle events from Amazon S3 in a scalable manner?
① A. Create an SNS subscription in Amazon Elastic Container Service (Amazon ECS) to process the event before the event is triggered in Lambda.
② B. Create an SNS subscription to process events in Amazon Elastic Kubernetes Service (Amazon EKS) before the events are executed in Lambda.
③ C. Create an SNS subscription that sends events to Amazon Simple Queue Service (Amazon SQS). Configure the SQS queue to trigger a Lambda function.
④ D. Create an SNS subscription that sends events to AWS Server Migration Service (AWS SMS). Configure a Lambda function to poll for SMS events.

  Amazon SQS는 이벤트 중심의 확장 가능한 메시지 처리를 위해 설계되었습니다. 대량의 메시지를 처리할 수 있으며 들어오는 워크로드에 따라 자동으로 확장됩니다. 이를 통해 직접 Lambda 호출에 비해 더 나은 로드 분산 및 확장이 가능합니다.

  정답은 3번입니다. Amazon S3의 이벤트를 확장 가능한 방식으로 처리하려면, Amazon Simple Queue Service(Amazon SQS)를 사용하여 이벤트를 버퍼링하고, Lambda 함수를 트리거하도록 SQS 대기열을 구성하는 것이 좋습니다.

  이러한 접근 방식은 다음과 같은 이유로 확장성이 뛰어나습니다:

  Amazon S3의 이벤트가 많아져도 SQS 대기열이 버퍼링 역할을 하므로 Lambda 함수가 과부하되지 않습니다.
  Lambda 함수는 필요할 때만 실행되므로 비용을 절약할 수 있습니다.
  SQS 대기열은 높은 처리량을 처리할 수 있어 확장성이 뛰어납니다.
  반면에 다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  1번과 2번 옵션은 Amazon ECS와 Amazon EKS를 사용하여 이벤트를 처리하지만, 이는 불필요한 복잡성과 추가 비용을 초래할 수 있습니다.
  4번 옵션은 AWS Server Migration Service(AWS SMS)를 사용하여 이벤트를 처리하지만, 이는 이벤트 처리와 관련이 없는 서비스입니다.

637. A solutions architect is designing a new service based on Amazon API Gateway. The request pattern to the service is unpredictable, and can suddenly change from 0 requests per second to over 500 requests per second. The total size of the data to be maintained in the backend database is currently less than 1 GB, and future growth is unforeseen. The data can be queried using simple key-value requests.
What combination of AWS services would meet these requirements? (Choose two.)
① A. AWS Fargate
② B. AWS Lambda
③ C. Amazon DynamoDB
④ D. Amazon EC2 Auto Scaling
⑤ E. MySQL-compatible Amazon Aurora

  확장 가능하고 예측할 수 없는 요청 패턴
  = AWS Lambda 확장 가능, 키-값 데이터 = Amazon DynamoDB

  여기서 언급된 서비스는 예측할 수 없는 요청 패턴과 데이터 크기 증가에 대처하는 것에 중점을 두고 있습니다. 이런 요구사항에 잘 맞는 서비스가 필요합니다.

  Amazon API Gateway는 요청을 처리하고 라우팅하는 데 사용되며 이는 연결된 서비스와 함께 사용되어야 합니다. 언급된 서비스 중에서 이에 가장 적합한 것은 아마도 C. 아마존 DynamoDB입니다.

  아마존 DynamoDB는 완전 관리형 NoSQL 키-값 및 문서 데이터베이스로, 높은 성능과 자동 크기 조정이 가능하므로 예측할 수 없는 요청 패턴과 데이터 크기 증가에 잘 대처할 수 있습니다. 이 서비스는 확장성과 탄력성이 뛰어나므로 작은 규모에서부터 큰 규모까지의 데이터를 처리하는 데 적합합니다.

  간단한 키-값 요청을 사용하여 데이터를 쿼리할 수 있는 기능도 제공하므로 문제에 언급된 요구사항에 부합합니다. 따라서, 이 경우 아마존 DynamoDB가 적합한 선택입니다.

  주어진 요구 사항을 고려할 때, 주요 고려사항은 예측 불가능한 트래픽 변동과 유연한 데이터 관리입니다.

  C. 아마존 DynamoDB: 키-값 저장소로서 높은 확장성과 자동 스케일링 기능을 제공하여 초당 0에서 500개 이상의 요청을 처리할 수 있습니다. 데이터베이스 크기가 현재 1GB 미만이고 향후 증가를 예측할 수 없음에도 불구하고, DynamoDB는 유연하게 확장 가능하며 성능 최적화에 적합합니다. 또한 간단한 키-값 쿼리를 지원합니다.
  정답 중 두 가지를 선택해야 하는 상황에서, DynamoDB가 필수적이지만 다른 적합한 옵션으로는:

  B. AWS Lambda: 이벤트 기반 컴퓨팅 서비스로, 트래픽 변동에 따라 자동으로 실행되는 함수를 통해 요청을 처리할 수 있습니다. API Gateway와 통합하면, 요청이 들어올 때마다 Lambda 함수가 동적으로 실행되어 백엔드 로직을 처리하고 DynamoDB와 통신할 수 있습니다. 이 조합은 트래픽의 불규칙성을 효과적으로 관리할 수 있습니다.
  따라서, 가장 적합한 답안은 C. 아마존 DynamoDB와 B. AWS Lambda입니다. 이들 서비스의 결합은 가변적인 트래픽과 유연한 데이터 관리 요구 사항을 잘 충족시킵니다. 하지만 문제의 정답이 단일로 지정되어 있다면, 핵심 데이터 관리 솔루션인 C. 아마존 DynamoDB가 핵심적입니다.

638. A company collects research data and shares it with employees around the world. The company wants to collect and store the data in an Amazon S3 bucket and process it in the AWS Cloud. The company shares this data with its employees. The company needs a secure solution for the AWS Cloud that minimizes operational overhead. Which solution meets these requirements?
① A. Use an AWS Lambda function to generate a pre-signed URL for S3. Instruct your employees to use the URL.
② B. Create an IAM user for each employee. Create an IAM policy that grants S3 access for each employee. Instruct employees to use the AWS Management Console.
③ C. Create an S3 file gateway. Create a share for uploads and a share for downloads. Allow employees to mount the shares on their local computers to use the S3 file gateway.
④ D. Configure the AWS Transfer Family SFTP endpoint. Select the Custom Identity Provider option. Manage user credentials using AWS Secrets Manager. Instruct your employees to use Transfer Family.

  AWS Transfer Family(옵션 D) AWS Transfer Family SFTP 엔드포인트를 구성하면 직원이 S3 버킷에 데이터에 액세스하고 데이터를 전송할 수 있는 안전하고 편리한 방법을 제공할 수 있습니다. 사용자 지정 자격 증명 공급자 옵션을 사용하면 기존 자격 증명 시스템과 통합할 수 있으며, AWS Secrets Manager를 사용하여 사용자 자격 증명을 안전하게 관리할 수 있습니다.
  A는 AWS Lambda 함수를 사용하여 S3의 미리 서명된 URL을 생성할 것을 제안합니다. 이것이 작동할 수는 있지만 수동으로 URL을 생성하고 공유해야 하기 때문에 확장성이 낮거나 사용자 친화적이지 않을 수 있습니다.
  B는 S3 액세스에 대한 IAM 정책을 사용하여 각 직원에 대해 IAM 사용자를 생성할 것을 제안합니다. 각 직원의 IAM 사용자를 관리하는 것이 번거롭고 확장성이 떨어질 수 있으므로 운영 오버헤드가 더 많이 발생합니다.
  C는 S3 파일 게이트웨이 사용을 제안합니다. 이것이 작동할 수는 있지만 추가 구성 요소가 필요하며 SFTP 액세스를 위해 AWS Transfer Family를 사용하는 것만큼 간단하거나 효율적이지 않을 수 있습니다.

  정답은 D번입니다.

  회사의 요구 사항은 데이터 공유의 편의성과 동시에 운영 오버헤드를 최소화하는 보안 솔루션을 찾는 것입니다.

  A번은 미리 서명된 URL을 사용하여 일시적인 접근 권한을 부여하지만, 관리와 보안 측면에서 수동적인 개입이 필요하며 대규모로 관리하기 어려울 수 있습니다.
  B번은 IAM 사용자와 정책을 통해 세밀한 액세스 제어가 가능하지만, 각 직원에게 개별 계정을 생성하고 관리해야 하므로 운영 오버헤드가 증가할 수 있습니다.
  C번의 S3 파일 게이트웨이는 직원들이 로컬 환경에서 S3를 마치 파일 시스템처럼 사용할 수 있게 하지만, 복잡한 설정과 유지 관리가 필요하며 모든 직원이 이를 효과적으로 사용할 수 있는지 보장하기 어렵습니다.
  D번의 AWS Transfer Family는 SFTP 엔드포인트를 통해 중앙 집중식으로 데이터 전송을 관리할 수 있어 보안과 운영 효율성을 동시에 높일 수 있습니다. 사용자 정의 ID 공급자와 AWS Secrets Manager를 활용하면 인증과 자격 증명 관리가 자동화되어 운영 오버헤드를 최소화하고, 보안성을 강화할 수 있습니다. 따라서, 운영 효율성과 보안을 모두 충족시키는 최적의 솔루션은 D번입니다.

639. A company is building a new furniture inventory application. The company has deployed the application on a fleet of Amazon EC2 instances across multiple Availability Zones. The EC2 instances run behind an Application Load Balancer (ALB) in a VPC.
The solution architect has determined that incoming traffic appears to favor one EC2 instance, causing latency for some requests
. What should the solution architect do to resolve this issue?
① A. Disable session affinity (sticky sessions) in ALB.
② B. Replace ALB with Network Load Balancer
③ C. Increase the number of EC2 instances in each Availability Zone.
④ D. Adjust the health check frequency for ALB target groups

  고정 세션을 사용하면 동일한 클라이언트가 로드 밸런서 뒤의 동일한 인스턴스로 항상 리디렉션하는 데 도움이 되므로 고정을 활성화하면 백엔드 EC2 인스턴스에 대한 로드 불균형이 발생할 수 있습니다.

  애플리케이션이 여러 가용 영역에 걸쳐 EC2 인스턴스에 배포되었고 ALB 뒤에서 실행되는 경우, 트래픽이 하나의 EC2 인스턴스를 선호하여 지연 시간이 발생하는 문제를 해결하려면 세션 선호도(고정 세션)를 비활성화하는 것이 좋습니다.

  ALB의 세션 선호도 기능은 동일한 클라이언트의 요청을 항상 같은 EC2 인스턴스로 전달하게 합니다. 이 기능이 활성화된 경우, 일부 EC2 인스턴스의 부하가 높아질 수 있고 지연 시간을 발생시킬 수 있습니다. 따라서 이 기능을 비활성화하면 ALB가 들어오는 트래픽을 여러 EC2 인스턴스에 분산시켜 부하를 더 равномер하게 분배할 수 있습니다.

  따라서 이 문제를 해결하려면 ALB에서 세션 선호도(고정 세션)를 비활성화하는 것이 좋습니다. 다른 옵션들은 이 문제를 직접 해결하지 않습니다. Network Load Balancer로 교체하는 것은 다른 유형의 로드 밸랜서를 사용하지만 세션 선호도 문제를 해결하지는 않습니다. EC2 인스턴스 수를 늘리는 것은 처리 능력을 증가시키지만 지연 시간 문제를 직접 해결하지는 않습니다. 또한 ALB 대상 그룹에 대한 상태 확인 빈도 조정은 대상 그룹의 상태를 더 자주 확인하지만 세션 선호도 문제를 해결하지는 않습니다.

  수신한 트래픽이 특정 EC2 인스턴스에만 집중되는 현상은 세션 선호도 또는 고정 세션 설정 때문일 가능성이 큽니다. 이 설정이 활성화되어 있으면 클라이언트의 요청이 동일한 인스턴스로 계속 라우팅되어 부하 분산이 제대로 이루어지지 않으며, 그 결과 일부 인스턴스 과부하로 요청 지연이 발생합니다.

  따라서 문제 해결을 위해 가장 적절한 방법은 ALB에서 세션 선호도를 비활성화하는 것입니다. 이렇게 하면 클라이언트 요청이 다양한 EC2 인스턴스로 균형 있게 분산되어 부하 분산이 개선되고, 결과적으로 요청 지연 시간이 줄어들게 됩니다.

  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  B 옵션: Network Load Balancer는 프로토콜 특성상 성능 향상이 있을 수 있지만, 현재 문제의 핵심인 트래픽 분산 문제 해결에는 직접적인 영향을 미치지 않습니다.
  C 옵션: 인스턴스 수를 늘리는 것은 부하 분산을 도울 수 있지만, 이미 있는 인스턴스들 간의 부하 균형이 이루어지지 않는 한 근본적인 해결책은 아닙니다.
  D 옵션: 상태 확인 빈도 조정은 주로 서비스의 가용성 모니터링과 관련된 설정으로, 트래픽과 부하 분산 문제 해결에는 직접적인 연관성이 적습니다.
  결론적으로, 정답은 1번입니다.

884. A solutions architect is designing a three-tier web application. The architecture consists of an internet-facing Application Load Balancer (ALB) and a web tier hosted on Amazon EC2 instances in a private subnet. The application tier, containing the business logic, runs on EC2 instances in the private subnet. The database tier consists of Microsoft SQL Server running on EC2 instances in the private subnet. Security is the company's top priority.
What combination of security group configurations should the solutions architect use? (Choose three.)
① A. Configure the security group for the web tier to allow inbound HTTPS traffic to the security group for the ALB.
② B. Configure the security group for the web tier to allow outbound HTTPS traffic to 0.0.0.0/0.
③ C. Configure the security group for the database tier to allow inbound Microsoft SQL Server traffic from the security group for the application tier.
④ D. Configure the security group on the database tier to allow outbound HTTPS traffic to the security group on the web tier and Microsoft SQL Server traffic.
⑤ E. Configure the security group for the application tier to allow inbound HTTPS traffic from the security group for the web tier.
⑥ F. Configure the application tier security group to allow outbound HTTPS traffic and Microsoft SQL Server traffic to the web tier security group.

  웹 계층과 애플리케이션 계층, 데이터베이스 계층은 각각 별도의 보안 그룹을 가지고 있기 때문에 이들 사이의 통신을 허용하기 위해 적절한 보안 그룹을 구성해야합니다.

  웹 계층의 보안 그룹은 ALB에서 오는 HTTPS 요청을 허용하도록 구성해야 합니다. 애플리케이션 계층의 보안 그룹은 웹 계층에서 오는 HTTPS 요청을 허용하도록 구성해야 합니다. 이 경우 5번이 적합합니다.

  또한 데이터베이스 계층은 애플리케이션 계층에서 오는 Microsoft SQL Server 트래픽을 허용하도록 구성해야 합니다.

  이러한 설정으로 인해 웹 계층은 ALB의 HTTPS 요청을 받을 수 있고, 애플리케이션 계층은 웹 계층의 HTTPS 요청을 받을 수 있으며, 데이터베이스 계층은 애플리케이션 계층의 Microsoft SQL Server 트래픽을 받을 수 있습니다.

  따라서_company의 최우선 과제인 보안도 유지할 수 있습니다.

  정답은 1번, 5번, 그리고 보안 관점에서 보다 안전한 구성을 위해 간접적으로 언급되지 않았지만 합리적인 선택으로 2번을 포함할 수 있으나, 주어진 정답에 따라 주요 세 가지는 다음과 같습니다:

  A (ALB에 대한 보안 그룹의 인바운드 HTTPS 트래픽 허용): 웹 계층의 ALB는 외부에서의 HTTPS 트래픽을 처리해야 하므로, 이 설정은 외부 사용자의 안전한 접근을 보장합니다. 웹 서버로의 안전한 트래픽 유입을 위해 필수적입니다.

  E (애플리케이션 계층의 보안 그룹에 웹 계층 HTTPS 트래픽 허용): 애플리케이션 계층은 웹 계층으로부터 적절히 통신해야 합니다. 따라서 웹 계층에서 애플리케이션 계층으로의 HTTPS 또는 필요한 프로토콜 트래픽을 허용하는 설정이 필요합니다. 이는 비즈니스 로직의 정상적인 작동을 위해 중요합니다.

  비록 주어진 정답에 명시되지 않았지만, 보안과 실제 운영의 관점에서 고려할 만한 추가적인 선택으로 2번 (웹 계층의 아웃바운드 HTTPS 트래픽 허용)을 언급할 수 있습니다:

  B (비슷한 맥락의 아웃바운드 허용) 대신, 2번을 선택적으로 고려할 수 있는 이유는 웹 서버가 외부와의 HTTPS 통신이 필요한 경우가 많기 때문입니다. 하지만 이는 과도한 허용으로 인해 보안 위험을 증가시킬 수 있으므로, 필요한 특정 아웃바운드 트래픽만을 허용하도록 세밀하게 제한하는 것이 이상적입니다.
  주어진 정답에 따라 가장 적절한 세 가지는 1번, 5번이며, 보안과 실제 운영의 균형을 위해 2번의 개념적 이해는 중요하지만, 주어진 정답에서는 명시되지 않았습니다.

885. A company has released a new version of its production application. The company's workload uses Amazon EC2, AWS Lambda, AWS Fargate, and Amazon SageMaker.
Now that usage has stabilized, the company wants to optimize workload costs. The company wants to guarantee the most services with the fewest savings plans.
What combination of savings plans would meet these requirements? (Choose two.)
① A. Purchase an EC2 instance Savings Plan for Amazon EC2 and SageMaker.
② B. Purchase a Compute Savings Plan for Amazon EC2, Lambda, and SageMaker.
③ C. Purchase a SageMaker Savings Plan.
④ D. Purchase Compute Savings Plans for Lambda, Fargate, and Amazon EC2.
⑤ E. Purchase an EC2 Instance Savings Plan for Amazon EC2 and Fargate.

  이 회사의 워크로드는 Amazon EC2, AWS Lambda, AWS Fargate 및 Amazon SageMaker를 사용합니다. 여기서 EC2, Lambda, Fargate는 컴퓨팅 서비스에 해당합니다.

  이러한 요구 사항을 충족하는 저축 계획 조합은 Lambda, Fargate 및 Amazon EC2에 대한 Compute Savings Plan을 구매하는 것입니다.

  Compute Savings Plan은 EC2 인스턴스, Lambda 함수 및 Fargate와 같은 컴퓨팅 서비스에서 사용량이 안정된 경우에 비용을 최적화하는 데 도움을 주는 저축 계획입니다.

  따라서 이 조건에 부합하는 가장 적합한 저축 계획은 Lambda, Fargate 및 Amazon EC2에 대한 Compute Savings Plan입니다.

  정답은 4번입니다.

  회사는 EC2, Lambda, Fargate, SageMaker 네 가지 서비스를 사용하며, 비용 최적화를 목표로 합니다. Savings Plan은 사용량이 예측 가능할 때 비용 절감 효과를 극대화할 수 있습니다.

  Compute Savings Plan (4번)은 EC2, Lambda, Fargate와 같이 컴퓨팅 기반 서비스에 적용 가능하며, 이 세 서비스를 모두 포함하고 있어 넓은 범위의 워크로드를 보장합니다.
  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다.

  1번, 5번 (EC2 인스턴스 Savings Plan): SageMaker는 EC2 인스턴스 기반으로도 실행될 수 있지만, Savings Plan은 특정 인스턴스 유형에만 적용됩니다. 모든 SageMaker 워크로드를 커버하기 어렵습니다.
  2번: EC2, Lambda에 최적화되었지만 Fargate는 포함되지 않습니다.
  3번: SageMaker만 커버하며, 다른 중요한 서비스 (EC2, Lambda, Fargate)는 제외됩니다.

886. The company uses a Microsoft SQL Server database. The company's applications connect to the database. The company wants to migrate to an Amazon Aurora PostgreSQL database with minimal changes to the application code. What combination of steps would meet these requirements? (Choose two.)
① A. Rewrite the SQL queries in your application using AWS Schema Conversion Tool (AWS SCT).
② B. Enable Babelfish in Aurora PostgreSQL to execute SQL queries in your applications.
③ C. Migrate database schema and data using AWS Schema Conversion Tool (AWS SCT) and AWS Database Migration Service (AWS DMS).
④ D. Connect your application to Aurora PostgreSQL using Amazon RDS Proxy.
⑤ E. Rewrite SQL queries in your application using AWS Database Migration Service (AWS DMS).

  정답은 3번과 B번입니다.

  3번 (C): AWS SCT는 SQL Server 스키마를 Aurora PostgreSQL로 변환하는데 효과적입니다. AWS DMS는 이러한 변환된 스키마와 함께 데이터를 안전하게 마이그레이션하는 데 사용됩니다. 애플리케이션 코드 변경을 최소화하려는 요구 사항에 정확히 부합합니다.

  B번: Babelfish는 Aurora PostgreSQL에 통합된 기능으로, T-SQL (SQL Server의 쿼리 언어)을 지원하여 애플리케이션이 SQL Server 쿼리를 그대로 사용하면서도 PostgreSQL에서 실행될 수 있게 합니다. 이는 코드 수정을 최소화하는 데 매우 유용합니다.

  다른 옵션들은 왜 적합하지 않은가요?

  1번 (A): 애플리케이션 내 SQL 쿼리를 직접 다시 작성하는 것은 요구 사항인 코드 변경 최소화와 충돌합니다.
  4번 (D): RDS Proxy는 연결 관리에 도움이 되지만, 마이그레이션 자체를 해결하지 않습니다.
  5번 (E): AWS DMS는 데이터 마이그레이션에 탁월하지만, SQL 쿼리 재작성을 위한 도구가 아닙니다.

887. A company plans to rehost its applications on Amazon EC2 instances that use Amazon Elastic Block Store (Amazon EBS) as attached storage.
The solution architect must design the solution so that all newly created Amazon EBS volumes are encrypted by default. The solution must also prevent the creation of unencrypted EBS volumes.
Which solution meets these requirements?
① A. Configure your EC2 account properties to always encrypt new EBS volumes.
② B. Use AWS Config to configure an encrypted volume identifier. Apply the default AWS Key Management Service (AWS KMS) key.
③ C. Configure AWS Systems Manager to create encrypted copies of EBS volumes. Reconfigure EC2 instances to use encrypted volumes.
④ D. Create a customer-managed key in AWS Key Management Service (AWS KMS). Configure AWS Migration Hub to use the key when migrating your company's workloads.

  Amazon Elastic Block Store(Amazon EBS) 볼륨의 암호화를 기본으로 설정하려면 EC2 계정 속성을 구성하는 것이 가장 간단한 방법입니다.

  이렇게 설정하면 모든 새로 생성되는 EBS 볼륨이 자동으로 암호화되므로, 암호화되지 않은 볼륨이 생성되는 것을 방지할 수 있습니다.

  이 방법은 AWS에서 제공하는 기본 기능이며, 추가적인 리소스나 서비스를 사용할 필요가 없습니다. 따라서, 이 요구 사항을 충족하는 가장 эффектив한 솔루션은 항상 새 EBS 볼륨을 암호화하도록 EC2 계정 속성을 구성하는 것입니다.

  정답은 1번입니다.

  회사의 요구 사항은 모든 새 Amazon EBS 볼륨이 기본적으로 암호화되어 생성되며, 암호화되지 않은 볼륨의 생성을 방지하는 것입니다.

  1번 옵션은 EC2 계정 설정을 통해 모든 EBS 볼륨이 자동으로 암호화되도록 할 수 있습니다. AWS에서는 특정 KMS 키와 연계된 옵션을 통해 EBS 볼륨의 기본 암호화 설정을 계정 수준에서 관리할 수 있습니다. 이렇게 설정하면 새로운 볼륨이 생성될 때 자동으로 암호화되어 생성되므로, 요구 사항을 완벽하게 충족합니다.

  2번 옵션의 AWS Config는 주로 규정 준수와 리소스 상태 모니터링에 초점을 맞추며, 즉시 생성 시점의 암호화를 강제하는 기능은 제공하지 않습니다.

  3번 옵션은 이미 존재하는 EBS 볼륨을 암호화된 복사본으로 만드는 방식으로, 새 볼륨 생성 시점의 암호화 요구를 직접 해결하지 못합니다.

  4번 옵션은 고객 관리형 키를 생성하고 마이그레이션 과정에서 사용하는 것에 초점을 맞추고 있어, 신규 볼륨 생성 시의 기본 암호화 설정을 직접적으로 다루지 않습니다.

  따라서, 모든 새 EBS 볼륨이 기본적으로 암호화되도록 하는 가장 효과적이고 직접적인 방법은 1번 옵션인 계정 설정을 통해 암호화를 강제하는 것입니다.

888. An e-commerce company wants to collect user clickstream data from its website for real-time analytics. The website experiences fluctuating traffic patterns throughout the day. The company needs a scalable solution that can adapt to varying levels of traffic. Which solution meets these requirements?
① A. Capture clickstream data using a data stream from Amazon Kinesis Data Streams in on-demand mode. Process the data in real time using AWS Lambda.
② B. Capture clickstream data using Amazon Kinesis Data Firehose. Process the data in real time using AWS Glue.
③ C. Capture clickstream data using Amazon Kinesis Video Streams. Process the data in real time using AWS Glue.
④ D. Capture clickstream data using Amazon Managed Service for Apache Flink (formerly Amazon Kinesis Data Analytics). Process the data in real time using AWS Lambda.

  전자상거래 회사의 요구 사항을 충족하는 솔루션은 1번입니다.

  온디맨드 모드에서 Amazon Kinesis Data Streams의 데이터 스트림을 사용하여 클릭스트림 데이터를 캡처하는 이유는 다음과 같습니다.

  데이터 스트림: Amazon Kinesis Data Streams는 대규모의 데이터 스트림을 수집, 처리 및 분석할 수 있는 완전 관리형 서비스입니다.
  실시간 처리: AWS Lambda를 사용하여 실시간으로 데이터를 처리할 수 있습니다.
  확장성: 온디맨드 모드에서 실행하면 트래픽 패턴에 따라 자동으로 확장 및 축소할 수 있습니다.
  따라서, 1번 솔루션이 전자상거래 회사의 요구 사항을 충족하며, 확장성이 뛰어나고 실시간 분석이 가능합니다.

  정답은 1번입니다.

  전자상거래 플랫폼의 실시간 클릭스트림 데이터 처리에는 확장성과 실시간 처리 능력이 필수적입니다.

  Amazon Kinesis Data Streams는 변동하는 트래픽 패턴에 유연하게 대응하는 수직 및 수평 확장이 가능한 스트리밍 서비스입니다. 이는 높은 처리량과 낮은 지연 시간을 제공하여 실시간 분석에 적합합니다.

  AWS Lambda는 이벤트 기반 컴퓨팅 서비스로, Kinesis Data Streams에서 데이터가 도착할 때 자동으로 트리거되어 실시간 데이터 처리 로직을 실행합니다. 이는 서버 관리 없이 효율적인 실시간 분석 파이프라인 구축을 가능하게 합니다.

  다른 옵션들은 왜 적절하지 않은가요?

  2번: Kinesis Data Firehose는 주로 데이터를 저장소(예: S3)로 안정적으로 전달하는 데 초점을 맞추며, 실시간 처리보다는 배치 처리에 더 적합합니다. AWS Glue는 데이터 카탈로그링 및 ETL 작업에 강점을 가지고 있지만, 실시간 처리에는 오버헤드가 있습니다.
  3번: Kinesis Video Streams는 비디오 스트리밍 데이터에 최적화된 서비스입니다. 클릭스트림 데이터 처리에는 불필요한 기능과 비용이 발생합니다.
  4번: Apache Flink용 Amazon Managed Service는 강력한 스트리밍 처리 기능을 제공하지만, 문제에서 제시된 단순한 실시간 처리 요구 사항에는 AWS Lambda와 Kinesis Data Streams의 조합이 더 간결하고 효율적입니다.

889. A global company runs workloads on AWS. The company's applications use Amazon S3 buckets across AWS regions for sensitive data storage and analysis. The company stores millions of objects in multiple S3 buckets every day. The company wants to identify all S3 buckets that do not have versioning enabled.
Which solution meets these requirements?
① A. Set up an AWS CloudTrail event with a rule that identifies all S3 buckets in multiple regions that do not have versioning enabled.
② B. Use Amazon S3 Storage Lens to identify all S3 buckets across regions that do not have versioning enabled.
③ C. Enable IAM Access Analyzer for S3 to identify all S3 buckets that do not have versioning enabled across regions.
④ D. Create an S3 multi-region access point to identify all S3 buckets that do not have versioning enabled across regions.

  글로벌 기업은 AWS에서 워크로드를 실행하고 있으며, 여러 S3 버킷에 매일 수백만 개의 객체를 저장합니다. 버전 관리가 활성화되지 않은 모든 S3 버킷을 식별하려고 하는데, Amazon S3 Storage Lens를 사용하면 이러한 요구 사항을 충족할 수 있습니다. Amazon S3 Storage Lens는.centralized-dashboard와 몇 가지 메트릭을 제공하여 버전 yönetim 상태를 포함한 S3 버킷에 대한 정보를 제공합니다. 이를 통해 버전 관리가 활성화되지 않은 버킷을 쉽게 식별할 수 있습니다. 따라서, 정답은 Amazon S3 Storage Lens를 사용하는 것입니다.

  정답은 B. Amazon S3 Storage Lens입니다.

  Amazon S3 Storage Lens는 AWS 계정 내 여러 S3 버킷의 스토리지 사용 현황을 모니터링하고 분석하는 서비스입니다. 공간 사용량, 액세스 패턴, 객체 수명, 성능 지표 등 다양한 메트릭을 제공하며, 버전 관리 설정 여부를 포함한 버킷 설정 정보도 포함합니다. 따라서 글로벌 기업은 Storage Lens를 통해 리전 전체의 S3 버킷 중 버전 관리 기능이 활성화되지 않은 버킷들을 효율적으로 식별하고 분석할 수 있습니다.

  다른 옵션들은 요구 사항을 완전히 충족하지 못합니다:

  A. AWS CloudTrail: 이벤트 로그를 기록하지만, 버전 관리 상태를 직접 분석하는데는 제한적입니다.
  C. IAM 액세스 분석기: 주로 IAM 정책과 접근 권한을 분석하기 위한 도구로, S3 버킷의 버전 관리 상태를 탐지하는 데 적합하지 않습니다.
  D. 다중 지역 액세스 포인트: 주로 데이터 접근성을 개선하는 데 목적이 있으며, 버킷 설정 분석에는 사용되지 않습니다.

890. A company needs to optimize Amazon S3 storage costs for an application that generates a large number of irreproducible files. Each file is approximately 5 MB and is stored in Amazon S3 Standard storage.
The company must retain these files for four years before deleting them. The files must be immediately accessible. The files are frequently accessed during the first 30 days after object creation, but rarely accessed after that.
What is the most cost-effective solution to meet these requirements?
① A. Create an S3 lifecycle policy that moves files to S3 Glacier Instant Retrieval 30 days after object creation. Delete files 4 years after object creation.
② B. Create an S3 lifecycle policy that moves files to S3 One Zone-Infrequent Access (S3 One Zone-IA) 30 days after object creation. Delete files 4 years after object creation.
③ C. Create an S3 lifecycle policy that moves files to S3 Standard-Infrequent Access (S3 Standard-IA) 30 days after object creation. Delete files 4 years after object creation.
④ D. Create an S3 lifecycle policy that moves files to S3 Standard-Infrequent Access (S3 Standard-IA) 30 days after object creation. Move files to S3 Glacier Flexible Retrieval 4 years after object creation.

  이 문제는 Amazon S3 스토리지 비용을 최적화하는 방법을 묻고 있습니다. 회사의 요구 사항은 파일을 삭제하기 전에 4년 동안 보관해야 하며, 파일에 즉시 액세스할 수 있어야 합니다. 또한 파일은 처음 30일 동안 자주 액세스되지만 이후에는 거의 액세스되지 않습니다.

  이 요구 사항을 충족하는 가장 비용 효율적인 솔루션은 객체 생성 후 30일이 지나면 파일을 S3 Glacier Instant Retrieval로 이동하는 S3 수명 주기 정책을 생성하고, 객체 생성 후 4년이 지나면 파일을 삭제하는 것입니다. S3 Glacier Instant Retrieval는 자주 액세스하지 않는 데이터를 저장하기 위한 것으로, 데이터에 즉시 액세스할 수 있으며 비용이 저렴합니다.

  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다. S3 One Zone-Infrequent Access는 장애가 발생할 경우 데이터를 잃을 수 있기 때문에 적합하지 않습니다. S3 Standard-Infrequent Access는 즉시 액세스가 가능하지만, 비용이 더 높습니다. S3 Glacier 유연한 검색은 데이터에 즉시 액세스할 수 없기 때문에 적합하지 않습니다. 따라서 정답은 1번입니다.

  주어진 상황에서 가장 비용 효율적인 솔루션은 옵션 1입니다.

  파일은 초기 30일 동안 자주 액세스되지만 그 이후로는 액세스 빈도가 현저히 줄어듭니다. 이때, S3 Standard 스토리지는 높은 액세스 비용을 수반하므로, 초기 액세스 빈도가 낮아지는 시점에 비용 효율적인 저장 옵션으로 전환하는 것이 중요합니다.

  옵션 1은 30일이 지나면 파일을 S3 Glacier Instant Retrieval로 이동합니다. S3 Glacier Instant Retrieval은 높은 접근성을 유지하면서도 Standard 스토리지보다 훨씬 저렴한 비용으로 보관할 수 있습니다. 즉시 액세스가 필요한 상황에서도 빠르게 파일을 불러올 수 있는 장점이 있습니다. 4년 후 파일 삭제는 장기 보관 비용을 최소화합니다.

  옵션 2와 3 (S3 One Zone-IA, S3 Standard-IA)은 액세스 비용이 여전히 Standard보다 낮지만, Glacier Instant Retrieval에 비해 액세스 비용이 더 높을 수 있습니다. 특히 거의 액세스되지 않는 장기 보관 파일에 대해서는 비용 효율성이 떨어집니다.

  옵션 4는 중간 단계로 Standard-IA를 사용한 후 4년 후에 Glacier로 이동하는 방식으로, 초기 비용 절감 효과가 제한적일 수 있고, 4년 후 Glacier로의 이동은 추가적인 복잡성과 비용을 초래할 수 있습니다.

  따라서, 초기 액세스 빈도 감소 후 즉시 액세스가 필요하면서도 장기 보관 비용을 최소화하려는 요구 사항을 충족시키는 가장 적합한 선택은 옵션 1입니다.

891. A company runs a critical storage application in the AWS Cloud. The application uses Amazon S3 in two AWS regions. The company wants the application to send remote user data to the nearest S3 bucket without public network congestion. The company also wants application failover with minimal Amazon S3 management.
Which solution meets these requirements?
① A. Implement an active-active design between two regions. Configure your application to use the regional S3 endpoint closest to your users.
② B. Use an active-passive configuration for S3 multi-region access points. Create a global endpoint for each region.
③ C. Send user data to the regional S3 endpoint closest to the user. Configure S3 cross-account replication rules to keep S3 buckets synchronized.
④ D. Set up Amazon S3 to use multi-region access points in an active-active configuration with a single global endpoint. Configure S3 cross-region replication.

  회사가 요구하는 조건은 공용 네트워크 정체 없이 원격 사용자 데이터를 가장 가까운 S3 버킷으로 보내는 것이며, 최소한의 Amazon S3 관리로 애플리케이션 장애 조치를 원합니다.

  4번은 단일 글로벌 엔드포인트가 있는 활성-활성 구성에서 다중 지역 액세스 포인트를 사용하도록 Amazon S3를 설정하며, S3 교차 리전 복제를 구성합니다. 따라서 정답은 4번입니다.

  이 방법은 사용자의 요청을 가장 가까운 리전에自動으로 라우팅하는 다중 지역 액세스 포인트를 사용하여 공용 네트워크 정체를 최소화하며, S3 교차 리전 복제를 사용하여 데이터 일관성을 유지하고 최소한의 관리로 장애 조치를 제공합니다.

  정답인 4번 솔루션이 요구 사항을 가장 잘 충족합니다.

  사용자 근접성: 단일 글로벌 엔드포인트를 통해 사용자는 항상 가장 가까운 AWS 리전의 S3에 연결되므로 네트워크 지연과 정체를 최소화할 수 있습니다.
  장애 조치: 활성-활성 구성 덕분에 한 리전에 문제가 발생하더라도 다른 리전에서 서비스를 계속 제공할 수 있어 가용성이 향상됩니다.
  관리 간소화: S3 교차 리전 복제를 설정하면 데이터의 동기화를 자동화할 수 있어 수동 관리 부담을 줄입니다. 이로써 두 리전의 S3 버킷이 일관된 상태를 유지하면서도, 복잡한 다중 관리 없이 장애 상황에 대비할 수 있습니다.
  따라서 4번 옵션이 공용 네트워크 효율성, 장애 대비 능력, 그리고 관리 효율성을 모두 만족시킵니다.

892. A company is migrating its data center from an on-premises location to AWS. The company has several legacy applications hosted on individual virtual servers. The application design cannot be changed.
Each individual virtual server currently runs on its own EC2 instance. The solutions architect must ensure the application's stability and fault tolerance after the migration to AWS. The application runs on Amazon EC2 instances.
Which solution meets these requirements?
① A. Create at least one, and at most one, Auto Scaling group. Create an Amazon Machine Image (AMI) for each application instance. Use the AMI to create EC2 instances in the Auto Scaling group. Configure an Application Load Balancer in front of the Auto Scaling group.
② B. Use AWS Backup to create hourly backups of the EC2 instances hosting each application. Store the backups in Amazon S3 in separate Availability Zones. Configure your disaster recovery process to restore each application's EC2 instance from the most recent backup.
③ C. Create an Amazon Machine Image (AMI) for each application instance. Launch two new EC2 instances from the AMI. Place each EC2 instance in a separate Availability Zone. Configure a Network Load Balancer targeting the EC2 instances.
④ D. Use AWS Mitigation Hub Refactor Spaces to migrate each application from an EC2 instance. Break down each application's functionality into individual components. Host each application on Amazon Elastic Container Service (Amazon ECS) using the AWS Fargate launch type.

  각 애플리케이션은 개별 가상 서버에서 호스팅되며 AWS로 마이그레이션한 후 안정성과 내결함성을 보장해야 합니다. 현재는 각 가상 서버가 자체 EC2 인스턴스로 실행되고 있습니다.

  각 애플리케이션 인스턴스의 Amazon 머신 이미지를 생성하고 AMI에서 2개의 새로운 EC2 인스턴스를 시작합니다. 이후 각 EC2 인스턴스를 별도의 가용 영역에 배치하고 EC2 인스턴스를 대상으로 하는 Network Load Balancer를 구성하면 안정성과 내결함성을 보장할 수 있습니다.

  이러한 요구 사항을 충족하는 솔루션은 C입니다.

  정답은 3번입니다. 각 애플리케이션의 내결함성과 안정성을 보장하기 위해 주요 요구 사항은 고가용성과 재해 복구를 포함한 중복성입니다.

  3번 옵션은 다음과 같은 방식으로 이 요구 사항을 충족합니다:

  고가용성: 각 애플리케이션을 위한 AMI를 생성한 후, 두 개의 EC2 인스턴스를 별도의 가용 영역에 배포합니다. 이는 하나의 가용 영역에 문제가 발생하더라도 다른 가용 영역에서 서비스가 계속되도록 합니다.
  로드 밸런싱: Network Load Balancer를 통해 트래픽을 두 인스턴스 사이에 분산시킵니다. 이는 요청의 효율적인 처리와 장애 시 자동 전환 기능을 제공하여 시스템의 안정성을 높입니다.
  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  1번: Auto Scaling 그룹은 확장성과 부하 분산에 유용하지만, 고가용성을 위해 최소 두 개 이상의 인스턴스를 별도의 가용 영역에 배치하는 명시적인 언급이 없습니다.
  2번: 백업과 재해 복구는 중요한 요소이지만, 실시간 안정성과 내결함성을 즉시 보장하는 솔루션은 아닙니다. 백업 복원은 재해 발생 후 복구 과정에 초점을 맞춥니다.
  4번: 애플리케이션을 컨테이너화하고 Fargate를 사용하는 것은 현대적인 접근법이지만, 문제 조건에서 애플리케이션 디자인 변경을 피하고 현재 EC2 기반 구조를 유지해야 한다는 점을 고려할 때 적합하지 않습니다. 또한, 이 방법은 질문의 핵심 요구 사항인 고가용성과 직접적인 연결성이 덜 명확합니다.

893. A company wants to isolate workloads by creating AWS accounts for each workload. The company needs a solution that centrally manages networking components for the workloads. The solution also needs to create accounts with automated security controls (guardrails). What solution can meet these requirements with minimal operational overhead?
① A. Deploy accounts using AWS Control Tower. Create a networking account with a VPC with private and public subnets. Use AWS Resource Access Manager (AWS RAM) to share the subnets with workload accounts.
② B. Deploy accounts using AWS Organizations. Create a networking account with a VPC with private and public subnets. Use AWS Resource Access Manager (AWS RAM) to share the subnet with the workload account.
③ C. Deploy accounts using AWS Control Tower. Deploy a VPC for each workload account. Configure each VPC to route through the inspection VPC using a transit gateway connection.
④ D. Distribute accounts using AWS Organizations. Deploy a VPC for each workload account. Configure each VPC to route through the inspection VPC using a transit gateway connection.

  회사는 워크로드 격리 및 중앙 집중식 네트워크 구성 요소 관리를 목표로 하고 있습니다. 자동 보안 제어도 포함되어야 합니다.

  AWS Organizations를 사용하면 중앙 집중식으로 계정을 관리할 수 있으며 워크로드 격리를 제공해 줍니다. 네트워킹 계정을 생성하여 프라이빗 서브넷과 퍼블릭 서브넷이 있는 VPC를 관리할 수 있습니다. AWS Resource Access Manager(AWS RAM)를 사용하여 워크로드 계정과 서브넷을 공유할 수 있습니다.

  自動 보안 제어를 포함한 계정을 생성할 수 있기 때문에AWS Organizations를 사용하여 계정을 배포하고, 프라이빗 서브넷과 퍼블릭 서브넷이 있는 VPC가 있는 네트워킹 계정을 생성하여, AWS Resource Access Manager(AWS RAM)를 사용하여 워크로드 계정과 서브넷을 공유하는 것이 적절한 솔루션입니다.

  정답은 2번입니다. AWS Organizations를 사용하는 것이 최적의 솔루션입니다. 이 접근법은 다음과 같은 이유로 적합합니다:

  중앙화된 관리: AWS Organizations는 여러 AWS 계정을 중앙에서 관리할 수 있게 해줍니다. 이로 인해 네트워킹 구성 요소와 보안 정책을 효율적으로 일관성 있게 적용할 수 있습니다.

  중앙 보안 제어: AWS Organizations는 자동 보안 제어 기능을 통해 계정 생성 시 기본적인 가드레일을 설정할 수 있습니다. 이를 통해 각 워크로드 계정이 기본적인 보안 기준을 충족하도록 할 수 있습니다.

  VPC 활용: 각 워크로드에 독립적인 VPC를 생성하되, 단일 네트워킹 계정을 통해 중앙에서 VPC의 기본 구조(프라이빗 및 퍼블릭 서브넷)를 관리할 수 있습니다. 이는 AWS Resource Access Manager (AWS RAM)를 통해 공유 리소스를 효율적으로 관리할 수 있음을 의미하지만, 각 워크로드의 격리성을 유지하는 데 있어 직접적인 서브넷 공유보다는 개별 VPC 관리가 더 적합합니다.

  1번과 4번은 서브넷 공유를 제안하는데, 이는 워크로드 간의 격리성을 저하시킬 수 있습니다. 3번은 각 워크로드에 독립 VPC를 생성하지만, 중앙 네트워킹 관리와 자동 보안 제어 측면에서 AWS Organizations의 강점을 완전히 활용하지 못합니다. 따라서, 최소한의 운영 오버헤드로 중앙 관리와 보안 제어를 동시하게 충족시키는 가장 적합한 솔루션은 2번입니다.

894. The company hosts its website on Amazon EC2 instances behind an Application Load Balancer (ALB). The website serves static content. Website traffic is increasing. The company wants to minimize website hosting costs. Which solution meets these requirements?
① A. Move your website to an Amazon S3 bucket. Configure an Amazon CloudFront distribution for the S3 bucket.
② B. Move your website to an Amazon S3 bucket. Configure an Amazon ElastiCache cluster for the S3 bucket.
③ C. Move your website to AWS Amplify. Configure ALB to view your Amplify website.
④ D. Move your website to AWS Amplify. Configure your EC2 instance to cache your website.

  회사는 웹사이트 호스팅 비용을 최소화하려고 합니다. 이를 위해서는 정적 콘텐츠를 제공하는 웹사이트를 적절한 서비스로 이전하여 비용을ลด 가입해야 합니다.

  Amazon S3 버킷은 정적 콘텐츠를 저장하고 제공하기 적합한 서비스입니다. 또한 Amazon CloudFront를 사용하면 임시저장된 콘텐츠를 캐시하여 엣지 로케이션에 배포할 수 있습니다. 이렇게 하면 사용자에게 더 빠른 콘텐츠 제공과 더 낮은 비용이 가능합니다.

  따라서 정적 콘텐츠를 제공하는 웹사이트를 Amazon S3 버킷으로 이동시키고 S3 버킷에 대한 Amazon CloudFront 배포를 구성하는 것이 최선의 솔루션입니다. 이 접근 방식은 비용을 절감하고 웹 사이트 성능을 cải善하는 데 도움이 됩니다.

  정답은 1번입니다.

  정적 콘텐츠를 제공하는 웹사이트의 경우, 직접적인 서버 인스턴스 관리 없이도 효율적으로 배포하고 비용을 절감할 수 있는 방법이 필요합니다.

  Amazon S3 버킷 사용: 정적 콘텐츠는 S3에 저장하면 저렴한 저장 비용과 높은 가용성을 얻을 수 있습니다.
  Amazon CloudFront 활용: S3 버킷을 CloudFront 배포와 연계하면, 전 세계적으로 캐싱 및 콘텐츠 배포를 통해 트래픽 부하를 분산시키고, 네트워크 지연 시간을 줄일 수 있습니다. 이는 추가적인 서버 비용을 최소화하고 성능을 향상시킵니다.
  다른 옵션들은 요구 사항에 부합하지 않습니다:

  B 옵션은 ElastiCache는 주로 데이터베이스 캐싱에 사용되며, 정적 콘텐츠 최적화에는 적합하지 않습니다.
  C와 D 옵션에서 AWS Amplify는 웹 애플리케이션 개발에 유용하지만, 정적 콘텐츠 최적화에 있어서 CloudFront와 S3의 조합만큼 비용 효율적이지 않을 수 있습니다. 특히 D 옵션은 여전히 EC2 인스턴스를 사용하므로 관리 비용과 운영 복잡성이 증가할 수 있습니다.
  따라서, 비용 최소화와 성능 최적화를 동시에 추구하는 가장 효과적인 방법은 정적 콘텐츠를 S3에 저장하고 CloudFront를 통해 배포하는 1번 옵션입니다.

895. A company is implementing a shared storage solution for its media application hosted on AWS. The company needs the ability to access stored data using SMB clients. What solution meets these requirements with minimal management overhead?
① A. Create an AWS Storage Gateway volume gateway. Create a file share using the required client protocol. Connect your application server to the file share.
② B. Create an AWS Storage Gateway tape gateway. Configure the tape to use Amazon S3. Connect your application server to the tape gateway.
③ C. Create an Amazon EC2 Windows instance. Install and configure the Windows File Sharing role on the instance. Connect the application server to the file share.
④ D. Create an Amazon FSx file system for your Windows file server. Connect your application servers to the file system.

  최소한의 관리 오버헤드로 SMB 클라이언트를 사용하여 저장된 데이터에 액세스할 수 있는 기능을 제공하는 솔루션은 Windows 파일 서버용 Amazon FSx 파일 시스템을 생성하여 애플리케이션 서버를 파일 시스템에 연결하는 것입니다. 이 방법은 관리 오버헤드를 줄여주면서도 SMB 클라이언트를 사용하여 데이터에 액세스할 수 있는 기능을 제공합니다. 다른 방법들은 관리 오버헤드가 더 클 수 있으며, 필요한 기능을 제공하지 않을 수 있습니다. 따라서 Windows 파일 서버용 Amazon FSx 파일 시스템을 사용하는 것이 최적의 솔루션입니다.

  정답은 4번입니다. Amazon FSx for Windows File Server는 AWS에서 관리되는 완전 관리형 서비스로, SMB 프로토콜을 지원하여 기존 Windows 환경과 원활한 호환성을 제공합니다. 이 솔루션은 다음과 같은 이유로 적합합니다:

  최소화된 관리 오버헤드: FSx는 AWS가 직접 관리하므로, 파일 시스템의 패치, 업데이트, 성능 최적화 등 복잡한 관리 작업이 필요 없습니다.
  SMB 액세스: 애플리케이션에서 요구하는 대로 SMB 클라이언트를 통한 직접적인 파일 액세스가 가능합니다.
  고가용성 및 확장성: 내장된 고가용성 기능과 쉽게 확장 가능한 용량으로 안정적이고 유연한 스토리지 환경을 제공합니다.
  다른 옵션들과 비교했을 때:

  1번의 AWS Storage Gateway는 주로 온프레미스 스토리지와 AWS 클라우드 스토리지 간의 중개 역할을 하는데 초점을 맞추고 있어, 직접적인 SMB 액세스와 관리 편의성 측면에서 제한적입니다.
  2번의 테이프 게이트웨이는 백업 및 아카이브 용도에 더 적합하며, 실시간 SMB 액세스를 제공하지 않습니다.
  3번의 EC2 인스턴스 기반 파일 공유는 수동 관리가 필요하며, 인스턴스의 유지보수와 보안 설정 등 추가적인 오버헤드가 발생합니다.

896. A company is designing a disaster recovery (DR) strategy for its production application. The application is supported by a MySQL database on an Amazon Aurora cluster in the us-east-1 region. The company has selected the us-west-1 region as its DR region.
The company's target recovery point objective (RPO) is 5 minutes and its target recovery time objective (RTO) is 20 minutes. The company wants to minimize configuration changes.
What solution will meet these requirements with the most efficient operational efficiency?
① A. Create an Aurora read replica in us-west-1 with a size similar to the Aurora MySQL cluster writer instance of your production application.
② B. Convert the Aurora cluster to an Aurora global database. Configure managed failover.
③ C. Create a new Aurora cluster in us-west-1 with cross-region replication.
④ D. Create a new Aurora cluster in us-west-1. Synchronize both clusters using AWS Database Migration Service (AWS DMS).

  회사의 목표는 5분의 RPO와 20분의 RTO를 달성하면서 구성 변경을 최소화하는 것입니다. 이 요구 사항을 충족하는 가장 효율적인 솔루션은 Aurora 클러스터를 Aurora 글로벌 데이터베이스로 변환하고 관리형 장애 조치를 구성하는 것입니다.

  이러한 접근 방식은 장애가 발생할 경우 us-west-1 지역에서 자동으로 장애 조치를 수행할 수 있으므로 RTO 요구 사항을 충족할 수 있습니다. 또한 글로벌 데이터베이스는 데이터를 실시간으로 복제하므로 RPO 요구 사항도 충족할 수 있습니다.

  他の 옵션은 이러한 요구 사항을 충족하지 못하거나 구성 변경이 더 많이 필요할 수 있습니다. 예를 들어, 읽기 전용 복제본을 생성하거나 교차 리전 복제를 사용하는 경우 데이터를 수동으로 동기화하거나 구성 변경이 더 많이 필요할 수 있습니다.

  따라서 Aurora 클러스터를 Aurora 글로벌 데이터베이스로 변환하고 관리형 장애 조치를 구성하는 것이 가장 효율적인 솔루션입니다.

  회사의 재해 복구 목표인 RPO 5분과 RTO 20분을 효과적으로 달성하고 구성 변경을 최소화하려면, B 옵션이 가장 적합합니다.

  B 옵션은 Aurora 글로벌 데이터베이스를 활용하는 방법입니다. 이 솔루션은 다음과 같은 이점을 제공합니다:

  실시간 복제: Aurora 글로벌 데이터베이스는 자동으로 리전 간성 실시간 복제를 지원하므로 RPO 5분을 쉽게 만족시킵니다.
  자동 장애 조치: 내장된 관리형 장애 조치 기능 덕분에 DR 상황 발생 시 수동 개입 없이 문제 지역에서 복구 지역으로 빠르게 전환이 가능해 RTO 20분을 충족할 수 있습니다.
  구성 최소화: 기존 구성을 크게 변경하지 않고 글로벌 데이터베이스로 업그레이드만 하면 되므로, 회사의 요구 사항인 구성 변경 최소화를 실현할 수 있습니다.
  A 옵션은 읽기 전용 복제본만 제공해 DR 상황에서 데이터 동기화 지연이 발생할 수 있습니다. C 옵션과 D 옵션은 별도의 클러스터 생성 및 추가적인 동기화 서비스 필요로 인해 복잡성과 관리 부담이 증가하며, RTO와 RPO 목표 달성에 대한 보안이 덜합니다. 따라서, 가장 효율적이고 간단한 솔루션은 B 옵션입니다.

897. The company runs a critical data analysis task every week before the first day of the week. The task requires at least an hour to complete the analysis. The task is stateful and cannot be interrupted. The company needs a solution to run the task on AWS. Which solution meets these requirements?
① A. Create a container for the task. Use Amazon EventBridge Scheduler to schedule the task to run as an AWS Fargate task on an Amazon Elastic Container Service (Amazon ECS) cluster.
② B. Configure the task to run on an AWS Lambda function. Create a scheduled rule in Amazon EventBridge to invoke the Lambda function.
③ C. Configure an Auto Scaling group of Amazon EC2 Spot Instances running Amazon Linux. Configure a crontab entry on the instances to run the analysis.
④ D. Configure an AWS DataSync task to run the task. Configure a cron expression to run the task on a schedule.

  정답인 1번은 상태를 저장하며 중단을 허용할 수 없는 작업에 적합합니다. Amazon ECS 클러스터에서 AWS Fargate 작업으로 실행되도록 예약하기 때문입니다. 이론상으로는 상태를 저장할 수 없는 AWS Lambda에서 실행하거나 Auto Scaling 그룹을 구성하여 EC2 인스턴스에서 실행하는것 보다 가장 적합한 방법으로 보입니다. AWS DataSync 작업은 주로 데이터 동기화에 사용되므로 이러한 작업에는 적합하지 않습니다.

  회사의 요구 사항은 중단 없이 실행되어야 하며, 고정된 시간(최소 1시간)이 필요한 데이터 분석 작업을 매주 일정한 시간에 자동으로 실행하는 것입니다. 각 옵션을 살펴보면:

  B 옵션 (AWS Lambda): Lambda 함수는 일반적으로 짧은 실행 시간을 목표로 하며, 설정된 시간 제한(예: 15분 이상의 확장 가능하지만 여전히 제한적)을 초과하는 장시간 작업에 적합하지 않습니다.
  C 옵션 (EC2 스팟 인스턴스): EC2 인스턴스는 유연하지만, 스팟 인스턴스의 가용성 불안정성과 관리 복잡성은 요구 사항에 맞지 않습니다. 또한, Spot 인스턴스의 중단 가능성은 작업의 중단 없는 실행을 보장하지 않습니다.
  D 옵션 (AWS DataSync): DataSync는 주로 대용량 데이터의 동기화에 사용되며, 복잡한 분석 작업을 직접 실행하는 데 적합하지 않습니다.
  정답: 1번 등은 다음과 같이 적합합니다:

  Amazon ECS와 AWS Fargate:
  상태 저장 및 중단 불가능성: Fargate는 컨테이너 기반으로 작업의 상태를 유지하고 전체 작업이 완료되는 동안 실행될 수 있어 중단 없이 작업을 보장합니다.
  예약 및 자동화: Amazon EventBridge Scheduler를 사용하면 정확한 일정에 맞춰 작업을 예약할 수 있습니다. 이는 매주 일정 시간에 분석 작업을 자동으로 시작하도록 설정하는 데 이상적입니다.
  시간 요구 사항: Fargate는 장시간 작업을 지원하므로 최소 1시간이 필요한 분석 작업을 처리할 수 있습니다.
  따라서, 주어진 조건을 가장 잘 충족하는 솔루션은 1번 옵션입니다.

898. A company runs workloads in the AWS cloud. It wants to centrally collect security data to assess company-wide security and improve workload protection. What solution can meet these requirements with minimal development effort?
① A. Build a data lake in AWS Lake Formation. Use AWS Glue crawlers to ingest security data into the data lake.
② B. Configure an AWS Lambda function to collect security data in .csv format. Upload the data to an Amazon S3 bucket.
③ C. Configure a data lake in Amazon Security Lake to collect security data. Upload the data to an Amazon S3 bucket.
④ D. Configure an AWS Database Migration Service (AWS DMS) replication instance to load security data into an Amazon RDS cluster.

  회사는 보안 데이터를 중앙에서 수집하여 회사 전체의 보안을 평가하고 워크로드 보호를 개선하고자 합니다. 이를 위해 최소한의 개발 노력으로 이러한 요구 사항을 충족하는 솔루션을 찾고 있습니다.

  이 경우, Amazon Security Lake를 사용하여 보안 데이터를 수집하고 데이터 레이크를 구성하는 것이 적절한 솔루션입니다. Amazon Security Lake는 보안 관련 데이터를 한 곳에集中하여 분석하고 평가할 수 있는 서비스로, 회사의 보안을 평가하고 워크로드 보호를 개선하는 데 도움이 됩니다.

  따라서, 정답은 C입니다. 다른 옵션은 회사의 요구 사항을 충족하지 않거나, 더 많은 개발 노력이 필요하여 적절한 솔루션이 아닙니다.

  Amazon Security Lake는 보안 관련 데이터를 중앙 집중적으로 수집하고 관리하도록 설계된 서비스입니다. 이 서비스는 여러 AWS 서비스와 외부 소스에서 보안 데이터를 자동으로 수집하여 분석과 모니터링을 용이하게 합니다. 이를 통해 회사는 최소한의 개발 노력으로 보안 데이터를 효율적으로 통합하고 분석할 수 있습니다.

  1번 옵션의 AWS Lake Formation과 AWS Glue는 데이터 레이크 구축에 유용하지만, 보안 특화 기능이 부족합니다. 2번 옵션의 AWS Lambda와 S3는 데이터 저장에 적합하지만, 중앙 집중적 보안 데이터 관리와 분석을 위한 특화된 기능이 부족합니다. 4번 옵션의 AWS DMS와 RDS는 주로 데이터베이스 마이그레이션과 관리에 초점을 맞추고 있어 보안 데이터의 중앙집합 및 분석에는 적합하지 않습니다.

  따라서, 보안 데이터의 중앙 집중적 수집과 관리, 그리고 향상된 보안 평가와 워크로드 보호를 위한 가장 적합한 솔루션은 Amazon Security Lake (C 옵션)입니다.

899. A company is migrating five on-premises applications to a VPC in the AWS Cloud. Each application is currently deployed in an isolated virtual network on-premises and needs to be similarly deployed in the AWS Cloud. The applications must be connected to a shared services VPC. All applications must be able to communicate with each other.
If the migration is successful, the company plans to repeat the migration process for over 100 applications.
What solution can meet these requirements with minimal management overhead?
① A. Deploy a software VPN tunnel between the application VPC and the shared services VPC. Add a route between the application VPC and the shared services VPC in the corresponding subnet.
② B. Deploy a VPC peering connection between the application VPC and the shared services VPC. The peering connection adds a route between the application VPC and the shared services VPC in the corresponding subnet.
③ C. Deploy an AWS Direct Connect connection that routes from the application VPC to the shared services VPC and application VPC in the shared services VPC subnet. Add a route from the shared services VPC subnet to the application VPC.
④ D. Deploy a transit gateway with connections between the application VPC and the shared services VPC. Add a route between the application VPC and the shared services VPC for the corresponding subnet through the transit gateway.

  회사는 온프레미스 애플리케이션을 AWS 클라우드의 VPC로 마이그레이션하고 있습니다. 각 애플리케이션은 현재 격리된 가상 네트워크에 배포되어 있습니다. 따라서 AWS 클라우드에도 유사한 배포가 필요하며, 애플리케이션은 공유 서비스 VPC에 연결되어야 합니다.

  이러한 요구 사항을 충족하는 최소한의 관리 오버헤드 솔루션은 전송 게이트웨이를 사용하는 것입니다. 전송 게이트웨이는 중앙의 허브 역할을 하여 여러 VPC와 공유 서비스 VPC 간의 통신을 가능하게 합니다. 따라서 각 애플리케이션 VPC와 공유 서비스 VPC에 대한 경로를 추가하여 모든 애플리케이션이 서로 통신할 수 있게 합니다.

  적용 예로, 전송 게이트웨이를 배포하면 각 애플리케이션 VPC와 공유 서비스 VPC 간의 연결을 설정할 수 있으며, 이는 관리 오버헤드를 최소화하는 솔루션입니다. 또한, 전송 게이트웨이는 하나의 중앙 허브로 여러 VPC를 연결해 줌으로써, Scalability와 관리 효율성을 제공합니다.

  따라서, 정답은 4번입니다. 전송 게이트웨이를 사용하면 여러 애플리케이션 VPC와 공유 서비스 VPC 간의 통신을 효율적으로 관리할 수 있습니다.

  정답은 4번입니다. A회사가 요구하는 아키텍처는 확장성과 관리 오버헤드 최소화를 중요시합니다.

  1번 옵션의 소프트웨어 VPN은 수동 설정이 필요하고, 많은 VPC 간 연결을 관리하기 어렵습니다. 2번의 VPC 피어링은 VPC 간 직접 연결을 제공하지만, 대규모 확장성과 중앙 관리 기능이 부족합니다. 3번의 AWS Direct Connect는 고정된 연결을 제공하지만, 여러 VPC 간의 동적 관리와 확장성 측면에서 제약이 있습니다.

  4번 옵션인 전송 게이트웨이(Transit Gateway)는 이러한 문제를 해결합니다. Transit Gateway는 여러 VPC와 온프레미스 환경을 중앙에서 관리할 수 있어, 애플리케이션 VPC들과 공유 서비스 VPC 간의 통신을 간편하게 처리하고 확장이 용이합니다. 또한, 중앙 집중식 라우팅 관리로 인해 오버헤드를 최소화하며, 100개 이상의 애플리케이션에 대한 반복적인 마이그레이션 프로세스에도 효과적입니다. 이는 회사의 장기적인 확장 전략에 가장 적합한 솔루션입니다.

900. A company wants to run an on-premises application in a hybrid environment using Amazon Elastic Container Service (Amazon ECS). The application currently runs in an on-premises container.
The company needs a single container solution that can scale across on-premises, hybrid, or cloud environments. The company needs to run the new application container in the AWS Cloud and use a load balancer for HTTP traffic.
What combination of actions would meet these requirements? (Choose two.)
① A. Set up an ECS cluster using the AWS Fargate launch type for cloud application containers. Use the Amazon ECS Anywhere external launch type for on-premises application containers.
② B. Set up an Application Load Balancer for the cloud ECS service.
③ C. Set up a Network Load Balancer for the cloud ECS service.
④ D. Set up an ECS cluster using the AWS Fargate launch type. Use Fargate for both cloud application containers and on-premises application containers.
⑤ E. Set up an ECS cluster using the Amazon EC2 launch type for cloud application containers. Use Amazon ECS Anywhere with the AWS Fargate launch type for on-premises application containers.

  회사는 온프레미스와 하이브리드 또는 클라우드 환경에서 확장할 수 있는 단일 컨테이너 솔루션이 필요합니다. 회사는 AWS 클라우드에서 새로운 애플리케이션 컨테이너를 실행해야 하며 HTTP 트래픽용 로드 밸런서를 사용해야 합니다.

  이러한 요구 사항을 충족하는 작업 조합을 찾는 데 있습니다. 2개의 작업을 선택해야 하는데 다음 두 가지 방법이 있습니다.

  첫 번째 방법은 클라우드 애플리케이션 컨테이너에 대해 AWS Fargate 시작 유형을 사용하는 ECS 클러스터를 설정하고 온프레미스 애플리케이션 컨테이너에는 Amazon ECS Anywhere 외부 시작 유형을 사용하는 것입니다.

  두 번째 방법은 클라우드 ECS 서비스를 위한 Application Load Balancer를 설정하는 것인데 이는 HTTP 트래픽용 로드 밸런서를 사용해야 한다는 요구 사항을 충족합니다.

  따라서 두 번째 방법이 더 적절한 선택입니다. 다른 방법을 사용할 경우 AWS 클라우드에서 새로운 애플리케이션 컨테이너를 실행해야 한다는 요구 사항을 만족시키지 못합니다.

  따라서 정답은 클라우드 애플리케이션 컨테이너에 대해 AWS Fargate 시작 유형을 사용하는 ECS 클러스터를 설정하고 온프레미스 애플리케이션 컨테이너에는 Amazon ECS Anywhere 외부 시작 유형을 사용하며 클라우드 ECS 서비스를 위한 Application Load Balancer를 설정하는 것입니다.

  문제의 핵심은 하이브리드 환경에서 온프레미스와 클라우드 양쪽의 애플리케이션 컨테이너를 확장 가능하게 실행하면서, 클라우드 측에는 HTTP 트래픽을 처리할 수 있는 로드 밸런서를 구축하는 방법을 찾는 것입니다.

  옵션 B는 클라우드 내 ECS 서비스에 Application Load Balancer를 설정하는 방법을 제시합니다. 이는 HTTP 트래픽을 효과적으로 분산시키고 관리하는 데 필요한 조건을 충족합니다.
  옵션 A와 D는 온프레미스와 클라우드 모두에 Fargate 또는 ECS 클러스터를 사용하는 방법을 제시하지만, 문제에서 명시적으로 요구한 HTTP 트래픽을 위한 로드 밸런서 설정에 대한 직접적인 해결책을 제공하지 않습니다. 특히 온프레미스 환경에서의 Fargate 지원은 제한적입니다.
  옵션 C는 Network Load Balancer를 제시하지만, 이는 주로 TCP/UDP 트래픽에 더 적합하며 문제에서 요구하는 HTTP 트래픽 처리에는 적합하지 않습니다.
  옵션 E는 EC2와 Fargate를 혼합 사용하는 방법을 제시하지만, 역시 HTTP 트래픽을 위한 로드 밸런서 설정에 대한 명확한 해결책을 제공하지 않으며, 온프레미스 환경에서의 구현도 복잡합니다.
  따라서, 주어진 요구 사항 중 특히 HTTP 트래픽을 위한 로드 밸런서 설정이라는 핵심 요소를 충족시키는 작업 조합은 2번 (클라우드 ECS 서비스를 위한 Application Load Balancer 설정)입니다. 다른 요구 사항들을 충족시키기 위한 추가적인 구성이 필요할 수 있으나, 주어진 보기 중에서는 이 옵션이 가장 적합합니다.

901. A company is migrating its workloads to AWS. The company has sensitive and critical data in its on-premises relational database running on a SQL Server instance. The company wants to use the AWS Cloud to enhance security and reduce database operational overhead. Which solution meets these requirements?
① A. Migrate the database to an Amazon EC2 instance. Use AWS Key Management Service (AWS KMS) AWS-managed keys for encryption.
② B. Migrate the database to a multi-AZ Amazon RDS SQL Server DB instance. Use AWS Key Management Service (AWS KMS) AWS-managed keys for encryption.
③ C. Migrate data to an Amazon S3 bucket. Use Amazon Macie to ensure data security.
④ D. Migrate your database to an Amazon DynamoDB table. Use Amazon CloudWatch Logs to ensure data security.

  회사가 보안을 강화하고 운영 오버헤드를 줄일 수 있는 솔루션은 관계형 데이터베이스를 SQL Server DB 인스턴스용 다중 AZ Amazon RDS로 마이그레이션하는 것입니다. 암호화를 위해 AWS Key Management Service(AWS KMS) AWS 관리형 키를 사용하여 데이터를 보호할 수 있습니다. 이렇게 하면 데이터베이스의 가용성과 내구성을 높일 수 있으며, 자동 백업과 패치 같은 관리 기능을 이용할 수 있습니다. 또한 다중 AZ를 사용하면 고가용성을 제공할 수 있으므로 데이터베이스의 운영 오버헤드를 줄일 수 있습니다.

  회사의 요구 사항은 민감한 데이터의 보안 강화와 운영 오버헤드 감소입니다. 이에 따라 가장 적합한 솔루션은:

  2번: SQL Server 데이터베이스를 다중 AZ(Availability Zone) 설정의 Amazon RDS 인스턴스로 마이그레이션하는 것입니다. RDS는 관리형 서비스이므로 운영 오버헤드를 크게 줄일 수 있습니다. 또한, AWS KMS를 활용한 암호화는 데이터 보안을 강화합니다. 다중 AZ 옵션은 고가용성과 재해 복구를 지원하여 데이터의 안정성과 신뢰성을 높입니다.
  다른 옵션들은 다음과 같은 이유로 적합하지 않습니다:

  1번: EC2 인스턴스로 직접 관리하는 방식은 여전히 높은 운영 오버헤드를 수반합니다.
  3번: S3는 객체 스토리지이며, 관계형 데이터베이스의 복잡한 트랜잭션 처리와 직접적인 SQL 지원이 부족합니다. Macie는 데이터 탐지 및 보호에 도움이 되지만, 데이터베이스 운영 자체의 효율성은 크게 개선되지 않습니다.
  4번: DynamoDB는 NoSQL 데이터베이스로, 기존 SQL Server의 관계형 데이터베이스 특성을 완전히 대체하기 어렵습니다. 또한, CloudWatch Logs는 모니터링과 로깅에 중점을 두며, 직접적인 데이터 보안 강화에는 한계가 있습니다.
  따라서, 보안과 운영 효율성을 동시에 만족시키는 최적의 선택은 2번입니다.

902. My company is planning to migrate its applications to AWS. The company wants to improve the current availability of its applications. The company plans to use AWS WAF in its application architecture. Which solution meets these requirements?
① A. Create an Auto Scaling group containing multiple Amazon EC2 instances that host your application across two Availability Zones. Configure an Application Load Balancer (ALB) and set the Auto Scaling group as the target. Connect WAF to the ALB.
② B. Create a cluster placement group containing multiple Amazon EC2 instances hosting your application. Configure an Application Load Balancer and set the EC2 instances as targets. Attach WAF to the placement group.
③ C. Create two Amazon EC2 instances to host the application across two Availability Zones. Configure the EC2 instances as targets for an Application Load Balancer (ALB). Connect the WAF to the ALB.
④ D. Create an Auto Scaling group containing multiple Amazon EC2 instances that host the application across two Availability Zones. Configure an Application Load Balancer (ALB) and set the Auto Scaling group as the target. Attach WAF to the Auto Scaling group.

  애플리케이션의 가용성을 높이기 위해 여러 개의 가용 영역에서 실행되는 인스턴스를 사용해야 한다. 또한 Auto Scaling 그룹을 사용하여 인스턴스의 수를 자동으로 조정할 수 있어야 한다. 또한 Application Load Balancer를 사용하여 트래픽을 분산하고, WAF를 연결하여 보안을 강화할 수 있어야 한다.
  이러한 요구 사항을 충족하는 솔루션은 두 개의 가용 영역에 걸쳐 애플리케이션을 호스팅하는 여러 Amazon EC2 인스턴스를 포함하는 Auto Scaling 그룹을 생성하고, Application Load Balancer를 구성하여 Auto Scaling 그룹을 대상으로 설정한 후, WAF를 ALB에 연결하는 것이다. 이 솔루션은 가용성과 확장성을 제공同时 보안도 강화한다.

  정답은 1번입니다. 회사의 요구 사항을 충족하는 가장 효과적인 방법은 다음과 같은 이유들로 인해 1번 옵션이 적합합니다:

  다중 가용 영역 활용: 두 개의 가용 영역에 걸쳐 EC2 인스턴스를 배치하여 자연스러운 재해 복구와 높은 가용성을 달성합니다. 이는 단일 영역의 장애로 인한 서비스 중단을 방지합니다.

  Auto Scaling 그룹: 자동 확장 기능을 통해 트래픽 변동에 따라 동적으로 리소스를 조정할 수 있어, 효율적인 자원 관리와 성능 최적화를 제공합니다.

  Application Load Balancer (ALB): 여러 인스턴스로 트래픽을 분산시켜 부하를 균형 있게 분배하고, 고가용성과 확장성을 보장합니다.

  AWS WAF 연결: WAF를 ALB에 직접 연결함으로써, 애플리케이션 레벨에서의 악성 요청 필터링과 보안 강화를 실현합니다. 이렇게 하면 보안과 가용성이 동시에 향상됩니다.

  옵션 2와 3은 클러스터 배치 그룹이나 고정된 수의 인스턴스를 사용하여 유연성과 확장성 측면에서 한계가 있으며, 옵션 4는 WAF를 Auto Scaling 그룹이 아닌 ALB에 연결해야 하므로 잘못 구성되었습니다. 따라서, 가용성과 보안을 동시에 최적화하는 가장 적합한 해결책은 1번입니다.

903. A company manages a data lake in an Amazon S3 bucket accessed by numerous applications. Each S3 bucket contains a unique prefix for each application. The company wants to restrict each application to a specific prefix and have granular control over the objects under each prefix. What solution can meet these requirements with minimal operational overhead?
① A. Create a dedicated S3 access point and access point policy for each application.
② B. Create an S3 Batch Operations job to set ACL permissions for each object in the S3 bucket.
③ C. Replicate objects from the S3 bucket to new S3 buckets for each application. Create replication rules for each prefix.
④ D. Replicate objects in the S3 bucket to a new S3 bucket for each application. Create a dedicated S3 access point for each application.

  이 회사의 요구 사항은 액세스 제한과 세부적인 개체 제어입니다. 각 애플리케이션을 특정 접두사로 제한하고, 접두사 아래의 개체를 세부적으로 제어하는 최소한의 운영 오버헤드를 가진 솔루션을 찾기 위해 다음과 같은 방법을 생각할 수 있습니다.

  S3 배치 작업은 대량의 객체에 동일한 작업을 수행하는 좋은 방법입니다. ACL 권한을 설정하여 각 애플리케이션에 대한 접근을 제한할 수 있습니다.ACL은 객체 수준에서 사용자와 그룹에 대한 읽기 및 쓰기 권한을 정의하는 데 사용할 수 있습니다.

  따라서 S3 배치 작업을 사용하여 각 객체에 대한 ACL 권한을 설정하여 각 애플리케이션을 특정 접두사로 제한하고, 접두사 아래의 개체를 세부적으로 제어할 수 있습니다. 다른 방법은 운영 오버헤드가 더 크거나, 불필요한 복제나 추가 설정이 필요하므로, 최소한의 운영 오버헤드로 요구 사항을 충족하는 솔루션은 S3 배치 작업을 사용하는 것입니다.

  주어진 문제에서 회사는 최소한의 운영 오버헤드로 애플리케이션별 데이터 접근을 세분화하고 제어하려 합니다. 각 애플리케이션에 고유 접두사를 사용하는 상황에서 가장 효율적인 방법은 다음과 같습니다:

  정답: B

  B 옵션: S3 배치 작업을 통해 각 객체에 대한 ACL(Access Control List) 권한을 설정하는 방법은, 일괄 처리로 접근 제어를 구현할 수 있어 운영 오버헤드를 최소화합니다. 각 애플리케이션의 접두사 아래에 있는 객체에 대해 세밀한 권한 설정이 가능하며, 기존 버킷 구조를 변경하거나 복잡한 복제 과정 없이도 요구 사항을 충족시킬 수 있습니다.
  다른 옵션 분석:

  A 옵션: 액세스 포인트와 정책을 각 애플리케이션별로 생성하면 관리 복잡성이 증가하고, 운영 오버헤드가 늘어납니다.
  C 옵션: 객체를 애플리케이션별로 다른 버킷에 복제하면 데이터 관리와 복제 유지 관리에 상당한 리소스가 필요합니다.
  D 옵션: 이 옵션도 C와 유사하게 복제 과정이 필요하여 운영 비용과 복잡성이 증가합니다. 또한, 액세스 포인트 생성만으로는 접두사 기반의 세밀한 제어가 제한적일 수 있습니다.
  따라서, 최소한의 관리 노력으로 애플리케이션별 데이터 접근을 효과적으로 제어하려면 S3 배치 작업을 활용한 ACL 설정이 가장 적합합니다.

904. A company has an application that allows customers to upload images to an Amazon S3 bucket. Every night, the company launches an Amazon EC2 Spot Fleet to process all the images it receives that day. Each image takes two minutes to process and requires 512 MB of memory.
The solution architect needs to modify the application to process images as they are uploaded.

What change would most cost-effectively meet this requirement?
① A. Use S3 event notifications to write a message containing image details to an Amazon Simple Queue Service (Amazon SQS) queue. Configure an AWS Lambda function to read the message from the queue and process the image.
② B. Write a message containing image details to an Amazon Simple Queue Service (Amazon SQS) queue using S3 event notifications. Configure an EC2 Reserved Instance to read the message from the queue and process the image.
③ C. Use S3 event notifications to post a message containing image details to an Amazon SNS (Amazon SNS) topic. Subscribe to the topic and configure a container instance in Amazon Elastic Container Service (Amazon ECS) to process the image.
④ D. Use S3 event notifications to post a message containing image details to an Amazon SNS topic. Configure your AWS Elastic Beanstalk application to subscribe to the topic and process the image.

  정답은 1번입니다.

  S3 이벤트 알림을 사용하여 Amazon SQS 대기열에 이미지 세부 정보가 포함된 메시지를 쓰고, AWS Lambda 함수를 사용하여 이미지를 처리하는 것이 가장 비용 효율적인 방법입니다.

  이러한 이유로 인해 1번이 정답입니다:

  AWS Lambda 함수는 서버리스 컴퓨팅 서비스로, 이미지 처리 시에만 비용이 발생합니다.
  EC2 예약 인스턴스, Amazon ECS, Elastic Beanstalk은 서버가 항상 실행 중인 경우 비용이 발생합니다.
  따라서, 이미지를 처리하는 데 사용되는 컴퓨팅 리소스를 최적화하여 비용을 절감할 수 있습니다.
  또한, SQS와 Lambda를 사용하면 이미지 처리가 자동으로 수행되므로, 수동으로 EC2 스팟 인스턴스를 시작하고 종료할 필요가 없습니다.

  정답은 1번입니다. 이유는 다음과 같습니다:

  비용 효율성: AWS Lambda는 서버리스 아키텍처를 제공하므로, 실제 실행 시간에만 비용이 발생합니다. 이미지 처리가 필요할 때만 자동으로 실행되어, 기존의 EC2 스팟 인스턴스나 예약 인스턴스처럼 지속적인 비용이 발생하지 않습니다. 이는 특히 이미지 처리가 간헐적으로 발생하는 경우에 매우 경제적입니다.

  자동 확장: Lambda는 요청량에 따라 자동으로 확장됩니다. 즉, 이미지 업로드가 급증하더라도 추가 설정 없이도 처리 능력을 동적으로 늘릴 수 있습니다. 이는 EC2 기반 솔루션(옵션 B, C, D)에 비해 관리 부담을 줄이고 효율성을 높입니다.

  간결성과 유지보수: SQS와 Lambda의 조합은 간단하고 유지보수가 용이합니다. 메시지 큐(SQS)를 통해 안정적인 메시지 전달이 보장되며, Lambda 함수는 단일 목적(이미지 처리)에 최적화되어 있어 복잡성을 최소화합니다.

  옵션 B(EC2 예약 인스턴스)는 예측 가능한 워크로드에 더 적합하고, C와 D(ECS, Elastic Beanstalk)는 컨테이너 관리와 추가적인 오버헤드를 필요로 하므로, 요구 사항에 비해 비용과 복잡성이 높아집니다. 따라서 1번이 가장 비용 효율적이고 적합한 솔루션입니다.

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