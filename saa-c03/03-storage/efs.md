# EFS (Elastic File System)

- Multi-AZ attachment (vs. EBS's single-AZ limitation)
- Fully managed, auto-scales with usage
- Linux only (not supported on Windows)
- POSIX permissions control file access
- Objects unused for 90 days can transition to **EFS IA** (Infrequent Access) for cost savings
- Protected by EFS Security Groups — each **mount target** is an ENI in a VPC subnet, and like any ENI it has an attached security group; the mount target's SG must allow inbound **NFS (port 2049)** from the client's SG (or CIDR), and the client must allow outbound on that same port

> Exam-wording cue: "restrict which instances/subnets can even connect to an EFS file system" → **Security Groups on the mount targets** (network-layer gate — can you reach port 2049 at all). "Restrict what a connected client can read/write once mounted" → **POSIX permissions** (or IAM + Access Points for finer-grained, per-client-path control) — SGs don't control file/directory-level access, only network reachability.

## Cross-Account Access

Mounting one EFS file system from a different AWS account requires three separate things to all be true at once:

1. **Network connectivity between the two VPCs** — mount targets are just ENIs with private IPs in a subnet, so the client's VPC needs a path to the file system's VPC via **VPC Peering** or **Transit Gateway**
2. **A File System Policy on the EFS file system itself** (the owning account) — a resource-based policy (like an S3 bucket policy) granting the other account's principal `elasticfilesystem:ClientMount`, `ClientWrite`, `ClientRootAccess`, etc.
3. **An IAM policy in the consuming account** authorizing its own principals (e.g. the EC2 instance role) to call those same `elasticfilesystem:Client*` actions against that specific file system's ARN

> Exam-wording cue: "share one EFS file system across multiple AWS accounts/VPCs" needs all three — **VPC Peering/Transit Gateway** (network) **+ an EFS File System Policy** (resource-based, on the file system) **+ IAM permissions in the caller's account**. A distractor that only sets up one or two of these (e.g. "just peer the VPCs" or "just add an IAM policy") is incomplete — miss any one piece and the mount fails.

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> EFS explained as a network file system (NFS-like) supporting concurrent access from multiple instances/applications simultaneously, contrasted with EBS's one-volume-to-one-instance model.
>
> — *Netec S2, 3:27:10-3:28:10*

> Mount targets — one per Availability Zone — are what let compute resources in different AZs connect to the same EFS file system; can also be scoped to a single AZ to cut cost. EFS access protected via Security Groups, same mechanism as EC2.
>
> — *Netec S2, 3:28:45-3:29:07, 3:29:57-3:30:03, 3:30:14-3:30:26*

> Performance modes and storage classes named directly: standard vs. lower-latency/high-throughput performance modes; storage classes include Standard and an infrequent-access tier; lifecycle policies move files between classes for cost savings. Destination use case named: shared storage backing containers on ECS/Kubernetes.
>
> — *Netec S2, 3:30:36-3:31:30, 3:31:30-3:31:52*

> Storage-type comparison summary: block storage (EBS) → mostly manual scaling, one-to-one with an instance; object storage (S3) → highly scalable, unstructured data; file storage (EFS) → concurrent/shared access across instances/containers with automatic scaling.
>
> — *Netec S2, 3:31:52-3:32:45*

### From slides (pages 1-120)

- Use cases: content management, web serving, data sharing, WordPress
- Performance modes (set at creation time): General Purpose (default, latency-sensitive) vs. Max I/O (higher latency/throughput, highly parallel workloads)
- Throughput modes: Bursting (1TB = 50MiB/s baseline + burst), Provisioned (set fixed throughput regardless of storage size), Elastic (auto-scales, up to 3GiB/s read / 1GiB/s write — good for unpredictable workloads)
- Storage classes / lifecycle tiers: Standard (frequent access) → EFS-IA (infrequent access, cheaper storage/costlier retrieval) → Archive (rare access, ~50% cheaper); lifecycle policies move files automatically between tiers
- Availability tiers: Standard (multi-AZ, for prod) vs. One Zone (single AZ, for dev, backup enabled by default; combined with One Zone-IA gives over 90% cost savings)
- Encryption at rest via KMS
