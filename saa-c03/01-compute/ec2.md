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

### Amazon Machine Images (AMIs)

- A template for launching instances: AWS-provided, your own custom AMI (built from a configured instance, to avoid reconfiguring standard/security settings every time), or sourced from AWS Marketplace (third-party vendor AMIs, e.g. a security appliance, directly launchable)
- An AMI is really just **metadata + a pointer to an EBS snapshot** holding the actual block-level data — both the AMI registration and its backing snapshot are region-scoped objects
- **Copying an AMI to another region** copies the underlying EBS snapshot's data into that region first, then registers a new AMI there pointing at the copy — this is why a "copy AMI" action always produces a new snapshot in the destination region: the snapshot *is* the data being duplicated, not a side effect
- **EC2 Image Builder** — AWS-managed service for maintaining a custom-AMI pipeline (version and reuse a "template" for future launches, avoiding rework)

> Exam-wording cue: "copying an AMI to another region creates an extra/unexpected EBS snapshot there" is expected behavior, not a bug or wasted cost to eliminate — an AMI cannot exist in a region without its backing snapshot in that same region. A question framing this as a problem to solve is testing whether you understand AMIs are snapshot-backed, not that something is misconfigured.

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

> Exam-wording cue: demand follows a **known, predictable calendar pattern** (e.g. payroll runs every 1st of the month, a marketing event with a known start time) → **Scheduled Scaling** — set capacity ahead of time, no metric needed. Demand is **live/reactive to current load** (CPU spikes unpredictably) → **Dynamic/Target Tracking Scaling**. Demand has a **recurring but not manually-known pattern** AWS can learn from historical data → **Predictive Scaling**. A scenario stating the exact date/time of an expected spike is the giveaway for Scheduled, not Predictive or Dynamic.

> Exam-wording cue: within **Dynamic Scaling** itself, **Target Tracking** is the "just tell me the number" option — you set a target value for a metric (e.g. "keep average CPU at 40%"), and AWS automatically creates and manages the CloudWatch alarms and capacity adjustments to hold it there; this is AWS's recommended default for most threshold-based scaling and the answer whenever the scenario just wants a metric held near a target with minimal setup. **Step Scaling** is for when the *magnitude* of the breach should drive the *size* of the response (e.g. CPU at 90% scales out more instances than CPU at 65%) — you define the alarm and multiple adjustment steps yourself. **Simple Scaling** is the legacy option: one alarm, one fixed adjustment, then it waits out a full cooldown before evaluating again — rarely the correct exam answer once Target Tracking or Step Scaling is offered as an alternative.

- **Cooldown periods** (default 300s) affect how quickly instances are terminated/launched after a scaling activity — the ASG won't launch/terminate more instances until metrics stabilize; using a ready-to-use AMI reduces boot time and lets you shorten the cooldown

### Maintenance Without Losing Instances

- **Standby state** — manually pull a single `InService` instance out of the ASG's active rotation to patch/troubleshoot it: it's deregistered from the load balancer and excluded from health checks/metrics, but stays part of the group (not terminated); move it back to `InService` when done
  - When you put an instance into Standby you choose whether to **decrement desired capacity**: decrement it and the ASG leaves the gap alone; don't decrement it and the ASG launches a replacement instance to keep serving at full capacity while you work on the standby one
- **Suspending the `ReplaceUnhealthy` process** — a group-wide alternative: ASGs run several automatic background processes (`Launch`, `Terminate`, `HealthCheck`, `ReplaceUnhealthy`, `AZRebalance`, `AlarmNotification`, `ScheduledActions`, `AddToLoadBalancer`); suspending just `ReplaceUnhealthy` stops the ASG from terminating/replacing instances that fail health checks — useful when planned maintenance (e.g. a rolling patch that takes an app briefly offline) would otherwise look like an unhealthy instance and get killed and replaced mid-work

> Exam-wording cue: need to work on **one specific instance** without it being terminated or losing overall serving capacity → **Standby state** (don't decrement desired capacity). Need to perform maintenance **across the group** where health checks would otherwise flag instances as unhealthy during the work (e.g. a manual rolling update) → **suspend the `ReplaceUnhealthy` process** for the duration, then resume it. Both avoid termination; Standby is per-instance and explicit, `ReplaceUnhealthy` suspension is group-wide and health-check-driven.

> Exam-wording cue: instance-replacement **order** differs by process. **`AZRebalance`** (fixing an AZ imbalance) always **launches the replacement first, then terminates** the old instance — it can even temporarily exceed max size (by ~10%, rounded up) to do so, since the instances being replaced aren't broken, just unevenly distributed, and AWS avoids a capacity dip. **`ReplaceUnhealthy`** (a single failed instance) does the opposite: **terminates the unhealthy instance first, then launches** a replacement — no allowance to exceed max size, since the bad instance should come out immediately. A question asking "does the ASG launch or terminate first" hinges entirely on *which* process is triggering the replacement.

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> Historical framing: AWS compute evolved from "access to physical racks" toward a virtualization layer you provision directly from the console/API, without lower-level hardware management.
>
> — *Netec S2, 2:06:18-2:07:03*

> For the Architect Associate exam, the compute focus is stated as primarily EC2, EBS, and Lambda.
>
> — *Netec S2, 2:07:37-2:07:53*

> EC2 launch configuration walked through field by field: name/tags, AMI selection, instance type/size (chosen for capability needs), key pair for SSH vs. Session Manager as the safer alternative (avoids permanently exposing a port to the internet, reduces brute-force attack surface). Instance creation also requires VPC, subnet, public-IP auto-assign choice, and security group.
>
> — *Netec S2, 2:09:41-2:12:59, 2:13:18-2:13:58*

> EBS is explicitly grouped under "compute" (not purely storage) because it's the disk volume attached to an instance — OS and data volumes both configurable at launch; lifecycle actions: delete, stop, hibernate; multi-AZ HA configuration options also mentioned.
>
> — *Netec S2, 2:13:58-2:14:50*

> User Data: an OS-dependent script uploaded at launch time that runs automatically when the instance boots (e.g. auto-install a web server) so the instance "arrives" already configured.
>
> — *Netec S2, 2:14:58-2:15:42*

> Tags/metadata can mark an instance as production/test, used for tracking, search filtering, and especially cost tracking by tag.
>
> — *Netec S2, 2:16:08-2:16:47*

> AMIs described as templates — AWS-provided, your own custom AMI built from a configured instance (to avoid reconfiguring standard/security settings every time), or sourced from AWS Marketplace (third-party vendor AMIs, e.g. a security appliance, directly launchable).
>
> — *Netec S2, 2:16:55-2:18:52*

> EC2 Image Builder named directly as the AWS-managed service for maintaining a custom-AMI pipeline — version and reuse a "template" for future launches, avoiding rework.
>
> — *Netec S2, 2:24:36-2:24:56*

> Instance-type selection framed with a consumer-hardware analogy: choosing an EC2 instance type is like choosing a laptop/PC — general use needs modest specs, gaming/large databases need much more; picking the wrong type wastes money or under-serves the workload.
>
> — *Netec S2, 2:20:07-2:21:31*

> Instance type naming convention: family class (workload it's optimized for) + generation number + additional properties (e.g. processor type) + size after the dot.
>
> — *Netec S2, 2:21:42-2:22:06*

> Instance families with use cases: General Purpose (balanced, most default use cases); Memory Optimized (large in-memory data sets, DB servers); Compute Optimized (CPU-heavy, critical/high-performance apps, ML); Storage Optimized (large local databases, high I/O).
>
> — *Netec S2, 2:22:46-2:24:12*

> AWS Compute Optimizer named directly as a rightsizing/cost tool — checks whether the chosen instance type/size is over- or under-utilized.
>
> — *Netec S2, 2:25:44-2:26:06*

> Tenancy: shared by default (normal in a multi-tenant cloud); dedicated instance isolates hardware at the account level; dedicated host goes further, giving a specific physical server you fully control (including bringing your own licenses) — relevant for regulated industries with strict compliance/security requirements.
>
> — *Netec S2, 2:26:22-2:28:40*

> Placement Groups explained conceptually as logical organization of instances (separate from physical tenancy): Cluster (instances that mostly talk to each other, for performance), Spread (reduce correlated hardware failure across a distributed system), Partition (system aware of instance topology across partitions).
>
> — *Netec S2, 2:28:44-2:30:15*

> User Data and instance metadata both named as sources of information collectible from a running instance — metadata includes instance ID, IP addresses, usable within applications.
>
> — *Netec S2, 2:30:15-2:31:08*

> EBS volume types recap: general purpose (SSD or magnetic, cheaper), higher-performance SSD/premium disks optimized for I/O throughput for production, magnetic/cold-storage disks for infrequently accessed data — explicit reminder that disk performance (not just CPU/memory/network) affects application performance. Cost warning: unattached/leftover EBS volumes still bill for storage even if unused.
>
> — *Netec S2, 2:32:14-2:33:42, 2:33:42-2:34:08*

> EC2 purchasing options recap: On-Demand (quick tests/unpredictable traffic), Reserved Instances/Savings Plans (1-3yr commitment, stable/predictable workloads), Spot Instances (auction-style, up to ~90% discount, but can be interrupted/reclaimed — explicitly flagged as not suitable for workloads requiring high availability).
>
> — *Netec S2, 2:34:41-2:36:54*

> Horizontal scaling (scale-out/scale-in): add more instances to handle increased demand (e.g. a ticket sale event), then scale back in once demand subsides — the flexibility the cloud provides over fixed on-prem capacity. Vertical scaling: increase/decrease the capacity (CPU, storage, etc.) of a single resource, contrasted with horizontal.
>
> — *Netec S3, 3:27:52-3:29:24, 3:29:31-3:29:54*

> Launch Template: captures the configuration you'd otherwise set launching an individual instance (AMI, instance type, etc.), reused so every instance launched from it is identically configured. Auto Scaling Group: instances following a launch template placed into a group with minimum/desired/maximum capacity; if an instance fails, the group relaunches a replacement; can integrate with a load balancer and span multiple AZs for resilience.
>
> — *Netec S3, 3:30:20-3:30:49, 3:31:04-3:31:47*

> Scaling policy types: manual, scheduled (anticipate a known recurring demand pattern, e.g. a payroll system, and pre-emptively scale up), dynamic/reactive (based on a live metric like CPU usage), and predictive (learns a baseline usage pattern over time and proactively scales ahead of anticipated demand, requiring historical data first).
>
> — *Netec S3, 3:31:52-3:34:53*
