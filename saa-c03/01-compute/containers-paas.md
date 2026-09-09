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

### From Netec live training (to review)

> Elastic Beanstalk introduced by direct contrast with CloudFormation: less infrastructure control but a much simpler experience — upload code (Git or ZIP) and AWS provisions/runs a fitting environment, without you defining the underlying resources explicitly. Still runs on real infrastructure (e.g. EC2) underneath, but you don't manage that layer directly — though you can configure some redundancy options (single vs. multiple instances, load balancer, auto scaling). Configuration inputs: the application "container," the environment (dev/test/production), and the platform.
>
> — *Netec S4, 52:37-55:41, 54:40-55:04, 56:06-56:40*

> Beanstalk use cases: rapid deployment without managing a VPC or EC2 directly; migrating an existing app with minimal redesign by matching an existing supported platform (e.g. a Java app onto Tomcat, a .NET app onto Windows/.NET); quickly standing up an MVP/proof-of-concept. Can integrate with SQS and other AWS services within the app's runtime environment.
>
> — *Netec S4, 57:45-58:48, 57:11-57:19*

> Decoupling motivation: tightly coupled systems mean a failure in one component (e.g. payment service) can take down the whole app; decoupling (e.g. via a queue between app and payment service) means a failure in one part leaves the rest still (partially) functioning. Monolithic vs. microservices: microservices split an app into independent, decoupled components that can each be deployed/scaled/updated independently, potentially with their own datastore. Explicit caveat: microservices are not automatically the right answer for everything — added complexity/tooling/cost depending on team maturity; monolithic isn't inherently "bad."
>
> — *Netec S4, 2:12:33-2:14:06, 2:15:02-2:18:20, 2:16:06-2:19:21*

> Containers vs. VMs: a container is lighter than a VM because it doesn't need a full guest OS — just a runtime engine plus the app's dependencies; a VM needs a hypervisor plus a full guest OS per instance. Running Docker manually inside an EC2 instance is possible but scaling becomes hard because containers are then bounded by that one instance's capacity — why AWS offers dedicated container-orchestration services.
>
> — *Netec S4, 2:20:34-2:24:26, 2:24:54-2:26:19*

> Exam heuristic restated: prefer the managed-service answer — a container-orchestration service over self-managing Docker on EC2, because it needs less low-level administration.
>
> — *Netec S4, 2:26:19-2:27:34*

> Container image explained as the container's equivalent of an AMI — a packaged, reusable, layered bundle (base OS layer, runtime, app code, dependencies). Registry (ECR) explained as the storage/versioning layer for images, compared directly to a Git repository. Private repository (default) is access-restricted via IAM with vulnerability scanning; public repository is openly pullable, similar in spirit to Docker Hub — public image galleries (AWS's own included) are a source of pre-built base images.
>
> — *Netec S4, 2:28:59-2:30:09, 2:30:16-2:31:00, 2:32:50-2:36:19, 2:36:27-2:37:54, 2:39:15-2:39:51*

> ECS described mechanically: a cluster (group of resources) runs services, defined by task definitions (a template specifying container image, CPU/memory, env vars — analogous to how an AMI+instance-type defines an EC2 launch), and a task is a running instance of a task definition. ECS integrates with the registry, Route 53 for service discovery, IAM, CloudWatch Logs, and load balancers.
>
> — *Netec S4, 2:41:07-2:43:41, 2:44:11-2:44:26, 2:45:51*

> EKS described as the equivalent orchestration option for teams already using/preferring Kubernetes — AWS manages the Kubernetes control plane, reducing operational complexity while staying Kubernetes-compatible. Guidance: new to containers → ECS is more approachable; existing Kubernetes experience or multi-cloud-portability need → EKS fits better. "ECS/EKS Anywhere" named for running these platforms on-premises too.
>
> — *Netec S4, 2:47:06-2:48:51, 2:49:24-2:49:56, 2:52:50-2:53:09*

> Repeated exam heuristic: "minimal operational effort / not managing infrastructure" → Fargate; "server control required" → EC2 (as a launch type) — flagged as a frequently tested distinction.
>
> — *Netec S4, 2:51:44-2:53:43*

> Series of live container exam scenarios: (1) need scalability/availability, do not want to manage/provision infrastructure → Fargate (class initially leaned ECS/EKS-on-EC2 before correcting toward Fargate specifically); (2) on-prem containerized app, growing load, must move to AWS with minimal code/effort changes → ECS + Fargate + a load balancer; (3) on-prem containerized app, high volume, needs HA + minimal operational effort → ECR + ECS + Fargate, doubling as an "eliminate unnecessary-effort answers" drill (hosting your own registry when ECR exists, or building a custom AMI when unneeded, both flagged as wrong); (4) stateless workloads that can tolerate interruptions, minimize cost and operational load → Spot Instances + EKS/ECS — "supports interruptions" rules out On-Demand and points to Spot, "containers" points away from a raw EC2 ASG.
>
> — *Netec S4, 2:54:01-2:56:44, 2:56:46-2:58:44, 2:58:46-3:01:39, 3:01:49-3:05:46*
