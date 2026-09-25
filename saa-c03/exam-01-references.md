# Exam Cues — Session Extract 1

All `> Exam-wording cue:` blocks added during this session, pulled from files that are modified but not yet committed. Each entry links back to its source file — once committed, treat this as a point-in-time study extract rather than a live-synced index (edits to the source afterward won't be reflected here).

## [01-compute/containers-paas.md](01-compute/containers-paas.md)

> Exam-wording cue: "serverless, no infrastructure management, but the workload runs **longer than 15 minutes**" (a long batch job, a long-running background process/service) → **Fargate**, not Lambda — Lambda's hard 900-second (15-min) execution cap (see [lambda.md](01-compute/lambda.md)) rules it out regardless of how attractive "serverless" sounds in the wording. "Short-lived, event-driven, sub-15-minute task" → either works, but Lambda is usually the simpler/cheaper answer.

> Exam-wording cue: "different pods/services in the same EKS cluster need different, isolated AWS permissions" → **IRSA**. The same requirement on ECS → **ECS Task IAM Role** (see above). Either way, "give the node/instance role broader permissions instead" is the wrong answer — it violates least privilege by granting access to everything running on that node, not just the workload that needs it.

## [01-compute/ec2.md](01-compute/ec2.md)

> Exam-wording cue: "copying an AMI to another region creates an extra/unexpected EBS snapshot there" is expected behavior, not a bug or wasted cost to eliminate — an AMI cannot exist in a region without its backing snapshot in that same region. A question framing this as a problem to solve is testing whether you understand AMIs are snapshot-backed, not that something is misconfigured.

> Exam-wording cue: demand follows a **known, predictable calendar pattern** (e.g. payroll runs every 1st of the month, a marketing event with a known start time) → **Scheduled Scaling** — set capacity ahead of time, no metric needed. Demand is **live/reactive to current load** (CPU spikes unpredictably) → **Dynamic/Target Tracking Scaling**. Demand has a **recurring but not manually-known pattern** AWS can learn from historical data → **Predictive Scaling**. A scenario stating the exact date/time of an expected spike is the giveaway for Scheduled, not Predictive or Dynamic.

> Exam-wording cue: within **Dynamic Scaling** itself, **Target Tracking** is the "just tell me the number" option — you set a target value for a metric (e.g. "keep average CPU at 40%"), and AWS automatically creates and manages the CloudWatch alarms and capacity adjustments to hold it there; this is AWS's recommended default for most threshold-based scaling and the answer whenever the scenario just wants a metric held near a target with minimal setup. **Step Scaling** is for when the *magnitude* of the breach should drive the *size* of the response (e.g. CPU at 90% scales out more instances than CPU at 65%) — you define the alarm and multiple adjustment steps yourself. **Simple Scaling** is the legacy option: one alarm, one fixed adjustment, then it waits out a full cooldown before evaluating again — rarely the correct exam answer once Target Tracking or Step Scaling is offered as an alternative.

> Exam-wording cue: need to work on **one specific instance** without it being terminated or losing overall serving capacity → **Standby state** (don't decrement desired capacity). Need to perform maintenance **across the group** where health checks would otherwise flag instances as unhealthy during the work (e.g. a manual rolling update) → **suspend the `ReplaceUnhealthy` process** for the duration, then resume it. Both avoid termination; Standby is per-instance and explicit, `ReplaceUnhealthy` suspension is group-wide and health-check-driven.

> Exam-wording cue: instance-replacement **order** differs by process. **`AZRebalance`** (fixing an AZ imbalance) always **launches the replacement first, then terminates** the old instance — it can even temporarily exceed max size (by ~10%, rounded up) to do so, since the instances being replaced aren't broken, just unevenly distributed, and AWS avoids a capacity dip. **`ReplaceUnhealthy`** (a single failed instance) does the opposite: **terminates the unhealthy instance first, then launches** a replacement — no allowance to exceed max size, since the bad instance should come out immediately. A question asking "does the ASG launch or terminate first" hinges entirely on *which* process is triggering the replacement.

## [01-compute/lambda.md](01-compute/lambda.md)

> Exam-wording cue: the concurrency limit (1,000 by default) is a **pool shared across every function in the account/region**, not per-function — a burst of invocations from **any** trigger (SNS, S3 events, EventBridge, etc.) competes for that same shared pool, so one noisy function can throttle an unrelated one. SNS (and S3/EventBridge) invoke Lambda **asynchronously**, so throttled invocations there aren't dropped immediately — they auto-retry for up to 6 hours before hitting a DLQ, meaning symptoms show up as **delayed processing**, not outright failure. "Invocations from [any async trigger] are being delayed/dropped under high volume" → fix with **Reserved Concurrency** on the affected function (guarantees it a slice of the pool) or an account concurrency limit increase — not something fixable on the trigger's side.

## [02-networking/cloudfront-and-global-accelerator.md](02-networking/cloudfront-and-global-accelerator.md)

> Exam-wording cue: "CloudFront origin is on-premises / not an AWS resource" → **Custom Origin**, not the S3 or VPC Origin types (those are for AWS-hosted content). "Prevent users from bypassing CloudFront and hitting the origin directly" — for **S3** → **OAC**; for a **custom origin (including on-prem)** → a **custom header with a secret value**, checked at the origin.

## [02-networking/route53.md](02-networking/route53.md)

> Exam-wording cue: routing decided by **user location only** (country/state/continent), e.g. "restrict content to a specific country" or "serve localized content by region" → **Geolocation**. Routing decided by **actual measured network latency**, e.g. "fastest response time for users" → **Latency-based**. Routing that needs to **shift traffic volume toward/away from a resource** via a bias value, or involves non-AWS/on-prem endpoints by lat/long → **Geoproximity** (the only one of the three that can deliberately shrink or expand a resource's effective radius). "Location" in the question wording ≠ automatically Geolocation — check whether it's about compliance/content restriction (Geolocation) vs. speed (Latency) vs. shifting load with a bias (Geoproximity).

> Exam-wording cue: on-prem needs to resolve **AWS-side** names (e.g. a Private Hosted Zone record, an EC2 private DNS name) → **Inbound Endpoint** (traffic flows *into* the VPC's resolver). The VPC needs to resolve **on-prem** names (e.g. a corporate domain hosted on your own DNS server) → **Outbound Endpoint** (traffic flows *out* to your on-prem resolvers), paired with a **Resolver Rule** that forwards queries for that specific domain. Direction naming is from the VPC's perspective: Inbound = queries coming in, Outbound = queries going out.

## [02-networking/vpc.md](02-networking/vpc.md)

> Exam-wording cue: need to expose *one specific service* to many other VPCs/accounts (especially at scale, or where CIDRs might overlap) → **PrivateLink** (endpoint service + Interface Endpoint). Need full bidirectional network connectivity between two VPCs → **VPC Peering** (or **Transit Gateway** at scale, see [hybrid-connectivity.md](02-networking/hybrid-connectivity.md)). "Overlapping CIDRs" in the question is the classic tell that rules out Peering and points to PrivateLink.

## [03-storage/comparison.md](03-storage/comparison.md)

> Exam-wording cue: a scenario comparing storage costs for the *same small file* across S3/EFS/EBS is testing this exact distinction — **S3 and EFS bill only for actual data stored**; **EBS bills for the entire provisioned volume** — a small file on a large EBS volume can cost far more than the same file in S3 or EFS, even at a lower $/GB rate.

## [03-storage/ebs-instance-store.md](03-storage/ebs-instance-store.md)

> Exam-wording cue: a scenario emphasizing **high random I/O performance / very high IOPS** for **temporary, non-critical, or reproducible** data (buffer, cache, scratch space, temporary batch-processing output) → **Instance Store** — physically attached to the host, so it's faster than any network-attached volume, but disappears on stop/terminate. A scenario needing that same performance for data that **must persist and survive an instance stop/terminate/replacement** → **EBS Provisioned IOPS (io1/io2)** instead, which trades a bit of raw speed for durability/persistence. "High I/O" alone in the question isn't enough to pick Instance Store — check whether the data can be lost.

> Exam-wording cue: the two EBS families split by *what they're optimized to measure*. **SSD volumes** (gp2/gp3, io1/io2) are built for **transactional workloads with small, frequent read/write I/O**, where **IOPS** is the metric that matters — this is also why only SSD types are allowed as a **boot volume** (an OS needs fast small random reads at startup). **HDD volumes** (st1, sc1) are built for **large sequential/streaming workloads**, where **throughput (MiB/s)**, not IOPS, is the relevant measure — think big-data/data-warehousing scans, not booting an instance. A question ruling out an option because it "cannot be a boot volume" is almost always pointing at st1 or sc1.

## [03-storage/efs.md](03-storage/efs.md)

> Exam-wording cue: "restrict which instances/subnets can even connect to an EFS file system" → **Security Groups on the mount targets** (network-layer gate — can you reach port 2049 at all). "Restrict what a connected client can read/write once mounted" → **POSIX permissions** (or IAM + Access Points for finer-grained, per-client-path control) — SGs don't control file/directory-level access, only network reachability.

> Exam-wording cue: "share one EFS file system across multiple AWS accounts/VPCs" needs all three — **VPC Peering/Transit Gateway** (network) **+ an EFS File System Policy** (resource-based, on the file system) **+ IAM permissions in the caller's account**. A distractor that only sets up one or two of these (e.g. "just peer the VPCs" or "just add an IAM policy") is incomplete — miss any one piece and the mount fails.

## [03-storage/fsx.md](03-storage/fsx.md)

> Exam-wording cue: Lustre itself has **no cold storage tier** — its own storage (SSD/HDD, Scratch/Persistent) is always active/hot, built for compute. But because a Lustre file system can be **linked to an S3 bucket** as its data repository, data can flow out of Lustre into S3, and once there, normal **S3 Lifecycle rules** can transition it into IA/Glacier — so Lustre *enables a path* to cold storage via S3 rather than providing cold storage itself. "Parallel/distributed high-throughput HPC filesystem" → **FSx for Lustre**. "Cold/archival storage for rarely-accessed data" alone → **S3 Glacier** (or Lifecycle rules), not Lustre. A scenario combining fast HPC compute now with eventual archiving later is valid as **Lustre linked to S3, with S3 Lifecycle handling the cold tiering**.

## [03-storage/s3.md](03-storage/s3.md)

> Exam-wording cue: lifecycle transitions only move **forward** through this order — never backward:
> ```
> Standard
>    ↓
> Standard-IA  /  Intelligent-Tiering  /  One Zone-IA
>    ↓
> Glacier Instant Retrieval
>    ↓
> Glacier Flexible Retrieval
>    ↓
> Glacier Deep Archive
> ```
> Standard can transition directly to any class; Standard-IA/Intelligent-Tiering can only move onward into One Zone-IA or a Glacier tier; One Zone-IA can only move into Glacier; the Glacier tiers only move "colder" among themselves. A question describing a rule that moves an object **back** to a cheaper-but-more-available class (or skips straight from Deep Archive to Standard) is invalid as a lifecycle rule — that always requires a manual **restore**, not a transition.

> Exam-wording cue: Standard-IA vs. Intelligent-Tiering comes down to who picks the tier. Access pattern is **known/predictable** (e.g. "compliance backups accessed rarely") → **Standard-IA** — you choose it upfront, and it charges a per-GB **retrieval fee** every time you access an object, plus a 30-day minimum storage duration. Access pattern is **unknown, unpredictable, or changes over time** → **Intelligent-Tiering** — AWS monitors real access and moves objects between tiers automatically, with **no retrieval fees** at all (only a small per-object monitoring fee), and no minimum storage duration. "Unknown/unpredictable access patterns" in the question is the standard tell for Intelligent-Tiering.

> Exam-wording cue: need to protect an object for a **fixed, known duration** (e.g. "retain for 7 years") → **Retention Period**, in **Compliance mode** if it must be untouchable by *anyone including root/admins* (regulatory wording like "even the account root user cannot delete"), or **Governance mode** if some privileged users should still be able to override it. Need protection with **no defined end date** tied to an ongoing event (e.g. litigation, an investigation) that gets manually lifted later → **Legal Hold**, independent of and stackable with any Retention Period on the same object.

> Exam-wording cue: hitting **request-rate/throttling limits** on a bucket, or needing to **increase transactions per second** → spread objects across **more prefixes** (e.g. `bucket/a/`, `bucket/b/`, `bucket/c/...` instead of everything under one path) — since the ~3,500 write / ~5,500 read req/sec limit applies *per prefix*, not per bucket, more prefixes means more parallel throughput. A single sequential-looking key scheme (e.g. all keys starting with a timestamp) is the classic wrong setup that causes this bottleneck; randomizing/hashing a prefix into the key name is the standard fix.

> Exam-wording cue: S3 checks whether Transfer Acceleration would actually be **faster** than a standard transfer to the bucket's region before using it — if a client is already close to the bucket's Region (so acceleration wouldn't help), S3 **falls back to a normal transfer and doesn't charge the TA premium** for that request. So "does Transfer Acceleration always cost extra?" → **no**, the surcharge only applies when acceleration is actually used; a question framing a nearby-client upload as "extra cost from enabling TA" is testing this fallback behavior. This premium is on top of standard data transfer pricing, not a substitute for it — it only ever adds cost when acceleration actually kicks in, never removes it.

> Exam-wording cue: **data transfer IN** to S3 from the internet is always **free**, regardless of storage class — only the PUT/POST/COPY **request** itself carries a (small) per-request charge. What costs real money is **data transfer OUT** (to the internet, and often cross-region) plus ongoing storage. A question framing an upload-heavy workload as expensive is testing whether you know ingress is free — the cost driver is downloads/egress, not uploads.

## [04-databases/1-rds.md](04-databases/1-rds.md)

> Exam-wording cue: "need privileged OS/database-host access to support a third-party app's special configuration or patching requirements, on Oracle or SQL Server" → **RDS Custom** — standard RDS deliberately blocks this kind of access as part of being fully managed. If the scenario also emphasizes HA, remember Multi-AZ isn't a given here; it must be set up explicitly, unlike the "just toggle it" experience on standard RDS.

> Exam-wording cue: **synchronous** (Multi-AZ) means a write is only acknowledged back to the app once *both* the primary and standby have confirmed it — that round-trip is what makes automatic, zero-data-loss failover possible, at the cost of added write latency; it's why Multi-AZ is framed around **"no data loss"** and **"automatic failover."** **Asynchronous** (Read Replica) means the primary acknowledges the write immediately and pushes it to the replica afterward — faster writes, but the replica can lag ("eventually consistent"), and a failure right after a write but before it replicates means that data is **not** on the replica; that lag is why promoting a Read Replica is always a **manual** action, never automatic. So: "no data loss"/"automatic failover" → Multi-AZ (sync); "scale reads"/"heavy read load"/"improve read performance" → Read Replica (async) — sync vs. async is the mechanical reason those two framings map the way they do.

## [04-databases/3-rds-proxy.md](04-databases/3-rds-proxy.md)

> Exam-wording cue: "Lambda + RDS, connection errors/throttling during traffic **spikes**" → **SQS (decouple + rate-limit via Reserved Concurrency) + RDS Proxy (pool connections)**, both together. If the question describes timeouts/connection exhaustion under **steady** (not bursty) traffic, RDS Proxy alone is usually the intended answer, without needing SQS.

## [04-databases/dynamodb.md](04-databases/dynamodb.md)

> Exam-wording cue: "DynamoDB reads are slow," a **hot partition key**, or "microsecond DynamoDB reads with no application code changes" → **DAX** — it's API-compatible with DynamoDB, so the app keeps calling the same operations while DAX transparently caches in front of them. "Cache session data," cache results from a **different** database/computation, or need Redis-specific features (pub/sub, sorted sets) → **ElastiCache** instead, which requires you to write the cache-aside logic yourself (check cache → miss → query DynamoDB → write back to cache) rather than being transparent like DAX. See [4-elasticache.md](04-databases/4-elasticache.md).

## [05-analytics/streaming-and-big-data.md](05-analytics/streaming-and-big-data.md)

> Exam-wording cue: within Glue's own authoring tools — "visually clean/normalize/explore a dataset, no code, built for analysts" → **DataBrew** (spreadsheet-like UI, data profiling/cleanup, not full pipeline orchestration). "Visually build an ETL pipeline (source → transform → target), still generates Spark code" → **Glue Studio**. "Full programmatic control writing the Spark ETL job yourself" → **Glue ETL** directly. DataBrew is the odd one out: it's for preparing a single dataset, not authoring a multi-step job.

## [06-application-integration/sqs-sns.md](06-application-integration/sqs-sns.md)

> Exam-wording cue: a scenario requiring messages **never be processed more than once** or **strict ordering** (e.g. financial transactions, sequential commands) → **FIFO** — deduplication (via Deduplication ID or content-based dedup) and ordering (via Message Group ID) are FIFO-only guarantees. **Standard** queues give **at-least-once** delivery, meaning the consumer application itself must tolerate/handle duplicate and out-of-order messages (e.g. by making processing idempotent) — Standard never prevents duplicates on its own.

> Exam-wording cue: "reduce SQS costs/API calls" or "increase SQS throughput" → **batch API operations** (SendMessageBatch/DeleteMessageBatch), up to 10 messages per call. "Some messages in a Lambda-SQS batch fail and shouldn't cause the whole batch to be reprocessed" → **`ReportBatchItemFailures`**, not a bigger batch size or a separate queue.

## [07-security-identity/account-governance.md](07-security-identity/account-governance.md)

> Exam-wording cue: "centralize access for AWS Organizations accounts using our **existing on-prem Active Directory**, with minimal new infrastructure to manage, group-based/role-based access, and single sign-on" → **AD Connector + IAM Identity Center + Permission Sets**. The pattern: AD Connector proxies to on-prem AD (no directory data duplicated in AWS, users/groups stay managed on-prem), Identity Center federates against it as the identity source, and Permission Sets map existing **AD group membership** to IAM permissions per account — giving centralized, low-maintenance, scalable multi-account access without deploying AWS Managed Microsoft AD (unneeded extra infrastructure) or creating individual IAM users per employee (doesn't scale, no SSO, no group-based management).

## [07-security-identity/iam-sts-cognito.md](07-security-identity/iam-sts-cognito.md)

> Exam-wording cue: "limit the maximum permissions a **specific IAM role/user** can have, even permissions delegated by someone else" → **Permissions Boundary**. "Limit the maximum permissions for an **entire AWS account or OU** in an Organization, affecting everyone including root" → **SCP** (Service Control Policy) — same "ceiling, not a grant" concept, but SCPs live in AWS Organizations and apply account/OU-wide, while a permissions boundary is IAM-level and applies to one identity.

## [07-security-identity/waf-shield-firewall-manager.md](07-security-identity/waf-shield-firewall-manager.md)

> Exam-wording cue: "restrict/allow access to an **ALB** by country/geography" → **WAF Web ACL with a geo-match rule** attached to the ALB — the ALB itself has no native geo-filtering. If **CloudFront** is already in front of it, geo-restriction can also be done natively at the CloudFront distribution level, without WAF. If the resource is an **NLB**, geo-match via WAF isn't possible at all — WAF only attaches to Layer 7 resources (see the NLB/WAF cue above).

## [09-migration-transfer/migration-services.md](09-migration-transfer/migration-services.md)

> Exam-wording cue: source and target are **different database engines** (e.g. Oracle → Aurora PostgreSQL) → **SCT (schema) + DMS (data/CDC)** together, in that order. Source and target are the **same engine**, just a different platform (e.g. on-prem MySQL → RDS MySQL) → **DMS alone** is sufficient; SCT is unnecessary since there's no schema translation needed.

## [11-other-services/hybrid-edge-infrastructure.md](11-other-services/hybrid-edge-infrastructure.md)

> Exam-wording cue: "AWS services must run **physically inside our own data center**" (data residency, ultra-low local latency to on-prem systems, regulatory requirement to keep data on-site) → **Outposts**. "Just need **fast, private network connectivity** between our data center and AWS, workloads still run in AWS" → **Direct Connect** (or **Transit Gateway** for hubbing multiple VPCs/on-prem sites). The tell: does the workload itself need to be on-prem, or just reachable from on-prem — Outposts answers the former, Direct Connect/Transit Gateway answer the latter.
