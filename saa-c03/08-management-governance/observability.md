# Observability

- **AWS X-Ray** — distributed tracing for debugging and analyzing performance of serverless/microservice applications
- See [monitoring-audit.md](../07-security-identity/monitoring-audit.md) for CloudTrail, AWS Config, and the CloudWatch vs. CloudTrail vs. Config comparison — this file covers CloudWatch and EventBridge

## Amazon CloudWatch

- The foundational monitoring service: visibility into what's happening across the platform, feeding decisions about resource rightsizing, scaling, error detection, and security posture — not just passive collection, it can also automate a response
- Provides metrics for every AWS service, log centralization from multiple sources, and alarms

### Metrics

- A **metric** is a variable to monitor (e.g. `CPUUtilization`, `NetworkIn`), belonging to a **namespace** representing a service (e.g. EC2); a **dimension** is an attribute that narrows it to a specific resource (e.g. instance ID) — up to 30 dimensions per metric
- Metrics have timestamps; data points over time form the metric's history, usable as the basis for autoscaling decisions
- Build CloudWatch dashboards from metrics, or push **Custom Metrics** for anything not collected by default (e.g. RAM)
- **Metric Streams** — continually stream metrics to a destination with near-real-time delivery: Kinesis Data Firehose (and onward to its destinations), or a 3rd party (Datadog, Dynatrace, New Relic, Splunk, Sumo Logic); can filter to stream only a subset

### Logs

- **Log Groups** (arbitrary name, usually one per application) contain **Log Streams** (per instance/log file/container); define shared config on the group, including a retention/expiration policy (never expire, 1 day to 10 years)
- Encrypted by default, or with your own KMS key; can export to S3, Kinesis Data Streams/Firehose, Lambda, or OpenSearch
- **Sources**: SDK, CloudWatch Logs Agent / Unified Agent, Elastic Beanstalk, ECS, Lambda, VPC Flow Logs, API Gateway, CloudTrail (filtered), Route 53 DNS query logs
- **Logs Insights** — purpose-built query language to search/filter/aggregate/sort log events; auto-discovers fields from AWS services and JSON log events; can query multiple log groups across accounts; savable to dashboards; it's a query engine, not real-time (queries run up to 15 min)
- **S3 Export** (`CreateExportTask`) can take up to 12h to become available — not real-time; use **Logs Subscriptions** (real-time delivery to Kinesis Data Streams/Firehose/Lambda, with a Subscription Filter) instead for real-time processing
- **Cross-account / multi-region aggregation** — Logs Subscriptions can send log events to resources (KDS, KDF) in a different AWS account, via an IAM role (cross-account) plus a destination access policy — used to centralize logs from many accounts/regions into one place
- EC2 needs the CloudWatch Agent installed to ship logs (no logs flow by default; IAM permissions must be correct; the agent can also run on-premises) — the old **Logs Agent** only sends logs, the **Unified Agent** also collects system-level metrics (CPU, disk, RAM, netstat, processes, swap) and supports centralized config via SSM Parameter Store

### Alarms

- Define a **metric**, a **statistic** (average/max/min/etc.), an **evaluation period** (length of time in seconds to evaluate the metric; as low as 10s/30s/multiples of 60s for high-resolution custom metrics), and a **threshold**
- Three states: **OK**, **ALARM**, **INSUFFICIENT_DATA**. Worked example: CPU usage metric, instance-ID dimension, average over 5-minute periods, 25% threshold — 2 of the evaluated data points exceeding 25% triggers ALARM
- **Alarm targets/actions**: stop/terminate/reboot/recover an EC2 instance, trigger an Auto Scaling action, or notify SNS (from which you can do pretty much anything)
  - EC2 recovery preserves the same private/public/elastic IP, metadata, and placement group; recovery decisions draw on **Status Checks** — Instance status (the VM itself), System status (underlying hardware), and attached EBS status
- **Composite Alarms** — monitor the states of multiple other alarms with AND/OR conditions, to cut down "alarm noise" instead of relying on single-metric alarms alone
- Good to know: alarms can also be driven by **Logs Metric Filters**; test an alarm/notification without waiting for a real breach via `aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purposes"`

### CloudWatch Insights & Specialized Monitoring

- **Container Insights** — collect/aggregate/summarize metrics and logs from containers on ECS, EKS, Kubernetes-on-EC2, and Fargate (both for ECS and EKS); on EKS/Kubernetes it uses a containerized version of the CloudWatch Agent to discover containers
- **Lambda Insights** — monitoring/troubleshooting for serverless apps; collects system-level metrics (CPU, memory, disk, network) plus diagnostics like cold starts and worker shutdowns; delivered as a Lambda Layer
- **Contributor Insights** — analyzes any AWS-generated logs (VPC, DNS, etc.) to surface "Top-N" contributors (e.g. worst-behaving hosts/IPs/URLs) and total unique contributors; build custom rules or use AWS-provided sample/built-in rules
- **Application Insights** — automated dashboards to isolate problems in apps on EC2 (Java/.NET/IIS/databases) plus related resources (EBS, RDS, ELB, ASG, Lambda, SQS, DynamoDB, S3, ECS, EKS, SNS, API Gateway); powered by SageMaker; sends findings/alerts to EventBridge and SSM OpsCenter
- **Network Synthetic Monitor** — agentless monitoring of on-prem↔AWS network health (packet loss, latency, jitter) over Direct Connect/VPN, via ICMP/TCP tests; publishes results to CloudWatch Metrics

## Amazon EventBridge

- (formerly CloudWatch Events) the service for reacting to events and automating responses, across AWS services and 3rd-party/SaaS applications, without custom integration code for native services
- **Schedule rules** — cron-like (e.g. trigger a Lambda function every hour)
- **Event Pattern rules** — react to a service doing something (e.g. an EC2 instance state-change notification), filter the event, then route it to a target
- Broad target list: Lambda, ECS Task, SQS, Step Functions, SSM, SNS, CodePipeline, CodeBuild, EC2 actions, Kinesis Data Streams, AWS Batch, and 3rd parties (Zendesk, Datadog)
- Event buses can be accessed cross-account via **resource-based policies** (`events:PutEvents`) — use case: aggregate all events from your AWS Organization into a single account or region
- Events can be archived (all or filtered, indefinitely or for a set period), with the ability to replay them later
- **Schema Registry** — EventBridge can analyze events on a bus and infer/version their schema, and generate code bindings so your application knows in advance how event data is structured
- **Security** — a rule invoking a target needs permission on it: a resource-based policy (Lambda, SNS, SQS, S3, API Gateway) or an IAM role (EC2 Auto Scaling, SSM Run Command, ECS task)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 571-720.

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
