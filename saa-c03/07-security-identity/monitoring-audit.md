# Monitoring & Audit

## Amazon GuardDuty

- Threat detection service
- Integrates with CloudWatch + SNS for alerting/notifications

### More detail (from slides, pages 571-720)

- ML-based anomaly detection, one-click enable (30-day trial), no agents to install
- Input sources: CloudTrail event/management/S3-data-events logs, VPC Flow Logs, DNS logs; optional: EKS Audit Logs, RDS & Aurora, EBS, Lambda, S3 Data Events
- Has a dedicated finding type for cryptocurrency-mining attacks
- Route findings via EventBridge rules to Lambda or SNS

## AWS CloudTrail

- Audit trail of API calls across your account
- **Data Events** — logs resource-level operations (Data Plane)
- **Management Events** — logs resource management actions (Control Plane)
- Complement with **VPC Flow Logs** for network packet-level visibility

### More detail (from slides, pages 571-720)

- Enabled by default; records Console/SDK/CLI/AWS-service activity; a trail is All-Regions (default) or single-region; logs can go to CloudWatch Logs and/or S3; if a resource was deleted, check CloudTrail first
- Management Events are logged by default and can be split into Read vs. Write events; Data Events (S3 object-level, Lambda `Invoke`) are **not** logged by default due to volume, and can also be split into Read/Write
- **CloudTrail Insights** — baselines normal management-event activity, then flags anomalies (bad resource provisioning, service-limit hits, IAM action bursts, maintenance gaps) in the console and to S3
- Retention: 90 days in the CloudTrail event history; for longer retention, ship to S3 and query with Athena

## AWS Config

- Tracks and evaluates resource configuration over time, checks compliance against rules

### More detail (from slides, pages 571-720)

- Answers questions like "is any SG open to unrestricted SSH?" or "do my buckets allow public access?"; alerts via SNS; per-region service, but aggregable across regions/accounts; can export config data to S3 for Athena analysis
- Config Rules: 75+ AWS-managed rules, or custom rules defined in Lambda; can trigger on every config change and/or on a schedule; rules only **flag** non-compliance, they never block/deny an action
- Remediation: auto-remediate non-compliant resources via SSM Automation Documents (managed or custom, can invoke Lambda), with configurable remediation retries
- Notifications: EventBridge for non-compliance triggers, or SNS for all config/compliance state changes
- Pricing: no free tier — $0.003 per configuration item recorded/region + $0.001 per rule evaluation/region

### CloudWatch vs. CloudTrail vs. Config

- **CloudWatch** — performance monitoring (metrics/dashboards), events/alerting, log aggregation
- **CloudTrail** — who called which API, when (global service, trails can scope to specific resources)
- **Config** — what changed about a resource's configuration, and is it compliant, over time
- Example (an ELB): CloudWatch watches connection counts/error rates; Config tracks SG rule changes and enforces "must always have a cert attached"; CloudTrail shows who actually changed the load balancer

## Amazon Inspector (from slides, pages 571-720)

- Automated security assessments — continuous, only scans when needed
- Covers only 3 targets: EC2 instances (via the SSM agent — checks network reachability + OS vulnerabilities against a CVE database), container images pushed to ECR (scanned on push), and Lambda functions (scans code + dependencies on deploy)
- Produces a risk score per finding for prioritization; integrates with Security Hub and sends findings to EventBridge

## Amazon Macie (from slides, pages 571-720)

- Fully managed data security/privacy service using ML + pattern matching to discover sensitive data (e.g. PII) in S3, and alerts you to it

## Notes

<!-- Your own notes go here. -->
