# Exam Guide Cross-Reference

Maps [`official-exam-guide.md`](official-exam-guide.md) against what's actually documented in `saa-c03/`. Two views over the same underlying check, per [this discussion](#methodology):

1. **[By task](#by-task-primary-study-view)** — the exam's own unit (14 task statements across 4 weighted domains). Use this to decide what to study next, in proportion to how the exam is actually weighted.
2. **[By service](#by-service-lookup-index)** — every in-scope service from the guide, one row each, with a status and a link to where (if anywhere) it's covered. Use this to answer "is X covered, and where."

## Methodology

Status was derived by grepping every file under `saa-c03/` for each in-scope service name (case-insensitive, several name variants tried per service) and judging what came back:

- **✅** — a real section in a topically-appropriate file (not just a passing cross-reference)
- **⚠️** — only a passing/thin mention (e.g. named once in another file's bullet, or only in `misc-managed-services.md`/`README.md`), or a known partial gap already flagged in `open-research-items.md`
- **❌** — no mention found anywhere in the notes

This is a first pass from automated search, not a line-by-line read of every file — treat ⚠️/✅ boundary calls as a starting point to verify, not gospel. Domain/Task assignment in the service table is the guide's own category rounded to the domain(s) that category's task statements actually live in — not a claim that the guide names every service explicitly (most don't; only ~40 of the 117 in-scope services are named as an explicit "e.g." example anywhere in the four domain pages).

**Current tally**: 81 ✅ · 24 ⚠️ · 12 ❌ (of 117 in-scope services checked)

---

## By Task (primary study view)

| Domain | Task | Explicitly-named services/concepts | Coverage | Our files |
|---|---|---|---|---|
| **1** (30%) | 1.1 Secure access to AWS resources | IAM, IAM Identity Center, AWS STS, AWS Control Tower, SCPs | ✅ | `iam-sts-cognito.md`, `account-governance.md`, `aws-organizations-and-control-tower.md` |
| **1** (30%) | 1.2 Secure workloads and applications | Cognito, GuardDuty, Macie, Shield, WAF, IAM Identity Center, Secrets Manager, VPN, Direct Connect | ✅ | `waf-shield-firewall-manager.md`, `monitoring-audit.md`, `secrets-management.md`, `hybrid-connectivity.md` — generic "threat vectors" (DDoS/SQLi) concept itself isn't separately explained anywhere |
| **1** (30%) | 1.3 Data security controls | AWS KMS, ACM | ✅ | `kms-deep-dive.md`, `acm.md`, `aws-backup.md` for backup/replication |
| **2** (26%) | 2.1 Scalable, loosely coupled architectures | API Gateway, AWS Transfer Family, SQS, Secrets Manager, CDN, ALB, Fargate, Lambda, ECS, EKS, Step Functions | ✅ | `api-gateway.md`, `sqs-sns.md`, `containers-paas.md`, `lambda.md`, `orchestration.md` |
| **2** (26%) | 2.2 HA / fault-tolerant architectures | Route 53, Comprehend, Polly, ALB, RDS Proxy, X-Ray | ⚠️ partial | `disaster-recovery.md` (DR strategies ✅), `3-rds-proxy.md` ✅ — **X-Ray is still just a one-liner**, no segments/traces/service-map depth |
| **3** (24%) | 3.1 High-performing/scalable storage | S3, EFS, EBS | ✅ | `s3.md`, `efs.md`, `ebs-instance-store.md`, `comparison.md` |
| **3** (24%) | 3.2 High-performing, elastic compute | AWS Batch, EMR, Fargate, EC2 Auto Scaling, AWS Auto Scaling, Lambda, ECS, EKS | ⚠️ partial | `ec2.md`, `containers-paas.md`, `lambda.md`, `streaming-and-big-data.md` (EMR) — **AWS Batch is only a passing mention, no dedicated coverage** |
| **3** (24%) | 3.3 High-performing database solutions | ElastiCache, Aurora, DynamoDB | ✅ | `4-elasticache.md`, `2-aurora.md`, `dynamodb.md`, `3-rds-proxy.md` |
| **3** (24%) | 3.4 High-performing/scalable network | CloudFront, Global Accelerator, ALB, AWS VPN, Direct Connect, PrivateLink | ⚠️ partial | `cloudfront-and-global-accelerator.md`, `load-balancing.md`, `hybrid-connectivity.md` — **PrivateLink and Client VPN are both thin**, mentioned but never given their own section |
| **3** (24%) | 3.5 High-performing data ingestion/transformation | Athena, Lake Formation, Amazon Quick, DataSync, Storage Gateway, Glue, Kinesis, EMR | ✅ | `streaming-and-big-data.md`, `redshift-athena.md`, `migration-services.md` |
| **4** (20%) | 4.1 Cost-optimized storage | S3 Requester Pays, FSx, EFS, S3, EBS, DataSync, Transfer Family, Storage Gateway, Cost Explorer, Budgets, CUR | ⚠️ partial | Storage services themselves ✅ — **AWS Cost and Usage Report is a zero-coverage gap**, Budgets is thin |
| **4** (20%) | 4.2 Cost-optimized compute | Spot/RI/Savings Plans, Outposts, Lambda, EC2, Fargate, ALB/NLB/GWLB, Cost Explorer, Budgets, CUR | ⚠️ partial | `ec2.md`, `cost-management-tools.md`, `load-balancing.md` — **Savings Plans vs. Reserved Instances isn't compared head-to-head** (both covered individually) |
| **4** (20%) | 4.3 Cost-optimized database solutions | MySQL vs. PostgreSQL (RDS engines), DynamoDB, RDS, Aurora, Cost Explorer, Budgets, CUR | ✅ | `1-rds.md`, `2-aurora.md`, `dynamodb.md`, `choosing-a-database.md` |
| **4** (20%) | 4.4 Cost-optimized network architectures | ALB, Transit Gateway, VPC peering, Global Accelerator, VPC endpoints, Direct Connect, VPN, Cost Explorer, Budgets, CUR | ✅ | `hybrid-connectivity.md`, `vpc.md`, `cloudfront-and-global-accelerator.md` |

**Rollup**: 9 of 14 tasks look solidly ✅-covered; 5 are ⚠️ partial. The recurring gaps: **AWS Cost and Usage Report** (zero coverage, hits both 4.1 and 4.2), **AWS X-Ray** (one-liner, hits 2.2), **AWS Batch** (thin, hits 3.2), **PrivateLink/Client VPN** (thin, hits 3.4), and the **Savings Plans vs. RI** head-to-head comparison (hits 4.2). These line up with what `open-research-items.md` already flags — this table just ties them back to which *domain/task* each gap actually costs you points on.

---

## By Service (lookup index)

### Analytics — Domain 3 (Task 3.5), touches Domain 4 (Tasks 4.1/4.2 cost angle)

| Service | Status | Our file(s) |
|---|---|---|
| Amazon Athena | ✅ | `05-analytics/redshift-athena.md` |
| AWS Data Exchange | ❌ | — (no `05-analytics` file; matches existing gap note) |
| Amazon Data Firehose | ✅ | `05-analytics/streaming-and-big-data.md` |
| Amazon EMR | ✅ | `05-analytics/streaming-and-big-data.md` |
| AWS Glue | ✅ | `05-analytics/streaming-and-big-data.md` |
| Amazon Kinesis | ✅ | `05-analytics/streaming-and-big-data.md` |
| AWS Lake Formation | ✅ | `05-analytics/streaming-and-big-data.md` |
| Amazon MSK | ✅ | `05-analytics/streaming-and-big-data.md` |
| Amazon OpenSearch Service | ✅ | `05-analytics/streaming-and-big-data.md` |
| Amazon Quick (QuickSight) | ✅ | `05-analytics/streaming-and-big-data.md` |
| Amazon Redshift | ✅ | `05-analytics/redshift-athena.md` |

### Application Integration — Domain 2 (Task 2.1)

| Service | Status | Our file(s) |
|---|---|---|
| Amazon AppFlow | ⚠️ | `11-other-services/misc-managed-services.md` (one-liner) |
| Amazon EventBridge | ✅ | `06-application-integration/eventbridge.md` |
| Amazon MQ | ✅ | `06-application-integration/sqs-sns.md` |
| Amazon SNS | ✅ | `06-application-integration/sqs-sns.md` |
| Amazon SQS | ✅ | `06-application-integration/sqs-sns.md` |
| AWS Step Functions | ✅ | `06-application-integration/orchestration.md` |

### AWS Cost Management — Domain 4 (all tasks)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Budgets | ⚠️ | `12-cost-optimization/cost-management-tools.md` (thin — flagged in `open-research-items.md`) |
| AWS Cost and Usage Report | ❌ | — (known gap) |
| AWS Cost Explorer | ✅ | `12-cost-optimization/cost-management-tools.md` |
| Savings Plans | ✅ | `01-compute/ec2.md`, `12-cost-optimization/cost-management-tools.md` — not compared head-to-head vs. RIs |

### Compute — Domain 2 (2.1), Domain 3 (3.2), Domain 4 (4.2)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Batch | ⚠️ | `06-application-integration/eventbridge.md`, `11-other-services/misc-managed-services.md` (passing only) |
| Amazon EC2 | ✅ | `01-compute/ec2.md` |
| EC2 Auto Scaling | ✅ | `01-compute/ec2.md` |
| AWS Elastic Beanstalk | ✅ | `01-compute/containers-paas.md` |
| AWS Outposts | ✅ | `11-other-services/hybrid-edge-infrastructure.md` |
| AWS Serverless Application Repository | ❌ | — (known gap) |
| VMware Cloud on AWS | ⚠️ | `03-storage/fsx.md`, `09-migration-transfer/migration-services.md` (passing only) |
| AWS Wavelength | ⚠️ | `11-other-services/hybrid-edge-infrastructure.md` (thin vs. Outposts in the same file) |

### Containers — Domain 2 (2.1), Domain 3 (3.2)

| Service | Status | Our file(s) |
|---|---|---|
| Amazon ECR | ✅ | `01-compute/containers-paas.md` |
| Amazon ECS | ✅ | `01-compute/containers-paas.md` |
| Amazon ECS Anywhere | ⚠️ | `01-compute/containers-paas.md` (single Netec-quote mention) |
| Amazon EKS | ✅ | `01-compute/containers-paas.md` |
| Amazon EKS Anywhere | ⚠️ | `01-compute/containers-paas.md` (single Netec-quote mention) |
| Amazon EKS Distro | ❌ | — |

### Database — Domain 3 (3.3), Domain 4 (4.3)

| Service | Status | Our file(s) |
|---|---|---|
| Amazon Aurora | ✅ | `04-databases/2-aurora.md` |
| Amazon Aurora Serverless | ✅ | `04-databases/2-aurora.md` |
| Amazon DocumentDB | ⚠️ | `04-databases/choosing-a-database.md` (brief, in "Specialized Databases") |
| Amazon DynamoDB | ✅ | `04-databases/dynamodb.md` |
| Amazon ElastiCache | ✅ | `04-databases/4-elasticache.md` |
| Amazon Keyspaces | ⚠️ | `04-databases/choosing-a-database.md` (brief) |
| Amazon Neptune | ⚠️ | `04-databases/choosing-a-database.md` (brief) |
| Amazon RDS | ✅ | `04-databases/1-rds.md` |

### Developer Tools — Domain 2 (2.2)

| Service | Status | Our file(s) |
|---|---|---|
| AWS X-Ray | ⚠️ | `08-management-governance/x-ray.md` (one-liner only, no depth) |

### Front-End Web and Mobile — Domain 2 (2.1)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Amplify | ⚠️ | `11-other-services/misc-managed-services.md` (one-liner) |
| Amazon API Gateway | ✅ | `06-application-integration/api-gateway.md` |
| AWS Device Farm | ❌ | — (known gap) |

### Machine Learning — Domain 2 (2.2, as example "AWS Managed Services")

| Service | Status | Our file(s) |
|---|---|---|
| Amazon Comprehend | ✅ | `13-machine-learning/ai-services.md` |
| Amazon Lex | ✅ | `13-machine-learning/ai-services.md` |
| Amazon Polly | ✅ | `13-machine-learning/ai-services.md` |
| Amazon Rekognition | ✅ | `13-machine-learning/ai-services.md` |
| Amazon SageMaker AI | ✅ | `13-machine-learning/ai-services.md` |
| Amazon Textract | ✅ | `13-machine-learning/ai-services.md` |
| Amazon Transcribe | ✅ | `13-machine-learning/ai-services.md` |
| Amazon Translate | ✅ | `13-machine-learning/ai-services.md` |

> Note: `ai-services.md` also documents **Amazon Personalize**, which is explicitly **out-of-scope** per the current guide, and **Amazon Kendra**, which appears in neither the in-scope nor out-of-scope ML lists. See `official-exam-guide.md`'s Observations section.

### Management and Governance — Domain 1–4 (cross-cutting; heaviest in 2.2 and general knowledge)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Auto Scaling (unified) | ⚠️ | mentioned in passing (`containers-paas.md`, `lambda.md`, `dynamodb.md`) but no dedicated cross-service explanation |
| AWS CLI | ⚠️ | `11-other-services/misc-managed-services.md` (one-liner) |
| AWS CloudFormation | ✅ | `08-management-governance/cloudformation.md` |
| AWS CloudTrail | ✅ | `07-security-identity/cloudtrail.md` |
| Amazon CloudWatch | ✅ | `08-management-governance/cloudwatch.md` |
| AWS Compute Optimizer | ⚠️ | `01-compute/ec2.md`, `12-cost-optimization/cost-management-tools.md` (thin — flagged gap) |
| AWS Config | ✅ | `07-security-identity/aws-config.md` |
| AWS Control Tower | ✅ | `07-security-identity/aws-organizations-and-control-tower.md` |
| AWS Health Dashboard | ❌ | — (known gap) |
| AWS License Manager | ❌ | — (known gap) |
| Amazon Managed Grafana | ❌ | — (known gap) |
| Amazon Managed Service for Prometheus | ❌ | — (known gap) |
| AWS Management Console | ⚠️ | passing mention only (`iam-sts-cognito.md`) |
| AWS Organizations | ✅ | `07-security-identity/aws-organizations-and-control-tower.md`, `account-governance.md` |
| AWS Service Catalog | ❌ | — (known gap) |
| AWS Systems Manager | ✅ | `08-management-governance/systems-manager.md` |
| AWS Trusted Advisor | ✅ | `12-cost-optimization/cost-management-tools.md` |
| AWS Well-Architected Tool | ✅ | `well-architected-framework.md` |

### Media Services — not a primary domain focus, occasionally referenced in Domain 3.5

| Service | Status | Our file(s) |
|---|---|---|
| Amazon Elastic Transcoder | ⚠️ | `11-other-services/misc-managed-services.md` (one-liner) |
| Amazon Kinesis Video Streams | ❌ | — (known gap, natural fit alongside Kinesis Data Streams/Firehose) |

### Migration and Transfer — Domain 3 (3.5), Domain 4 (4.1)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Application Migration Service | ✅ | `09-migration-transfer/migration-services.md` |
| AWS DataSync | ✅ | `09-migration-transfer/migration-services.md` |
| AWS DMS | ✅ | `09-migration-transfer/migration-services.md` |
| AWS Snow Family | ✅ | `09-migration-transfer/snow-family.md` |
| AWS Transfer Family | ✅ | `09-migration-transfer/migration-services.md` |

### Networking and Content Delivery — Domain 3 (3.4), Domain 4 (4.4)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Client VPN | ⚠️ | `02-networking/vpc.md` (only in ENI/SG tables — known gap) |
| Amazon CloudFront | ✅ | `02-networking/cloudfront-and-global-accelerator.md` |
| AWS Direct Connect | ✅ | `02-networking/hybrid-connectivity.md` |
| Elastic Load Balancing (ELB) | ✅ | `02-networking/load-balancing.md` |
| AWS Global Accelerator | ✅ | `02-networking/cloudfront-and-global-accelerator.md` |
| AWS PrivateLink | ⚠️ | mentioned across `vpc.md`/`load-balancing.md`, no dedicated section |
| Amazon Route 53 | ✅ | `02-networking/route53.md` |
| AWS Site-to-Site VPN | ✅ | `02-networking/hybrid-connectivity.md` |
| AWS Transit Gateway | ✅ | `02-networking/hybrid-connectivity.md` |
| Amazon VPC | ✅ | `02-networking/vpc.md` |

### Security, Identity, and Compliance — Domain 1 (all tasks)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Artifact | ❌ | — (known gap — compliance report access) |
| AWS Certificate Manager (ACM) | ✅ | `07-security-identity/acm.md` |
| AWS CloudHSM | ⚠️ | `waf-shield-firewall-manager.md`, `s3.md` (passing only) |
| Amazon Cognito | ✅ | `07-security-identity/iam-sts-cognito.md` |
| Amazon Detective | ⚠️ | `07-security-identity/aws-organizations-and-control-tower.md` (single mention, no depth) |
| AWS Directory Service | ✅ | `07-security-identity/account-governance.md` (AWS Directory Service section) |
| AWS Firewall Manager | ✅ | `07-security-identity/waf-shield-firewall-manager.md` |
| Amazon GuardDuty | ✅ | `07-security-identity/monitoring-audit.md` |
| AWS IAM Identity Center | ✅ | `07-security-identity/account-governance.md`, `iam-sts-cognito.md` |
| Amazon Inspector | ✅ | `07-security-identity/monitoring-audit.md` |
| AWS KMS | ✅ | `07-security-identity/kms-deep-dive.md` |
| Amazon Macie | ✅ | `07-security-identity/monitoring-audit.md` |
| AWS Network Firewall | ✅ | `07-security-identity/waf-shield-firewall-manager.md` |
| AWS Resource Access Manager (RAM) | ⚠️ | `07-security-identity/aws-organizations-and-control-tower.md` (one-line seed bullet — known gap) |
| AWS Secrets Manager | ✅ | `07-security-identity/secrets-management.md` |
| AWS Security Hub | ⚠️ | `07-security-identity/monitoring-audit.md` (only as an Inspector integration target) |
| AWS Shield | ✅ | `07-security-identity/waf-shield-firewall-manager.md` |
| AWS WAF | ✅ | `07-security-identity/waf-shield-firewall-manager.md` |
| IAM | ✅ | `07-security-identity/iam-sts-cognito.md` |

### Serverless — Domain 2 (2.1), Domain 3 (3.2)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Fargate | ✅ | `01-compute/containers-paas.md` |
| AWS Lambda | ✅ | `01-compute/lambda.md` |

### Storage — Domain 3 (3.1), Domain 4 (4.1)

| Service | Status | Our file(s) |
|---|---|---|
| AWS Backup | ✅ | `10-resilience-dr/aws-backup.md` |
| Amazon EBS | ✅ | `03-storage/ebs-instance-store.md` |
| Amazon EFS | ✅ | `03-storage/efs.md` |
| Amazon FSx | ✅ | `03-storage/fsx.md` |
| Amazon S3 | ✅ | `03-storage/s3.md` |
| Amazon S3 Glacier | ✅ | `03-storage/glacier.md` |
| AWS Storage Gateway | ✅ | `03-storage/storage-gateway.md` |

---

## Next step

Work the ⚠️/❌ rows, roughly in order of how much task weight they touch: **AWS Cost and Usage Report** and **AWS X-Ray** first (each is the sole gap in an otherwise-covered task), then the Domain 3.2/3.4 thin spots (AWS Batch, PrivateLink, Client VPN), then the standalone zero-coverage services that don't have an obvious home yet (Health Dashboard, License Manager, Managed Grafana/Prometheus, Service Catalog, AWS Artifact, Device Farm, Kinesis Video Streams, Data Exchange, Serverless Application Repository, EKS Distro).
