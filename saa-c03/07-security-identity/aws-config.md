# AWS Config

- Tracks and evaluates resource configuration over time, checks compliance against rules
- Answers questions like "is any SG open to unrestricted SSH?" or "do my buckets allow public access?"; alerts via SNS; per-region service, but aggregable across regions/accounts; can export config data to S3 for Athena analysis
- **Config Rules** — 75+ AWS-managed rules, or custom rules defined in Lambda; can trigger on every config change and/or on a schedule; rules only **flag** non-compliance, they never block/deny an action
- **Remediation** — auto-remediate non-compliant resources via SSM Automation Documents (managed or custom, can invoke Lambda), with configurable remediation retries
- **Notifications** — EventBridge for non-compliance triggers, or SNS for all config/compliance state changes
- Pricing: no free tier — $0.003 per configuration item recorded/region + $0.001 per rule evaluation/region

> Exam-wording cue: see [monitoring-audit.md](monitoring-audit.md) for the full CloudWatch vs. CloudTrail vs. AWS Config comparison — in short, Config answers "what changed about this resource's configuration, and is it compliant, over time."

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 571-720.
