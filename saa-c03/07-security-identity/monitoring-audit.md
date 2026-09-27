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

> Exam-wording cue: all three sound like generic "security scanners" but ask different questions. **GuardDuty** — "is something suspicious happening?" (threat/anomaly detection across account activity, network traffic, DNS). **Inspector** — "is this resource vulnerable?" (CVE/vulnerability scanning of EC2, ECR images, Lambda). **Macie** — "is sensitive data exposed?" (PII/sensitive-data discovery, S3 only). If the question mentions PII or sensitive data classification → Macie; CVEs/vulnerabilities on compute → Inspector; anomalous account/network behavior → GuardDuty.

> Exam-wording cue: "**identify sensitive data** stored on S3" **and** "**monitor/protect** all S3 data **against malicious activity**" → this needs **both** services together, not either alone: **Amazon Macie** (scans S3 object content via ML/pattern matching to discover and classify sensitive data like PII) **+ Amazon GuardDuty** (with S3 Data Events as an input source, detecting anomalous/malicious activity against the bucket). Macie answers "is sensitive data exposed?"; GuardDuty answers "is something suspicious happening?" — a question naming both requirements in the same sentence is testing whether you reach for both services rather than picking just one.

## AWS Security Hub

- Aggregates and prioritizes **findings** (not raw logs) from GuardDuty, Inspector, Macie, IAM Access Analyzer, Firewall Manager, and third-party tools into one consolidated dashboard with a security score
- Natively supports **multi-account aggregation** across an AWS Organization, via a delegated administrator account — no custom code needed
- Findings can route to EventBridge for automated response workflows
- Runs automated compliance checks against standards (e.g. CIS AWS Foundations Benchmark, PCI DSS)

## Amazon Security Lake

- Automatically collects, normalizes (into **OCSF** — Open Cybersecurity Schema Framework), and centralizes **raw security logs/events** — CloudTrail, VPC Flow Logs, Route 53 Resolver logs, GuardDuty findings, etc. — from across AWS accounts/Organizations and third-party sources
- Stores everything in an **S3-based data lake you own**, in your own account
- **Purely a centralization/normalization/storage layer** — it does not evaluate security posture, score anything, or detect threats on its own; you (or a subscriber service, SIEM, or Athena queries) must separately analyze the data to get insights out of it

> Exam-wording cue: "centralize raw security **logs/events** for later analysis/SIEM ingestion" → **Amazon Security Lake**. "Aggregate/evaluate already-detected **findings** into a posture score/dashboard, least development effort" → **AWS Security Hub** — the word "**evaluate security posture**" specifically points to Security Hub, since Security Lake has no built-in evaluation capability of its own; it just gives you unified raw data that still needs separate querying/tooling to become insights.

## CloudWatch vs. CloudTrail vs. Config

- **CloudWatch** — performance monitoring (metrics/dashboards), events/alerting, log aggregation — see [cloudwatch.md](../08-management-governance/cloudwatch.md)
- **CloudTrail** — who called which API, when (global service, trails can scope to specific resources) — see [cloudtrail.md](cloudtrail.md)
- **Config** — what changed about a resource's configuration, and is it compliant, over time — see [aws-config.md](aws-config.md)
- Example (an ELB): CloudWatch watches connection counts/error rates; Config tracks SG rule changes and enforces "must always have a cert attached"; CloudTrail shows who actually changed the load balancer

## Notes

<!-- Your own notes go here. -->
