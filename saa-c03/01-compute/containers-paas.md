# Containers & PaaS

## Amazon ECS (Elastic Container Service)

- Docker container management, highly scalable and high-performance
- **Launch Types**:
  - **Fargate** — serverless, AWS-managed infrastructure
  - **EC2** — direct instance access, manual management
- **ECS Service Auto Scaling** via Application Auto Scaling
- Task-based billing: pay for running task time
- Uses **Task IAM roles** (not instance-level roles/groups)

## AWS Elastic Beanstalk

- PaaS for quick application deployment
- Built on EC2 — not serverless
- Handles provisioning, load balancing, scaling, and health monitoring for you

## Notes

<!-- Your own notes go here. -->

### From slides (pages 121-270)

- Elastic Beanstalk components: Application (collection of environments/versions/configs), Application Version (a code iteration), Environment (the AWS resources running one version at a time, of type Web Server Tier or Worker Tier) — you can run multiple environments (dev/test/prod)
- Supported platforms: Go, Java SE, Java+Tomcat, .NET Core (Linux) / .NET (Windows), Node.js, PHP, Python, Ruby, Packer Builder, and Docker (single container, multi-container, or preconfigured)
- Web Server Tier vs Worker Tier — the worker tier pulls messages from an SQS queue and scales based on queue depth; a web tier environment can push messages into that queue for the worker tier to process
- Deployment modes include Single Instance (good for dev)
- Beanstalk itself is free — you only pay for the underlying resources (EC2, RDS, ELB, etc.); you keep full control over configuration despite it being managed

### From slides (pages 271-420)

- Docker packages apps into containers that run identically across any host OS — no compatibility issues, predictable behavior, works with any language/OS/tech; good for microservices and lift-and-shift migrations
- Docker images live in repositories: Docker Hub (public) or Amazon ECR (private, plus a public gallery at gallery.ecr.aws)
- Container platform options: Amazon ECS (AWS's own container orchestrator), Amazon EKS (managed Kubernetes), AWS Fargate (serverless compute layer that works with either ECS or EKS), Amazon ECR (image storage for both)

### ECS/EKS/ECR deep dive (from slides, pages 421-570)

- ECS EC2 launch type: you provision/maintain the EC2 instances, each running the ECS Agent to register with the cluster; AWS still handles starting/stopping containers on them
- IAM roles for ECS: **EC2 Instance Profile** (EC2 launch type only — used by the ECS Agent itself for API calls, CloudWatch Logs, pulling from ECR, reading Secrets Manager/SSM) vs. **ECS Task Role** (per-task permissions, defined in the task definition, so different services can have different roles)
- Load balancer support: ALB for most use cases; NLB for high-throughput/high-performance or PrivateLink pairing; CLB supported but not recommended (no advanced features, no Fargate support)
- EFS can be mounted onto ECS tasks (EC2 or Fargate) for shared, persistent, multi-AZ storage — Fargate + EFS is fully serverless. S3 cannot be mounted as a filesystem
- ECS Service Auto Scaling (task count) is separate from EC2 Auto Scaling (instance count) — the former uses Application Auto Scaling on metrics like Average CPU/Memory or ALB Request Count Per Target, via Target Tracking, Step, or Scheduled scaling; Fargate makes this simpler since there's no underlying EC2 fleet to also scale
- For the EC2 launch type, capacity itself scales either via a plain ASG (on CPU) or an **ECS Capacity Provider** paired with an ASG (adds EC2 instances specifically when the cluster is short on CPU/RAM)
- ECS tasks can be invoked by EventBridge (including on a schedule) and can consume from SQS queues; EventBridge can also intercept "Task Stopped" state-change events
- Amazon ECR: private + public repos, backed by S3, IAM-controlled access, supports vulnerability scanning, image tags/versioning, and lifecycle policies
- Amazon EKS: managed Kubernetes control plane; alternative to ECS with a different (cloud-agnostic) API — good fit if you're already running Kubernetes on-prem or elsewhere; deploy one cluster per region for multi-region; logs/metrics via CloudWatch Container Insights
  - Node types: Managed Node Groups (EKS creates/manages an ASG of EC2 nodes for you, On-Demand or Spot), Self-Managed Nodes (you create/register nodes yourself, can use the EKS-Optimized AMI, ASG-managed), or Fargate (no nodes to manage at all)
  - Data volumes need a CSI-compliant driver + a StorageClass manifest; supports EBS, EFS (works with Fargate), FSx for Lustre, and FSx for NetApp ONTAP
