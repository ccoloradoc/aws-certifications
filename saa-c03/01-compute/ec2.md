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
  - **Spot Fleets** — a set of Spot (+ optional On-Demand) instances across multiple launch pools, launched to meet a target capacity; by default automatically replaces terminated Spot Instances to keep target capacity maintained. Allocation strategies:
    - **`lowestPrice`** — always picks whichever pool is currently cheapest; best when you just want the absolute lowest cost and don't need a specific instance type/size (the fleet decides the instance type for you from across its diversified pools)
    - **`diversified`** — spreads instances across multiple pools to reduce the impact of any single pool being interrupted
    - **`capacityOptimized`** — picks pools AWS predicts are least likely to be interrupted (favors stability over lowest price)
    - **`priceCapacityOptimized`** (recommended default) — balances both: looks at pools with the best capacity availability, then optimizes for price among those

> Exam-wording cue: a workload can run on **"multiple servers of various sizes, with a variable number of CPUs"** and just wants the **cheapest possible** capacity, with no preference for a specific instance type → **Spot Fleet with the `lowestPrice` allocation strategy** — this is exactly what lets you leave instance-type selection to AWS instead of hardcoding one type/size in advance, completing the "EMR on Spot" answer for a short, fault-tolerant, distributable batch job (see the analytics notes' EMR+Spot cue).

> Exam-wording cue: "**capabilities of Spot Instances/Fleets — select multiple**" → three separate, correct mechanics worth knowing individually: (1) a **persistent** Spot request **reopens automatically** after the instance is interrupted (a **one-time** request does not — it's fulfilled once and done); (2) a **Spot Fleet** automatically launches **replacement instances** to maintain its target capacity when instances are terminated; (3) **canceling an active Spot request does NOT terminate its running instance** — the request and the instance are separate things, and the instance must be terminated as its own explicit step. Distractors on this topic usually invert #1 or #3 — claiming a persistent request "gives up" after interruption, or that canceling a request "kills" the instance.
- **Reserved Instances** — up to 72% discount (this supersedes an older "40-60%" figure some cheat sheets still quote — AWS's current numbers go higher, especially 3-year All Upfront); 1 or 3 year term; No/Partial/All Upfront payment; Regional or Zonal scope; can trade on the Reserved Instance Marketplace
  - **Convertible Reserved Instances** — up to 66% discount; can change instance type, family, OS, scope, and tenancy
- **Savings Plans** — commit to $/hour usage for 1 or 3 years (up to 72% discount); locked to instance family + region, but flexible on size/OS/tenancy; usage beyond the commitment bills at On-Demand price
- **Dedicated Instances** — dedicated hardware, shared with other instances of the same account; AWS manages host allocation for you (no visibility into which physical server, no control over instance placement); billed per instance-hour plus a small one-time account-level fee
- **Dedicated Hosts** — an entire specific physical server dedicated to you, with full visibility (host ID, sockets, physical cores) and control over instance placement on it; billed **per host**, whether fully utilized or not — built for BYOL software licensed per-socket/per-core, or compliance requiring proof of exactly which physical server something ran on
- **Bare Metal EC2 Instances** — direct access to underlying server hardware

> Exam-wording cue: "isolate instances to a single tenant" / "single-tenant hardware" alone, with **no mention of per-socket/per-core licensing or needing host-level visibility** → **Dedicated Instances** is the more cost-effective answer — both options satisfy single-tenancy, but Dedicated Hosts charge for an entire host's capacity to provide visibility/control the scenario doesn't ask for. Regulatory/compliance wording alone doesn't automatically mean Dedicated Hosts; only reach for Dedicated Hosts when the scenario specifically needs host ID/socket/core visibility or BYOL licensing tied to physical hardware.

> Exam-wording cue: "quick tests / unpredictable, spiky traffic, no commitment" → **On-Demand**. "Stable, predictable, long-running workload, willing to commit 1-3 years for a discount" → **Reserved Instances** (fixed instance type/family) or **Savings Plans** (more flexible on size/OS/tenancy). "Fault-tolerant/interruptible workload, maximum cost savings, don't need a guaranteed specific instance type" → **Spot** (explicitly **not** suitable for anything requiring high availability, unless the workload itself tolerates losing instances). "Guaranteed capacity in a specific AZ, no discount needed" → **On-Demand Capacity Reservation**. "Licensing tied to physical cores/sockets, or compliance requiring dedicated hardware" → **Dedicated Hosts**; "just need isolated hardware, don't care about visibility into sockets/cores" → **Dedicated Instances** (cheaper, less control).

## Instance Tenancy

- Three tenancy options: **Shared** (default — multiple AWS accounts may share the same physical hardware), **Dedicated** (single-tenant hardware, no other customer shares the physical server), **Dedicated Host** (an entire specific physical server, full visibility/control — see Dedicated Hosts above)
- Tenancy can be set at **two levels** that must be reconciled: the **VPC's own tenancy attribute** (`default` or `dedicated`, set at VPC creation) and the **instance/Launch Template's own tenancy setting**
- **Resolution rule — tenancy always resolves toward `dedicated`** if either side specifies it:
  - A VPC with tenancy attribute `dedicated` **only allows** `dedicated` or `host` tenancy instances to be launched into it at all — `shared` isn't even an option there, regardless of what the Launch Template says
  - A VPC with tenancy attribute `default` (shared) still lets you **explicitly override** to `dedicated` at the instance/Launch Template level, even though the VPC's own default is shared
  - The only way to get a genuinely **shared**-tenancy instance is when **both** the VPC and the Launch Template are left at their default/shared setting

> Exam-wording cue: **tenancy resolves toward dedicated** whenever **either** the Launch Template **or** the VPC specifies it. If the **VPC's tenancy attribute is `dedicated`**, it simply won't allow shared-tenancy instances at all — everything launched into it becomes dedicated (or host) regardless of what the Launch Template says. If the VPC's tenancy is `default` (shared) but the **Launch Template explicitly specifies `dedicated`**, that explicit setting overrides the VPC's shared default. The only way to get a genuinely **shared**-tenancy instance is when **both** the Launch Template **and** the VPC are left at their default/shared setting — any single "dedicated" on either side is enough to force the result to dedicated.

## Launch Configuration

- **Launch Templates** — store instance launch parameters for reuse
- **User data** — up to 16KB of bootstrap script; runs **once**, at first boot only, and executes as the **root** user with **full (root-level) privileges** — no `sudo` needed in the script itself; used to automate tasks like installing updates/software or downloading files at launch
  - Accessible from within the instance via the **Instance Metadata Service (IMDS)** at `169.254.169.254/latest/user-data`
  - **Not encrypted** by default — don't put secrets/credentials directly in it; fetch them at runtime from Secrets Manager/SSM Parameter Store instead
- **Instance metadata** — available via URI or query tool (IMDS)
- **Root device volumes** — EBS-backed or Instance Store-backed
- **Run Command** (SSM) — manage live instances without SSH
- **EC2 Instance Connect** — browser-based SSH, no key file, AWS uploads a temporary key; works out-of-the-box only on Amazon Linux 2; port 22 must still be open

> Exam-wording cue: "run a script **once**, automatically, right when an instance first launches, with full/root privileges, no `sudo` needed" → **User Data**. "Run commands on an **already-running** instance, on-demand, without SSH" → **SSM Run Command** instead — User Data only fires at initial boot, never again, and always executes as root regardless of the AMI's default login user.

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

> Exam-wording cue: "**stop/start** cycle" + "**slow application startup / auxiliary software re-initialization** every time it's started" → **EC2 Hibernate** — it specifically preserves *in-memory state* across stop/start, skipping the OS boot process and re-initialization of whatever was already running in RAM. A faster instance type, a custom AMI, or a User Data script wouldn't fix this, since the bottleneck is the application's own runtime initialization, not raw boot speed.

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
  - **Minimum-cost math for "at least N instances always available, tolerating a single AZ failure"**: spreading N instances across just **2 AZs** requires **N instances in *each* AZ (2N total)** to guarantee N remain if either AZ fails — you don't know in advance which AZ goes down, so each must independently hold full capacity. Spreading across **3 AZs** instead, at roughly N/2 per AZ (~1.5N total), means losing any one AZ still leaves N running, since only 1/3 of capacity is lost per AZ instead of 1/2 — meeting the same guarantee with fewer total instances

> Exam-wording cue: "at least N instances **always** available, tolerating a **single AZ failure**, minimum cost" → don't just spread N across 2 AZs (that actually requires 2N total instances to guarantee the floor, not N). The cost-minimal design uses **3 AZs**, each holding roughly N/2 instances, so losing any one AZ still leaves N running — meeting the requirement with ~1.5N total instances instead of 2N.

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

### Troubleshooting: ASG Not Terminating an Unhealthy Instance

Besides a suspended `ReplaceUnhealthy` process, three other common (non-misconfiguration) reasons an unhealthy instance isn't being replaced yet:

- **`HealthCheckType` doesn't include ELB unless explicitly configured** — by default an ASG only checks EC2 status; an instance failing its ALB/NLB target health check won't be seen as unhealthy at all until `HealthCheckType` is set to include `ELB`
- **The health check grace period hasn't expired** — a newly-launched (or recently-relaunched) instance gets a grace window before failing checks count against it
- **The instance may be in `Impaired` status** — Amazon EC2 Auto Scaling doesn't immediately terminate an `Impaired` instance; it deliberately waits a few minutes to give it a chance to self-recover, and may also delay/skip action entirely if there's insufficient CloudWatch status-check data to act on confidently

> Exam-wording cue: "why isn't my ASG replacing an unhealthy instance" (troubleshooting, not "how do I prevent it") → check, in order: is `ReplaceUnhealthy` suspended, does `HealthCheckType` actually include `ELB`, has the health check grace period expired, and is the instance simply still within its post-`Impaired` wait window. All four are legitimate, non-bug explanations — the ASG is often working as designed, just not yet.

> Exam-wording cue: instance-replacement **order** differs by process. **`AZRebalance`** (fixing an AZ imbalance) always **launches the replacement first, then terminates** the old instance — it can even temporarily exceed max size (by ~10%, rounded up) to do so, since the instances being replaced aren't broken, just unevenly distributed, and AWS avoids a capacity dip. **`ReplaceUnhealthy`** (a single failed instance) does the opposite: **terminates the unhealthy instance first, then launches** a replacement — no allowance to exceed max size, since the bad instance should come out immediately. A question asking "does the ASG launch or terminate first" hinges entirely on *which* process is triggering the replacement.

## CloudWatch Alarm Actions: Stop, Terminate, Reboot, Recover

- A CloudWatch alarm on a **standalone EC2 instance** (not necessarily in an ASG) can trigger one of four actions: **Stop**, **Terminate**, **Reboot**, or **Recover**
- **Recover** responds to the **`StatusCheckFailed_System`** metric — a host/hardware-level failure — by migrating the instance to new underlying hardware during a reboot. **`StatusCheckFailed_Instance`** (an OS-level issue) is handled by a simple **Reboot** action instead, not Recover

> Exam-wording cue: "**CloudWatch alarm** automatically **recovers** an **impaired** EC2 instance — **select two correct statements**" → (1) the recovered instance keeps its **same instance ID, private IPs, Elastic IPs, and metadata** — it's identical from the outside; (2) **in-memory (RAM) data is lost**, since recovery works by migrating the instance to new underlying hardware during a reboot. The triggering metric is **`StatusCheckFailed_System`** (a host/hardware-level failure) — **not** `StatusCheckFailed_Instance` (an OS-level issue, which points to a simple **reboot** action instead, not recovery).

> Exam-wording cue: "**single EC2 instance**, no ASG, **no elaborate DR strategy**, small user base, **max downtime of 10 minutes**, **cost-effective and automatic** recovery **for the instance**" → a **CloudWatch alarm on `StatusCheckFailed_System`** with the **"Recover this instance"** action. No new infrastructure (no ASG, no multi-AZ/DR architecture) is needed — just an alarm on the existing instance; it automatically restores the *same* instance (ID, IPs, data) within minutes of a host-level failure, comfortably inside a 10-minute downtime bound, at effectively zero extra cost.

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
