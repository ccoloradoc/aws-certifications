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
