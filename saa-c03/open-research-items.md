# Open Research Items

Index of every concept still marked `## To research` across these notes that the slide deck (processed page-by-page into the various `### From slides` sections) never answered. Cross-checked file by file after the full pass — anything not listed here that appears under a "To research" heading has since been answered in that file's "Answers" section.

Use this as your study checklist: pick a row, go to the file, fill in the gap yourself (or from another source), then delete the row.

| Concept | File | Note |
|---|---|---|
| "Security of" vs. "security in" the cloud; responsibility shift across IaaS/PaaS/serverless; concrete patch/physical-security/config examples | [07-security-identity/shared-responsibility-model.md](07-security-identity/shared-responsibility-model.md) | Entire file — zero slide coverage despite being flagged as directly testable |
| AWS CodeCommit, CodeBuild, CodeDeploy, CodePipeline; deployment strategies (in-place, blue/green); tie-in to Elastic Beanstalk | [08-management-governance/cicd-services.md](08-management-governance/cicd-services.md) | Entire file — this course doesn't appear to cover CI/CD services at all |
| AWS Local Zones; AWS Wavelength | [11-other-services/hybrid-edge-infrastructure.md](11-other-services/hybrid-edge-infrastructure.md) | Only Outposts got answered |
| AWS Budgets; AWS Compute Optimizer; Cost Allocation Tags; AWS Pricing Calculator | [12-cost-optimization/cost-management-tools.md](12-cost-optimization/cost-management-tools.md) | Only Cost Explorer, Cost Anomaly Detection, and Trusted Advisor got answered |
| Savings Plans vs. Reserved Instances flexibility trade-off (direct comparison) | [12-cost-optimization/cost-management-tools.md](12-cost-optimization/cost-management-tools.md) | Both are individually covered in [ec2.md](01-compute/ec2.md), but not compared head-to-head |
| Lambda layers | [01-compute/lambda.md](01-compute/lambda.md) | — |
| Event source mappings — poll-based (SQS, Kinesis, DynamoDB Streams) vs. push-based invocation model | [01-compute/lambda.md](01-compute/lambda.md) | Concurrency/pricing/cold-starts/VPC are covered; this mechanism isn't |
| DynamoDB GSI vs. LSI | [04-databases/dynamodb-advanced.md](04-databases/dynamodb-advanced.md) | — |
| DynamoDB Transactions (`TransactWriteItems` / `TransactGetItems`) | [04-databases/dynamodb-advanced.md](04-databases/dynamodb-advanced.md) | — |
| Partner event buses; EventBridge vs. SNS vs. SQS (when to use which) | [06-application-integration/eventbridge.md](06-application-integration/eventbridge.md) | Event buses (default/custom), rules, targets, and Schema Registry are covered |
| ACM private certificates (vs. public) | [07-security-identity/acm.md](07-security-identity/acm.md) | Only the public-certificate flow got covered |
| Envelope encryption (how KMS wraps data keys instead of data) | [07-security-identity/kms-deep-dive.md](07-security-identity/kms-deep-dive.md) | — |
| KMS Grants vs. key policies for temporary/programmatic access | [07-security-identity/kms-deep-dive.md](07-security-identity/kms-deep-dive.md) | — |
| AWS Resource Access Manager (RAM) — how cross-account resource sharing actually works | [09-migration-transfer/migration-services.md](09-migration-transfer/migration-services.md) | Still just the one-line seed bullet; [organizations-control-tower.md](07-security-identity/organizations-control-tower.md) points here for it |
| AWS Snowcone specs/use case | [09-migration-transfer/snow-family.md](09-migration-transfer/snow-family.md) | Snowball Edge (both variants) is covered in detail |
| AWS Snowmobile (exabyte-scale) | [09-migration-transfer/snow-family.md](09-migration-transfer/snow-family.md) | — |
| SSM State Manager (maintaining defined config state on instances) | [08-management-governance/systems-manager.md](08-management-governance/systems-manager.md) | Session Manager, Run Command, Patch Manager, Automation are covered |
| Concrete RTO/RPO numbers AWS DRS targets | [10-resilience-dr/elastic-disaster-recovery.md](10-resilience-dr/elastic-disaster-recovery.md) | Mechanism (continuous block-level replication) is covered, specific target numbers aren't |
| Each Well-Architected pillar's specific design principles; how exam wording maps to a pillar | [well-architected-framework.md](well-architected-framework.md) | Only the cross-pillar general guiding principles + the Well-Architected Tool got covered |
