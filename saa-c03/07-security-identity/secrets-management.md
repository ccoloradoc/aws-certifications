# Secrets Manager vs. Parameter Store

Classic comparison question not covered in the base cheat sheet.

## SSM Parameter Store

Part of Systems Manager — see [systems-manager.md](../08-management-governance/systems-manager.md).

- Secure storage for configuration and secrets
- Optional seamless encryption using KMS (see [kms-deep-dive.md](kms-deep-dive.md))
- Serverless, scalable, durable, easy SDK
- Version tracking of configurations/secrets
- Security through IAM
- Notifications through Amazon EventBridge
- Integration with CloudFormation

### Hierarchy

- Parameters are organized in paths (e.g. `/other-department/...`)
- Secrets Manager secrets are readable through Parameter Store at `/aws/reference/secretsmanager/<secret_ID>`
- Public AWS-provided parameters exist too, e.g. `/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2` for the latest Amazon Linux AMI

### Standard vs. Advanced Tiers

| | Standard | Advanced |
|---|---|---|
| Parameters per account per Region | 10,000 | 100,000 |
| Max value size | 4 KB | 8 KB |
| Parameter policies | No | Yes |
| Cost | No additional charge | Charges apply |
| Storage pricing | Free | $0.05 per advanced parameter per month |

### Parameter Policies (Advanced Only)

- Assign a TTL (expiration date) to a parameter to force updating or deleting sensitive data such as passwords; multiple policies can be assigned at a time
- **Expiration** — deletes the parameter
- **ExpirationNotification** — EventBridge notification before expiry
- **NoChangeNotification** — EventBridge notification if the parameter hasn't changed for a set period

## AWS Secrets Manager

- Newer service, meant specifically for storing secrets
- Forces rotation of secrets every X days
- Automates generation of new secret values on rotation (uses a Lambda function)
- Integrates with Amazon RDS (MySQL, PostgreSQL, Aurora) — "mostly meant for RDS integration"
- Secrets are encrypted using KMS

### Multi-Region Secrets

- Replicate secrets across multiple Regions; Secrets Manager keeps the read replicas in sync with the primary secret
- A read replica can be promoted to a standalone secret
- Use cases: multi-Region apps, disaster recovery strategies, multi-Region databases

## Choosing Between Them

- Rule of thumb: RDS/Aurora credential rotation → Secrets Manager; general config/secrets storage with no rotation requirement → Parameter Store (free) is the more cost-effective answer

> Exam-wording cue: "automatically rotate database credentials" or "rotate secrets every X days" → **Secrets Manager**; "cheapest/free way to store configuration or a secret" or "hierarchical config" → **Parameter Store**; "replicate a secret to another Region for DR" → **Secrets Manager multi-Region secrets**.

## Where These Show Up in Other Services

- **ECS** — the EC2 instance profile (EC2 launch type) lets the ECS agent reference sensitive data in Secrets Manager or SSM Parameter Store
- **RDS Proxy** — enforces IAM authentication for the DB and securely stores credentials in Secrets Manager
- **CloudWatch Unified Agent** — centralized configuration via SSM Parameter Store

## To research

- Secrets Manager pricing per secret (slides don't give numbers)
- Secrets Manager native rotation beyond RDS/Aurora (slides only mention MySQL, PostgreSQL, Aurora)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 661-690 (Parameter Store, Secrets Manager, multi-Region secrets), with cross-service mentions from pages 181-210, 421-450, and 571-600.
