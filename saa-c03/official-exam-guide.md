# SAA-C03 Official Exam Guide — Index

A structured capture of the [official AWS exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html) for SAA-C03, fetched 2026-09-19. This is step one: get the guide's own structure down verbatim (domains, task statements, knowledge/skills, in-scope/out-of-scope services). Mapping each item to where it's covered in `saa-c03/` is a follow-up pass — see `## Next step` at the bottom.

## Exam basics

- 4 content domains, weighted differently — see below
- 50 scored questions + 15 unscored (not identified on the exam) — multiple choice (1 correct + 3 distractors) or multiple response (2+ correct of 5+ options)
- Scaled score 100–1,000; passing score **720**; compensatory scoring — no need to pass each domain individually, only the overall exam
- Target candidate: ≥1 year hands-on experience designing cloud solutions on AWS

## Content Domains & Task Statements

### Domain 1: Design Secure Architectures (30% of scored content)

#### Task 1.1: Design secure access to AWS resources

Knowledge of:
- Access controls and management across multiple accounts
- AWS federated access and identity services (e.g. IAM, AWS IAM Identity Center)
- AWS global infrastructure (e.g. Availability Zones, AWS Regions)
- AWS security best practices (e.g. the principle of least privilege)
- The AWS shared responsibility model

Skills in:
- Applying AWS security best practices to IAM users and root users (e.g. MFA)
- Designing a flexible authorization model that includes IAM users, groups, roles, and policies
- Designing a role-based access control strategy (e.g. AWS STS, role switching, cross-account access)
- Designing a security strategy for multiple AWS accounts (e.g. AWS Control Tower, SCPs)
- Determining the appropriate use of resource policies for AWS services
- Determining when to federate a directory service with IAM roles

#### Task 1.2: Design secure workloads and applications

Knowledge of:
- Application configuration and credentials security
- AWS service endpoints
- Control ports, protocols, and network traffic on AWS
- Secure application access
- Security services with appropriate use cases (e.g. Amazon Cognito, Amazon GuardDuty, Amazon Macie)
- Threat vectors external to AWS (e.g. DDoS, SQL injection)

Skills in:
- Designing VPC architectures with security components (e.g. security groups, route tables, network ACLs, NAT gateways)
- Determining network segmentation strategies (e.g. public subnets vs. private subnets)
- Integrating AWS services to secure applications (e.g. AWS Shield, AWS WAF, IAM Identity Center, AWS Secrets Manager)
- Securing external network connections to/from the AWS Cloud (e.g. VPN, AWS Direct Connect)

#### Task 1.3: Determine appropriate data security controls

Knowledge of:
- Data access and governance
- Data recovery
- Data retention and classification
- Encryption and appropriate key management

Skills in:
- Aligning AWS technologies to meet compliance requirements
- Encrypting data at rest (e.g. AWS KMS)
- Encrypting data in transit (e.g. ACM using TLS)
- Implementing access policies for encryption keys
- Implementing data backups and replications
- Implementing policies for data access, lifecycle, and protection
- Rotating encryption keys and renewing certificates

### Domain 2: Design Resilient Architectures (26% of scored content)

#### Task 2.1: Design scalable and loosely coupled architectures

Knowledge of:
- API creation and management (e.g. Amazon API Gateway, REST API)
- AWS managed services with appropriate use cases (e.g. AWS Transfer Family, Amazon SQS, AWS Secrets Manager)
- Caching strategies
- Design principles for microservices (e.g. stateless vs. stateful workloads)
- Event-driven architectures
- Horizontal scaling and vertical scaling
- How to appropriately use edge accelerators (e.g. CDN)
- How to migrate applications into containers
- Load balancing concepts (e.g. ALB)
- Multi-tier architectures
- Queuing and messaging concepts (e.g. publish/subscribe)
- Serverless technologies and patterns (e.g. AWS Fargate, AWS Lambda)
- Storage types with associated characteristics (e.g. object, file, block)
- The orchestration of containers (e.g. Amazon ECS, Amazon EKS)
- When to use read replicas
- Workflow orchestration (e.g. AWS Step Functions)

Skills in:
- Designing event-driven, microservice, and/or multi-tier architectures based on requirements
- Determining scaling strategies for components used in an architecture design
- Determining the AWS services required to achieve loose coupling based on requirements
- Determining when to use containers
- Determining when to use serverless technologies and patterns
- Recommending appropriate compute, storage, networking, and database technologies based on requirements
- Using purpose-built AWS services for workloads

#### Task 2.2: Design highly available and/or fault-tolerant architectures

Knowledge of:
- AWS global infrastructure (e.g. Availability Zones, AWS Regions, Amazon Route 53)
- AWS Managed Services with appropriate use cases (e.g. Amazon Comprehend, Amazon Polly)
- Basic networking concepts (e.g. route tables)
- Disaster recovery strategies (e.g. backup and restore, pilot light, warm standby, active-active failover, RPO, RTO)
- Distributed design patterns
- Failover strategies
- Immutable infrastructure
- Load balancing concepts (e.g. ALB)
- Proxy concepts (e.g. Amazon RDS Proxy)
- Service quotas and throttling (e.g. configuring service quotas for a standby environment)
- Storage options and characteristics (e.g. durability, replication)
- Workload visibility (e.g. AWS X-Ray)

Skills in:
- Determining automation strategies to ensure infrastructure integrity
- Determining the AWS services required to provide a highly available and/or fault-tolerant architecture across Regions or AZs
- Identifying metrics based on business requirements to deliver a highly available solution
- Implementing designs to mitigate single points of failure
- Implementing strategies to ensure the durability and availability of data (e.g. backups)
- Selecting an appropriate DR strategy to meet business requirements
- Using AWS services that improve the reliability of legacy applications and applications not built for the cloud
- Using purpose-built AWS services for workloads

### Domain 3: Design High-Performing Architectures (24% of scored content)

#### Task 3.1: Determine high-performing and/or scalable storage solutions

Knowledge of:
- Hybrid storage solutions to meet business requirements
- Storage services with appropriate use cases (e.g. Amazon S3, Amazon EFS, Amazon EBS)
- Storage types with associated characteristics (e.g. object, file, block)

Skills in:
- Determining storage services and configurations that meet performance demands
- Determining storage services that can scale to accommodate future needs

#### Task 3.2: Design high-performing and elastic compute solutions

Knowledge of:
- AWS compute services with appropriate use cases (e.g. AWS Batch, Amazon EMR, AWS Fargate)
- Distributed computing concepts supported by AWS global infrastructure and edge services
- Queuing and messaging concepts (e.g. publish/subscribe)
- Scalability capabilities with appropriate use cases (e.g. EC2 Auto Scaling, AWS Auto Scaling)
- Serverless technologies and patterns (e.g. AWS Lambda, Fargate)
- The orchestration of containers (e.g. Amazon ECS, Amazon EKS)

Skills in:
- Decoupling workloads so that components can scale independently
- Identifying metrics and conditions to perform scaling actions
- Selecting the appropriate compute options and features (e.g. EC2 instance types) to meet business requirements
- Selecting the appropriate resource type and size (e.g. Lambda memory) to meet business requirements

#### Task 3.3: Determine high-performing database solutions

Knowledge of:
- AWS global infrastructure (e.g. Availability Zones, AWS Regions)
- Caching strategies and services (e.g. Amazon ElastiCache)
- Data access patterns (e.g. read-intensive vs. write-intensive)
- Database capacity planning (e.g. capacity units, instance types, Provisioned IOPS)
- Database connections and proxies
- Database engines with appropriate use cases (e.g. heterogeneous migrations, homogeneous migrations)
- Database replication (e.g. read replicas)
- Database types and services (e.g. serverless, relational vs. non-relational, in-memory)

Skills in:
- Configuring read replicas to meet business requirements
- Designing database architectures
- Determining an appropriate database engine (e.g. MySQL vs. PostgreSQL)
- Determining an appropriate database type (e.g. Amazon Aurora, Amazon DynamoDB)
- Integrating caching to meet business requirements

#### Task 3.4: Determine high-performing and/or scalable network architectures

Knowledge of:
- Edge networking services with appropriate use cases (e.g. Amazon CloudFront, AWS Global Accelerator)
- How to design network architecture (e.g. subnet tiers, routing, IP addressing)
- Load balancing concepts (e.g. ALB)
- Network connection options (e.g. AWS VPN, AWS Direct Connect, AWS PrivateLink)

Skills in:
- Creating a network topology for various architectures (e.g. global, hybrid, multi-tier)
- Determining network configurations that can scale to accommodate future needs
- Determining the appropriate placement of resources to meet business requirements
- Selecting the appropriate load balancing strategy

#### Task 3.5: Determine high-performing data ingestion and transformation solutions

Knowledge of:
- Data analytics and visualization services with appropriate use cases (e.g. Amazon Athena, AWS Lake Formation, Amazon Quick)
- Data ingestion patterns (e.g. frequency)
- Data transfer services with appropriate use cases (e.g. AWS DataSync, AWS Storage Gateway)
- Data transformation services with appropriate use cases (e.g. AWS Glue)
- Secure access to ingestion access points
- Sizes and speeds needed to meet business requirements
- Streaming data services with appropriate use cases (e.g. Amazon Kinesis)

Skills in:
- Building and securing data lakes
- Designing data streaming architectures
- Designing data transfer solutions
- Implementing visualization strategies
- Selecting appropriate compute options for data processing (e.g. Amazon EMR)
- Selecting appropriate configurations for ingestion
- Transforming data between formats (e.g. .csv to .parquet)

### Domain 4: Design Cost-Optimized Architectures (20% of scored content)

#### Task 4.1: Design cost-optimized storage solutions

Knowledge of:
- Access options (e.g. an S3 bucket with Requester Pays)
- AWS cost management service features (e.g. cost allocation tags, multi-account billing)
- AWS cost management tools with appropriate use cases (e.g. Cost Explorer, Budgets, Cost and Usage Report)
- AWS storage services with appropriate use cases (e.g. Amazon FSx, Amazon EFS, Amazon S3, Amazon EBS)
- Backup strategies
- Block storage options (e.g. HDD vs. SSD volume types)
- Data lifecycles
- Hybrid storage options (e.g. AWS DataSync, AWS Transfer Family, AWS Storage Gateway)
- Storage access patterns
- Storage tiering (e.g. cold tiering for object storage)
- Storage types with associated characteristics (e.g. object, file, block)

Skills in:
- Designing appropriate storage strategies (e.g. batch uploads to S3 vs. individual uploads)
- Determining the correct storage size for a workload
- Determining the lowest cost method of transferring data for a workload to AWS storage
- Determining when storage auto scaling is required
- Managing S3 object lifecycles
- Selecting the appropriate backup and/or archival solution
- Selecting the appropriate service for data migration to storage services
- Selecting the appropriate storage tier
- Selecting the correct data lifecycle for storage
- Selecting the most cost-effective storage service for a workload

#### Task 4.2: Design cost-optimized compute solutions

Knowledge of:
- AWS cost management service features (e.g. cost allocation tags, multi-account billing)
- AWS cost management tools with appropriate use cases (e.g. Cost Explorer, Budgets, Cost and Usage Report)
- AWS global infrastructure (e.g. Availability Zones, AWS Regions)
- AWS purchasing options (e.g. Spot, Reserved Instances, Savings Plans)
- Distributed compute strategies (e.g. edge processing)
- Hybrid compute options (e.g. AWS Outposts)
- Instance types, families, and sizes (e.g. memory optimized, compute optimized, virtualization)
- Optimization of compute utilization (e.g. containers, serverless, microservices)
- Scaling strategies (e.g. auto scaling, hibernation)

Skills in:
- Determining an appropriate load balancing strategy (ALB vs. NLB vs. Gateway Load Balancer)
- Determining appropriate scaling methods and strategies for elastic workloads (horizontal vs. vertical, EC2 hibernation)
- Determining cost-effective AWS compute services with appropriate use cases (e.g. Lambda, EC2, Fargate)
- Determining the required availability for different classes of workloads (production vs. non-production)
- Selecting the appropriate instance family for a workload
- Selecting the appropriate instance size for a workload

#### Task 4.3: Design cost-optimized database solutions

Knowledge of:
- AWS cost management service features (e.g. cost allocation tags, multi-account billing)
- AWS cost management tools with appropriate use cases (e.g. Cost Explorer, Budgets, Cost and Usage Report)
- Caching strategies
- Data retention policies
- Database capacity planning (e.g. capacity units)
- Database connections and proxies
- Database engines with appropriate use cases (e.g. heterogeneous migrations, homogeneous migrations)
- Database replication (e.g. read replicas)
- Database types and services (e.g. relational vs. non-relational, Amazon Aurora, Amazon DynamoDB)

Skills in:
- Designing appropriate backup and retention policies (e.g. snapshot frequency)
- Determining an appropriate database engine (e.g. MySQL vs. PostgreSQL)
- Determining cost-effective AWS database services with appropriate use cases (e.g. DynamoDB vs. RDS, serverless)
- Determining cost-effective AWS database types (e.g. time series format, columnar format)
- Migrating database schemas and data to different locations and/or different database engines

#### Task 4.4: Design cost-optimized network architectures

Knowledge of:
- AWS cost management service features (e.g. cost allocation tags, multi-account billing)
- AWS cost management tools with appropriate use cases (e.g. Cost Explorer, Budgets, Cost and Usage Report)
- Load balancing concepts (e.g. ALB)
- NAT gateways (e.g. NAT instance vs. NAT gateway costs)
- Network connectivity (e.g. private lines, dedicated lines, VPNs)
- Network routing, topology, and peering (e.g. AWS Transit Gateway, VPC peering)
- Network services with appropriate use cases (e.g. DNS)

Skills in:
- Configuring appropriate NAT gateway types for a network (single shared vs. one per AZ)
- Configuring appropriate network connections (Direct Connect vs. VPN vs. internet)
- Configuring appropriate network routes to minimize network transfer costs (Region-to-Region, AZ-to-AZ, private-to-public, Global Accelerator, VPC endpoints)
- Determining strategic needs for CDNs and edge caching
- Reviewing existing workloads for network optimizations
- Selecting an appropriate throttling strategy
- Selecting the appropriate bandwidth allocation for a network device (single vs. multiple VPNs, Direct Connect speed)

## Technologies and Concepts

Non-exhaustive high-level categories the exam draws on (order isn't weighting):

Compute · Cost management · Database · Disaster recovery · High performance · Management and governance · Microservices and component delivery · Migration and data transfer · Networking, connectivity, and content delivery · Resiliency · Security · Serverless and event-driven design principles · Storage

## In-Scope AWS Services

Non-exhaustive, grouped by the guide's own categories.

**Analytics**: Amazon Athena · AWS Data Exchange · Amazon Data Firehose · Amazon EMR · AWS Glue · Amazon Kinesis · AWS Lake Formation · Amazon MSK · Amazon OpenSearch Service · Amazon Quick · Amazon Redshift

**Application Integration**: Amazon AppFlow · Amazon EventBridge · Amazon MQ · Amazon SNS · Amazon SQS · AWS Step Functions

**AWS Cost Management**: AWS Budgets · AWS Cost and Usage Report · AWS Cost Explorer · Savings Plans

**Compute**: AWS Batch · Amazon EC2 · Amazon EC2 Auto Scaling · AWS Elastic Beanstalk · AWS Outposts · AWS Serverless Application Repository · VMware Cloud on AWS · AWS Wavelength

**Containers**: Amazon ECR · Amazon ECS · Amazon ECS Anywhere · Amazon EKS · Amazon EKS Anywhere · Amazon EKS Distro

**Database**: Amazon Aurora · Amazon Aurora Serverless · Amazon DocumentDB · Amazon DynamoDB · Amazon ElastiCache · Amazon Keyspaces · Amazon Neptune · Amazon RDS · Amazon Redshift

**Developer Tools**: AWS X-Ray

**Front-End Web and Mobile**: AWS Amplify · Amazon API Gateway · AWS Device Farm

**Machine Learning**: Amazon Comprehend · Amazon Lex · Amazon Polly · Amazon Rekognition · Amazon SageMaker AI · Amazon Textract · Amazon Transcribe · Amazon Translate

**Management and Governance**: AWS Auto Scaling · AWS CLI · AWS CloudFormation · AWS CloudTrail · Amazon CloudWatch · AWS Compute Optimizer · AWS Config · AWS Control Tower · AWS Health Dashboard · AWS License Manager · Amazon Managed Grafana · Amazon Managed Service for Prometheus · AWS Management Console · AWS Organizations · AWS Service Catalog · AWS Systems Manager · AWS Trusted Advisor · AWS Well-Architected Tool

**Media Services**: Amazon Elastic Transcoder · Amazon Kinesis Video Streams

**Migration and Transfer**: AWS Application Migration Service · AWS DataSync · AWS DMS · AWS Snow Family · AWS Transfer Family

**Networking and Content Delivery**: AWS Client VPN · Amazon CloudFront · AWS Direct Connect · Elastic Load Balancing (ELB) · AWS Global Accelerator · AWS PrivateLink · Amazon Route 53 · AWS Site-to-Site VPN · AWS Transit Gateway · Amazon VPC

**Security, Identity, and Compliance**: AWS Artifact · ACM · AWS CloudHSM · Amazon Cognito · Amazon Detective · AWS Directory Service · AWS Firewall Manager · Amazon GuardDuty · AWS IAM Identity Center · Amazon Inspector · AWS KMS · Amazon Macie · AWS Network Firewall · AWS RAM · AWS Secrets Manager · AWS Security Hub · AWS Shield · AWS WAF · IAM

**Serverless**: AWS Fargate · AWS Lambda

**Storage**: AWS Backup · Amazon EBS · Amazon EFS · Amazon FSx (all types) · Amazon S3 · Amazon S3 Glacier · AWS Storage Gateway

## Out-of-Scope AWS Services

Non-exhaustive; AWS offerings entirely unrelated to the target job role are excluded from this list entirely (i.e. absence here doesn't mean in-scope).

**Application Integration**: Amazon MWAA

**AR and VR**: Amazon Sumerian

**Blockchain**: Amazon Managed Blockchain

**Compute**: Amazon Lightsail

**Database**: Amazon RDS on VMware

**Developer Tools**: AWS CDK · AWS CloudShell · AWS CodeArtifact · AWS CodeBuild · AWS CodeCommit · AWS CodeDeploy · Amazon Corretto · AWS Fault Injection Simulator (FIS) · AWS Tools and SDKs

**Front-End Web and Mobile**: Amazon Location Service

**Game Tech**: Amazon GameLift

**Internet of Things**: All services

**Machine Learning**: Apache MXNet on AWS · AWS DeepComposer · AWS Deep Learning AMIs · AWS Deep Learning Containers · Amazon DevOps Guru · Amazon Elastic Inference · Amazon HealthLake · AWS Inferentia · **Amazon Personalize** · PyTorch on AWS · TensorFlow on AWS

**Management and Governance**: AWS Console Mobile Application · AWS Distro for OpenTelemetry

**Media Services**: AWS Elemental Appliances and Software · MediaConnect · MediaConvert · MediaLive · MediaPackage · MediaTailor · Amazon IVS

**Migration and Transfer**: Migration Evaluator

**Networking and Content Delivery**: AWS Cloud Map

**Quantum Technologies**: Amazon Braket

**Satellite**: AWS Ground Station

## Observations worth acting on later

- **Amazon Personalize is explicitly out-of-scope** in this guide, but `13-machine-learning/ai-services.md` currently documents it as if in-scope (it's in the slide deck, which predates this guide revision or simply goes beyond the exam). Worth a callout in that file once we do the mapping pass.
- **Amazon Kendra appears in neither the in-scope nor out-of-scope ML lists** — also currently documented in `ai-services.md`. Both lists are explicitly "non-exhaustive," so this isn't proof it's out — just unconfirmed either way.
- **"Amazon Quick"** appears (twice) as the in-scope name for the analytics/BI service — likely AWS's newer short/rebranded name covering what our notes call QuickSight. Needs verification before renaming anything.
- The **out-of-scope Developer Tools list (CodeBuild/CodeCommit/CodeDeploy/CDK/CloudShell/...)** matches what `open-research-items.md` already flagged from the Netec deck: *"this course doesn't appear to cover CI/CD services at all"* — the official guide confirms that's correctly out of scope, not just a deck gap.
- **IoT is entirely out of scope** ("All services") — relevant since IoT Core shows up as a source in the Kinesis pipeline example in `05-analytics/streaming-and-big-data.md`; it's fine as pipeline context but not an exam topic itself.
- Several in-scope services have **no file yet** in `saa-c03/`: AWS Data Exchange, Amazon AppFlow, Amazon MQ, VMware Cloud on AWS, AWS Wavelength, Amazon ECS/EKS Anywhere & EKS Distro, Amazon Keyspaces, Amazon Neptune, AWS Device Farm, AWS Health Dashboard, AWS License Manager, Amazon Managed Grafana/Prometheus, AWS Service Catalog, Amazon Elastic Transcoder, Amazon Kinesis Video Streams, AWS Client VPN, AWS Artifact — several of these already appear as rows in `open-research-items.md`'s exam-guide gap-check section, confirming they were already known gaps.

## Next step

Cross-reference the **In-Scope AWS Services** list above against every file under `saa-c03/`, service by service: confirm coverage exists, flag anything missing, and flag anything documented that's actually out-of-scope per this guide (starting with Personalize above). This will likely fold into or replace parts of `open-research-items.md`'s exam-guide gap-check section rather than duplicate it.
