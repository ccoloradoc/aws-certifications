# EFS (Elastic File System)

- Multi-AZ attachment (vs. EBS's single-AZ limitation)
- Fully managed, auto-scales with usage
- Linux only (not supported on Windows)
- POSIX permissions control file access
- Objects unused for 90 days can transition to **EFS IA** (Infrequent Access) for cost savings
- Protected by EFS Security Groups

## Notes

<!-- Your own notes go here. -->

### From slides (pages 1-120)

- Use cases: content management, web serving, data sharing, WordPress
- Performance modes (set at creation time): General Purpose (default, latency-sensitive) vs. Max I/O (higher latency/throughput, highly parallel workloads)
- Throughput modes: Bursting (1TB = 50MiB/s baseline + burst), Provisioned (set fixed throughput regardless of storage size), Elastic (auto-scales, up to 3GiB/s read / 1GiB/s write — good for unpredictable workloads)
- Storage classes / lifecycle tiers: Standard (frequent access) → EFS-IA (infrequent access, cheaper storage/costlier retrieval) → Archive (rare access, ~50% cheaper); lifecycle policies move files automatically between tiers
- Availability tiers: Standard (multi-AZ, for prod) vs. One Zone (single AZ, for dev, backup enabled by default; combined with One Zone-IA gives over 90% cost savings)
- Encryption at rest via KMS
