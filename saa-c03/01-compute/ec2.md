# EC2

## Instance Types

AWS offers 300+ EC2 instance types across 5 instance families, each with varying resource focuses (compute, memory, storage, GPU, general purpose).

- Naming convention: `[class][generation].[size]` — e.g. `m5.2xlarge` = M class, 5th gen, 2xlarge size
- Sizing & configuration options at launch: OS (Linux/Windows/macOS), CPU/cores, RAM, storage (network-attached EBS/EFS vs. hardware Instance Store), network card (speed + public IP), firewall (security group), and a bootstrap script (EC2 User Data)
- Family use cases (beyond the naming convention):
  - **General Purpose** — balances compute/memory/networking; good for web servers, code repositories (e.g. `t2.micro`)
  - **Compute Optimized** — high-performance processors for batch processing, media transcoding, high-performance web servers, HPC, scientific modeling/ML, dedicated gaming servers
  - **Memory Optimized** — fast performance for large in-memory datasets: relational/non-relational databases, distributed caches, in-memory BI databases, real-time big-data processing
  - **Storage Optimized** — high sequential local read/write throughput: OLTP systems, relational/NoSQL databases, in-memory cache (e.g. Redis), data warehousing, distributed file systems

## Purchasing Options

- **On-Demand Instances** — pay per second/hour, no commitment
- **On-Demand Capacity Reservations** — reserve capacity in an AZ; no time commitment, no discount, billed at On-Demand rate whether used or not; combine with Reserved Instances/Savings Plans to add a discount
- **Spot Instances** — 50–90% discount, can be reclaimed by AWS
  - **Spot Fleets** — a set of Spot (+ optional On-Demand) instances across multiple launch pools; allocation strategies: `lowestPrice`, `diversified`, `capacityOptimized`, `priceCapacityOptimized` (recommended default)
- **Reserved Instances** — up to 72% discount (this supersedes an older "40-60%" figure some cheat sheets still quote — AWS's current numbers go higher, especially 3-year All Upfront); 1 or 3 year term; No/Partial/All Upfront payment; Regional or Zonal scope; can trade on the Reserved Instance Marketplace
  - **Convertible Reserved Instances** — up to 66% discount; can change instance type, family, OS, scope, and tenancy
- **Savings Plans** — commit to $/hour usage for 1 or 3 years (up to 72% discount); locked to instance family + region, but flexible on size/OS/tenancy; usage beyond the commitment bills at On-Demand price
- **Dedicated Instances** — dedicated hardware, shared with other instances of the same account
- **Dedicated Hosts** — physically isolated hardware, useful for licensing/compliance
- **Bare Metal EC2 Instances** — direct access to underlying server hardware

## Launch Configuration

- **Launch Templates** — store instance launch parameters for reuse
- **User data** — up to 16KB of bootstrap script; runs **once**, at first boot only, and executes as the **root** user; used to automate tasks like installing updates/software or downloading files at launch
- **Instance metadata** — available via URI or query tool (IMDS)
- **Root device volumes** — EBS-backed or Instance Store-backed
- **Run Command** (SSM) — manage live instances without SSH
- **EC2 Instance Connect** — browser-based SSH, no key file, AWS uploads a temporary key; works out-of-the-box only on Amazon Linux 2; port 22 must still be open

## EC2 Hibernate

- Preserves in-memory (RAM) state to a file on the root EBS volume for faster reboot
- Root EBS volume must be encrypted; RAM must be under 150GB; not supported on bare metal
- Supported families include C3/C4/C5/I3/M3/M4/R3/R4/T2/T3
- Max 60 days hibernated; available for On-Demand, Reserved, and Spot

## Placement Groups

- **Cluster** — low latency, high throughput, good for HPC (single AZ)
- **Partition** — distributes instances across logical partitions, reduces correlated failure; up to 7 partitions per AZ, spans multiple AZs, scales to 100s of instances; a partition failure doesn't affect other partitions; used for HDFS/HBase/Cassandra/Kafka
- **Spread** — each instance on distinct hardware, reduces correlated failure for small critical workloads; max 7 instances per AZ per group; can span AZs

## Elastic Network Interfaces (ENI)

An ENI is what a security group and an IP address actually attach to — see the full fundamentals, plus the "what can have an ENI / what can have a security group" comparison tables, in [vpc.md](../02-networking/vpc.md). The one EC2-specific fact: an ENI can be created independently of any instance, then moved between EC2 instances on the fly for failover (still bound to the AZ it was created in).

## Scalability & High Availability

- **Scalability** — an application/system's ability to handle greater load by adapting; two kinds:
  - **Vertical Scalability** — increase the instance's own size (e.g. `t2.micro` → `t2.large`, or as far as `t2.nano` at 0.5GB RAM/1 vCPU up to `u-12tb1.metal` at 12.3TB RAM/448 vCPUs); common for non-distributed systems like a single database; RDS and ElastiCache scale this way; hits a hardware ceiling eventually
  - **Horizontal Scalability** (= elasticity) — increase the *number* of instances/systems instead; implies a distributed system; the natural fit for modern/web applications; achieved via an Auto Scaling Group + Load Balancer
- **High Availability** — running an application across at least 2 data centers (i.e. 2+ Availability Zones) so it survives losing one; usually goes hand-in-hand with horizontal scaling, but is a related, distinct concept from scalability
  - Can be **passive** (e.g. RDS Multi-AZ standby) or **active** (e.g. a horizontally-scaled ASG serving traffic from every AZ)
  - For EC2 specifically: HA means running the ASG and Load Balancer across multiple AZs, not just scaling within one

## Auto Scaling

- **Auto Scaling Groups (ASG)** paired with Elastic Load Balancers
- ASG attributes: Launch Template (AMI + instance type, user data, EBS volumes, security groups, key pair, IAM role, network/subnets), Load Balancer info, Min/Max/Initial capacity, scaling policies — ASGs themselves are free, you only pay for the underlying instances
- **Scaling policies**: Simple, Scheduled, Dynamic, Step, Target Tracking
  - Scales on CloudWatch alarms (metric computed across the whole ASG); good metrics to scale on: `CPUUtilization`, `RequestCountPerTarget`, Network In/Out, or a custom pushed metric
  - **Predictive Scaling** — continuously forecasts load and schedules scaling ahead of time
- **Cooldown periods** (default 300s) affect how quickly instances are terminated/launched after a scaling activity — the ASG won't launch/terminate more instances until metrics stabilize; using a ready-to-use AMI reduces boot time and lets you shorten the cooldown

## Notes

<!-- Your own notes go here. -->
