# Secrets Manager vs. Parameter Store

Classic comparison question not covered in the base cheat sheet.

## To research

- **AWS Secrets Manager** — automatic rotation, native RDS/Aurora integration, pricing per secret
- **Systems Manager Parameter Store** (see [systems-manager.md](../08-management-governance/systems-manager.md)) — free tier (Standard), no automatic rotation out of the box, hierarchical parameter organization
- When the exam expects Secrets Manager vs. Parameter Store as the "correct" answer
- Encryption: both integrate with KMS (see [kms-deep-dive.md](kms-deep-dive.md))

## Answers (from slides, pages 571-720)

- **SSM Parameter Store**: secure hierarchical config/secrets storage, optional KMS encryption, serverless, version-tracked, IAM-secured, EventBridge notifications, CloudFormation integration
  - Standard tier: 10,000 parameters/account/region, 4KB max value, free
  - Advanced tier: 100,000 parameters, 8KB max value, supports Parameter Policies, $0.05/advanced-parameter/month
  - Parameter Policies (advanced only) can set a TTL/expiration on a parameter and fire EventBridge notifications on expiration or on no-change — good for forcing rotation of things like passwords
- **AWS Secrets Manager**: newer, purpose-built for secrets; forces rotation on a schedule and can auto-generate new secret values via a Lambda function on rotation; encrypted with KMS; deep native integration with RDS/Aurora (MySQL, PostgreSQL) — this is the main reason to reach for it over Parameter Store
- **Multi-Region Secrets** — Secrets Manager can replicate a secret to other regions as read replicas kept in sync with the primary, promotable to standalone secrets; use cases: multi-region apps/DBs, DR
- Rule of thumb: if the question mentions RDS/Aurora credential rotation, the answer is Secrets Manager; if it's general config/secrets storage without a rotation requirement, Parameter Store (free) is the more cost-effective correct answer

## Notes

<!-- Your own notes go here. -->
