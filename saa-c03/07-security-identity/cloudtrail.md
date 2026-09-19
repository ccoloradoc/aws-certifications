# AWS CloudTrail

- Audit trail of API calls across your account
- **Data Events** — logs resource-level operations (Data Plane)
- **Management Events** — logs resource management actions (Control Plane)
- Complement with **VPC Flow Logs** for network packet-level visibility
- Enabled by default; records Console/SDK/CLI/AWS-service activity; a trail is All-Regions (default) or single-region; logs can go to CloudWatch Logs and/or S3; if a resource was deleted, check CloudTrail first
- Management Events are logged by default and can be split into Read vs. Write events; Data Events (S3 object-level, Lambda `Invoke`) are **not** logged by default due to volume, and can also be split into Read/Write
- **CloudTrail Insights** — baselines normal management-event activity, then flags anomalies (bad resource provisioning, service-limit hits, IAM action bursts, maintenance gaps) in the console and to S3
- Retention: 90 days in the CloudTrail event history; for longer retention, ship to S3 and query with Athena

> Exam-wording cue: see [monitoring-audit.md](monitoring-audit.md) for the full CloudWatch vs. CloudTrail vs. AWS Config comparison — in short, CloudTrail answers "who called which API, when."

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 571-720.

### From Netec live training (to review)

> CloudTrail described explicitly as a historical/audit-focused service, distinct from CloudWatch's performance focus: records every API call against your account (console, CLI, SDK) — who did what, when, from where, and the result. Framed around answering "who did X" for security/audit/compliance investigations (e.g. who stopped an instance). Event categories: management/administrative events (default-on) vs. data events (e.g. reading an S3 object, invoking Lambda — not enabled by default due to volume). Trail config can centralize logs from multiple accounts into one S3 bucket, choose event types, apply encryption, route to CloudWatch Logs.
>
> — *Netec S3, 2:51:38-2:54:21, 2:52:44-2:53:05, 2:57:42-2:57:59, 2:54:48-2:55:53, 2:56:02-2:57:03*
