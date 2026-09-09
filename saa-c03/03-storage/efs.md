# EFS (Elastic File System)

- Multi-AZ attachment (vs. EBS's single-AZ limitation)
- Fully managed, auto-scales with usage
- Linux only (not supported on Windows)
- POSIX permissions control file access
- Objects unused for 90 days can transition to **EFS IA** (Infrequent Access) for cost savings
- Protected by EFS Security Groups

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
