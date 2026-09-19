# Monitoring & Audit

## Amazon GuardDuty

- Threat detection service
- Integrates with CloudWatch + SNS for alerting/notifications

### More detail (from slides, pages 571-720)

- ML-based anomaly detection, one-click enable (30-day trial), no agents to install
- Input sources: CloudTrail event/management/S3-data-events logs, VPC Flow Logs, DNS logs; optional: EKS Audit Logs, RDS & Aurora, EBS, Lambda, S3 Data Events
- Has a dedicated finding type for cryptocurrency-mining attacks
- Route findings via EventBridge rules to Lambda or SNS

## Amazon Inspector (from slides, pages 571-720)

- Automated security assessments — continuous, only scans when needed
- Covers only 3 targets: EC2 instances (via the SSM agent — checks network reachability + OS vulnerabilities against a CVE database), container images pushed to ECR (scanned on push), and Lambda functions (scans code + dependencies on deploy)
- Produces a risk score per finding for prioritization; integrates with Security Hub and sends findings to EventBridge

## Amazon Macie (from slides, pages 571-720)

- Fully managed data security/privacy service using ML + pattern matching to discover sensitive data (e.g. PII) in S3, and alerts you to it

## CloudWatch vs. CloudTrail vs. Config

- **CloudWatch** — performance monitoring (metrics/dashboards), events/alerting, log aggregation — see [cloudwatch.md](../08-management-governance/cloudwatch.md)
- **CloudTrail** — who called which API, when (global service, trails can scope to specific resources) — see [cloudtrail.md](cloudtrail.md)
- **Config** — what changed about a resource's configuration, and is it compliant, over time — see [aws-config.md](aws-config.md)
- Example (an ELB): CloudWatch watches connection counts/error rates; Config tracks SG rule changes and enforces "must always have a cert attached"; CloudTrail shows who actually changed the load balancer

## Notes

<!-- Your own notes go here. -->
