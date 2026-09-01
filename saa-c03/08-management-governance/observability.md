# Observability

- **AWS X-Ray** — distributed tracing for debugging and analyzing performance of serverless/microservice applications
- **Amazon CloudWatch** — metrics, logs, alarms (referenced throughout other services, e.g. GuardDuty + SNS alerting)

## Notes

<!-- Your own notes go here. -->

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
