# EC2

## Instance Types

AWS offers 300+ EC2 instance types across 5 instance families, each with varying resource focuses (compute, memory, storage, GPU, general purpose).

## Purchasing Options

- **On-Demand Instances** — pay per second/hour, no commitment
- **On-Demand Capacity Reservations** — reserve capacity in an AZ without commitment
- **Spot Instances** — 50–90% discount, can be reclaimed by AWS
- **Reserved Instances** — 1–3 year commitment, 40–60% discount
- **Dedicated Instances** — dedicated hardware, shared with other instances of the same account
- **Dedicated Hosts** — physically isolated hardware, useful for licensing/compliance
- **Bare Metal EC2 Instances** — direct access to underlying server hardware

## Launch Configuration

- **Launch Templates** — store instance launch parameters for reuse
- **User data** — up to 16KB of bootstrap script, runs on first boot
- **Instance metadata** — available via URI or query tool (IMDS)
- **Root device volumes** — EBS-backed or Instance Store-backed
- **Run Command** (SSM) — manage live instances without SSH

## Placement Groups

- **Cluster** — low latency, high throughput, good for HPC (single AZ)
- **Partition** — distributes instances across logical partitions, reduces correlated failure
- **Spread** — each instance on distinct hardware, reduces correlated failure for small critical workloads

## Auto Scaling

- **Auto Scaling Groups (ASG)** paired with Elastic Load Balancers
- **Scaling policies**: Simple, Scheduled, Dynamic, Step, Target Tracking
- **Cooldown periods** affect how quickly instances are terminated/launched after a scaling activity

## Notes

<!-- Your own notes go here. -->

### From slides (pages 1-120)

- Instance naming convention: `[class][generation].[size]` — e.g. `m5.2xlarge` = M class, 5th gen, 2xlarge size
- Purchasing options detail:
  - Reserved Instances — up to 72% discount (supersedes the "40-60%" figure in the seed cheat sheet above — AWS's current numbers go higher, especially 3-year All Upfront); 1 or 3 year term; No/Partial/All Upfront payment; Regional or Zonal scope; can trade on the Reserved Instance Marketplace
  - Convertible Reserved Instances — up to 66% discount; can change instance type, family, OS, scope, and tenancy
  - Savings Plans — commit to $/hour usage for 1 or 3 years (up to 72% discount); locked to instance family + region, but flexible on size/OS/tenancy; usage beyond the commitment bills at On-Demand price
  - Spot Fleets — a set of Spot (+ optional On-Demand) instances across multiple launch pools; allocation strategies: `lowestPrice`, `diversified`, `capacityOptimized`, `priceCapacityOptimized` (recommended default)
  - Capacity Reservations — no time commitment, no discount, billed at On-Demand rate whether used or not; combine with Reserved Instances/Savings Plans to add a discount
- EC2 Instance Connect — browser-based SSH, no key file, AWS uploads a temporary key; works out-of-the-box only on Amazon Linux 2; port 22 must still be open
- EC2 Hibernate — preserves in-memory (RAM) state to a file on the root EBS volume for faster reboot
  - Root EBS volume must be encrypted; RAM must be under 150GB; not supported on bare metal
  - Supported families include C3/C4/C5/I3/M3/M4/R3/R4/T2/T3
  - Max 60 days hibernated; available for On-Demand, Reserved, and Spot
- Placement Groups detail:
  - Spread — max 7 instances per AZ per group; can span AZs
  - Partition — up to 7 partitions per AZ, spans multiple AZs, scales to 100s of instances; a partition failure doesn't affect other partitions; used for HDFS/HBase/Cassandra/Kafka
- Elastic Network Interface (ENI) — can be created independently and moved between EC2 instances on the fly for failover; bound to a specific AZ
- ASG attributes: Launch Template (AMI + instance type, user data, EBS volumes, security groups, key pair, IAM role, network/subnets), Load Balancer info, Min/Max/Initial capacity, scaling policies — ASGs themselves are free, you only pay for the underlying instances
- ASG scales on CloudWatch alarms (metric computed across the whole ASG); good metrics to scale on: `CPUUtilization`, `RequestCountPerTarget`, Network In/Out, or a custom pushed metric
- Predictive Scaling — continuously forecasts load and schedules scaling ahead of time
- Cooldown period (default 300s) — after a scaling activity, the ASG won't launch/terminate more instances until metrics stabilize; using a ready-to-use AMI reduces boot time and lets you shorten the cooldown
