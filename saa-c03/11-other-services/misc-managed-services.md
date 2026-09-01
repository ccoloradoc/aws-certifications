# Other Managed Services

- **AWS App Mesh** — application networking/service mesh for microservices
- **AWS CodeStar** — quick project setup for development/deployment pipelines
- **AWS Batch** — batch computing job processing
- **Amazon Lex** — conversational voice/text interfaces (chatbots)
- **Amazon Connect** — cloud-based call center platform
- **Amazon SES** (Simple Email Service) — marketing/transactional email service
- **Amazon AppStream** — application streaming service
- **Amazon Elastic Transcoder** — video/audio format conversion
- **AWS CLI** — command-line interface, requires IAM access keys
- **AWS LightSail** — simplified VPS-style alternative bundling servers, MySQL, storage, and load balancing
- **AWS IoT Core** — connects and manages IoT devices in the cloud
- **Amazon Neptune** — managed graph database for interactive graph queries

## Notes

<!-- Your own notes go here. -->

### From slides (pages 721-870)

- **Amazon SES** detail: inbound + outbound email at scale, reputation dashboard, delivery/bounce/complaint stats, DKIM + SPF support, shared/dedicated/customer-owned IP options, send via Console/API/SMTP; for transactional, marketing, and bulk email
- **Amazon Pinpoint** — 2-way (outbound+inbound) marketing communications across email/SMS/push/voice/in-app, with segmentation, personalization, and full campaign management (templates, schedules, targeted segments) — the key difference from SNS/SES is that those make you manage audience/content/schedule yourself per message, while Pinpoint gives you campaign tooling on top
- **AWS Batch**: fully managed batch job processing (a "batch" job has a defined start/end, unlike a continuous service); dynamically launches EC2 or Spot Instances sized to the job's needs; jobs are Docker images run on ECS/EKS/Fargate. Batch vs. Lambda — Lambda has a time limit, limited runtimes, and limited temp disk (fully serverless); Batch has no time limit, supports any Docker-packaged runtime, and relies on EBS/instance store for disk (backed by EC2, optionally AWS-managed)
- **Amazon AppFlow** — managed integration between SaaS apps (Salesforce, SAP, Zendesk, Slack, ServiceNow, etc.) and AWS (S3, Redshift) or other SaaS (Snowflake), on a schedule/event/on-demand, with built-in filtering/validation and either public-internet or PrivateLink transport — avoids hand-writing point-to-point integrations
- **AWS Amplify** — toolkit for building/deploying full-stack web & mobile apps: auth, storage, REST/GraphQL APIs, CI/CD, pub/sub, analytics, AI/ML predictions, monitoring; connects to GitHub/CodeCommit/Bitbucket/GitLab or direct upload
- **Instance Scheduler on AWS** — a CloudFormation-deployed *solution* (not a standalone service) that auto start/stops EC2, ASGs, and RDS instances on a schedule (e.g. off outside business hours) for up to 70% savings; schedules live in a DynamoDB table, driven by resource tags + Lambda; supports cross-account/cross-region

### High Performance Computing (HPC) overview (from slides, pages 721-870)

Cloud elasticity fits HPC well (genomics, computational chemistry, risk modeling, weather prediction, ML, autonomous driving) — pay only for the burst of resources used. Building blocks, mostly detailed elsewhere in these notes:
- **Data transfer**: Direct Connect (GB/s, ongoing), Snowball/Snowmobile (PB-scale one-time), DataSync (ongoing on-prem↔S3/EFS/FSx)
- **Compute/networking**: CPU/GPU-optimized EC2, Spot/Spot Fleet + Auto Scaling for cost, Cluster Placement Groups for low latency (see [ec2.md](../01-compute/ec2.md))
  - Enhanced Networking (SR-IOV): **ENA** up to 100Gbps (current) vs. the legacy Intel 82599 VF up to 10Gbps
  - **Elastic Fabric Adapter (EFA)** — an ENA built for tightly-coupled HPC inter-node communication, Linux-only, uses MPI, bypasses the OS kernel for low-latency reliable transport
- **Storage**: instance-attached (EBS up to 256,000 IOPS on io2 Block Express, or Instance Store for millions of IOPS) vs. network storage (S3 for blobs, EFS scaling IOPS with size or via provisioned IOPS, **FSx for Lustre** as the HPC-optimized distributed filesystem backed by S3 — see [fsx.md](../03-storage/fsx.md))
- **Orchestration**: AWS Batch for multi-node parallel jobs across instances; **AWS ParallelCluster** — open-source, config-file-driven HPC cluster deployment tool that automates VPC/subnet/cluster/instance-type setup and can enable EFA
