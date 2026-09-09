# Observability

- **AWS X-Ray** — distributed tracing for debugging and analyzing performance of serverless/microservice applications
- **Amazon CloudWatch** — metrics, logs, alarms (referenced throughout other services, e.g. GuardDuty + SNS alerting)

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> CloudWatch framed as the foundational monitoring service: visibility into what's happening across the platform, feeding decisions about resource rightsizing, scaling, error detection, and security posture. Provides logs, metrics, alarms, log centralization from multiple sources, and the ability to automate a response, not just passively collect data.
>
> — *Netec S3, 2:40:33-2:42:19, 2:42:51-2:43:38*

> Metrics explained mechanically: a metric (e.g. CPU usage) lives under a namespace representing a service (e.g. EC2), and a dimension narrows it to a specific resource (e.g. instance ID) — data points over time form the metric's history, usable as the basis for autoscaling decisions.
>
> — *Netec S3, 2:43:45-2:46:14*

> CloudWatch Logs explained mechanically: logs from multiple sources (Lambda, applications, on-prem servers via Windows Event Logs or Linux syslog) are grouped into a Log Group, which defines shared config like retention period.
>
> — *Netec S3, 2:46:41-2:51:29*

> CloudTrail described explicitly as a historical/audit-focused service, distinct from CloudWatch's performance focus: records every API call against your account (console, CLI, SDK) — who did what, when, from where, and the result. Framed around answering "who did X" for security/audit/compliance investigations (e.g. who stopped an instance). Event categories: management/administrative events (default-on) vs. data events (e.g. reading an S3 object, invoking Lambda — not enabled by default due to volume). Trail config can centralize logs from multiple accounts into one S3 bucket, choose event types, apply encryption, route to CloudWatch Logs.
>
> — *Netec S3, 2:51:38-2:54:21, 2:52:44-2:53:05, 2:57:42-2:57:59, 2:54:48-2:55:53, 2:56:02-2:57:03*

> Direct comparison: CloudWatch = performance/monitoring/CPU-type metrics; CloudTrail = account activity/audit trail of API calls (management and data-plane both).
>
> — *Netec S3, 2:58:06-2:58:37*

> VPC Flow Logs: capture network traffic metadata (source/destination IP, ports, protocol, byte counts, accept/reject) at the VPC, subnet, or ENI level — use cases: traffic analysis, detecting suspicious activity, diagnosing connectivity problems (e.g. a blocked administrative port). Destinations: CloudWatch Logs, S3.
>
> — *Netec S3, 2:59:20-3:03:33*

> CloudWatch Alarms turn passive monitoring into an automated response: define a metric, a statistic (average/max), an evaluation period, and a threshold — three states: OK, ALARM, INSUFFICIENT_DATA. Worked example: CPU usage metric, instance-ID dimension, average over 5-minute periods, 25% threshold — 2 of the evaluated data points exceeding 25% triggers ALARM. Alarm actions: notify an administrator (e.g. via SNS) or trigger auto scaling.
>
> — *Netec S3, 3:04:47-3:07:42, 3:07:53-3:11:11, 3:11:41-3:12:01*

> Amazon EventBridge introduced as the service for reacting to events and automating responses across integrated AWS services and external/third-party applications, without needing custom integration code for native services — including forwarding AWS events to an external security event-management system. Components: event bus (default + custom buses), rules (filter events by type/origin), targets (SNS, Lambda, another AWS service). Worked example: creating an EC2 instance is itself an event EventBridge can detect and react to (trigger Lambda, send an alert, kick off a workflow) — framed as useful for automating security corrections. Also supports custom events from your own applications/SaaS integrations.
>
> — *Netec S3, 3:12:30-3:13:28, 3:13:35-3:14:23, 3:14:28-3:15:30, 3:15:36-3:16:15, 3:16:25-3:16:50, 3:17:05-3:17:22, 3:19:06-3:19:28*

### CloudWatch deep dive (from slides, pages 571-720)

- Metrics: a variable (e.g. `CPUUtilization`) belongs to a namespace, has dimensions (up to 30, e.g. instance ID) and timestamps; build dashboards, or push Custom Metrics (e.g. RAM, which isn't collected by default)
- Metric Streams: continuously stream metrics near-real-time to Kinesis Data Firehose or 3rd parties (Datadog, Dynatrace, New Relic, Splunk, Sumo Logic), optionally filtered to a subset
- Logs: Log Groups (usually per-app) contain Log Streams (per instance/container/file); configurable expiration (never to 10 years); encrypted by default (or with your own KMS key); can export to S3, Kinesis Data Streams/Firehose, Lambda, OpenSearch
  - Sources: SDK, CloudWatch Logs Agent / Unified Agent, Elastic Beanstalk, ECS, Lambda, VPC Flow Logs, API Gateway, CloudTrail (filtered), Route 53 DNS query logs
  - **Logs Insights** — purpose-built query language to search/filter/aggregate/sort log events, auto-discovers fields, can query multiple log groups across accounts, savable to dashboards; it's a query engine, not real-time (queries run up to 15 min)
  - **S3 Export** (`CreateExportTask`) can take up to 12h to become available — not real-time; use **Logs Subscriptions** (to Kinesis Data Streams/Firehose/Lambda, with a filter) for real-time processing instead, including cross-account delivery
  - EC2 needs the CloudWatch Agent installed to ship logs (no logs flow by default); the old **Logs Agent** only sends logs, the **Unified Agent** also collects system metrics (CPU, disk, RAM, netstat, processes, swap) and supports centralized config via SSM Parameter Store
- Alarms: states OK / INSUFFICIENT_DATA / ALARM; evaluation period as low as 10s for high-resolution custom metrics; can target EC2 actions (stop/terminate/reboot/recover — recovery preserves private/public/elastic IP, metadata, placement group), trigger Auto Scaling, or notify SNS; **Composite Alarms** combine multiple alarms with AND/OR to cut alarm noise; alarms can also be driven by Logs Metric Filters
- **CloudWatch Container Insights** — metrics/logs for ECS, EKS, Kubernetes-on-EC2, and Fargate (needs an agent for Kubernetes)
- **CloudWatch Lambda Insights** — a Lambda Layer that collects system-level metrics (CPU/memory/disk/network) plus diagnostics like cold starts and worker shutdowns
- **CloudWatch Contributor Insights** — finds "Top-N" contributors from any AWS-generated logs (e.g. worst-behaving hosts/IPs/URLs) via custom or AWS-sample rules
- **CloudWatch Application Insights** — automated dashboards/troubleshooting for apps on EC2 (Java/.NET/IIS/DBs) plus related resources (EBS, RDS, ELB, ASG, Lambda, SQS, DynamoDB, S3, ECS, EKS, SNS, API Gateway); powered by SageMaker; sends findings to EventBridge and SSM OpsCenter
- **CloudWatch Network Synthetic Monitor** — agentless ICMP/TCP monitoring of on-prem↔AWS network health over Direct Connect/VPN

### Amazon EventBridge (from slides, pages 571-720)

- Formerly CloudWatch Events; supports Schedule rules (cron-like, e.g. trigger Lambda hourly) and Event Pattern rules (react to a service event, e.g. an EC2 state change, and route to Lambda/SQS/SNS/etc.)
- Broad target list: Lambda, ECS Task, SQS, Step Functions, SSM, SNS, CodePipeline, CodeBuild, EC2 actions, Kinesis Data Streams, AWS Batch, and 3rd parties (Zendesk, Datadog)
- Event buses can be shared cross-account via resource-based policies (e.g. `events:PutEvents`), and events can be archived (indefinitely or for a set period) with the ability to replay them later
- **Schema Registry** — EventBridge can infer/version the schema of events on a bus and generate code bindings for your application
- Security: a rule invoking a target needs permission on it — via a resource-based policy (Lambda, SNS, SQS, S3, API Gateway) or an IAM role (EC2 Auto Scaling, SSM Run Command, ECS task)
