# Instance Store & EBS

## Instance Store

- Block-level **ephemeral** storage, physically attached to the host
- Ideal for: temporary data, caches, buffers, replicated data
- Very high performance, low latency
- Cost included in instance pricing
- **Hibernation** preserves memory/EBS data; stop or terminate loses instance store data

## EBS (Elastic Block Store)

### SSD Volumes

- **General Purpose (gp2, gp3)** — low latency, good default for development/most workloads
- **Provisioned IOPS (io1, io2)** — sub-millisecond latency, sustained high IOPS for critical workloads

### HDD Volumes

- **Cold HDD (sc1)** — infrequent access, lowest cost
- **Throughput Optimized (st1)** — frequent access, good for data warehousing/big data

### Limitations

- Single-AZ attachment (Multi-Attach exists but only within a single AZ)
- Single point of failure unless snapshotted/replicated
- Not fully managed — doesn't auto-scale

### Management

- **EBS Data Lifecycle Manager** automates snapshot backups
- Supports encryption at rest and in transit

## Notes

<!-- Your own notes go here. -->

### From slides (pages 1-120)

- Delete on Termination — root EBS volume is deleted by default when the instance terminates; other attached volumes are not deleted by default (both configurable)
- Snapshot features:
  - Snapshot Archive tier — 75% cheaper, restore takes 24-72 hours
  - Recycle Bin — retains deleted snapshots for recovery (configurable 1 day - 1 year retention)
  - Fast Snapshot Restore (FSR) — eliminates first-use latency on a restored snapshot ($$$)
- AMI creation process: launch & customize an instance → stop it (for data integrity) → build the AMI (also creates EBS snapshots) → launch new instances from it
- gp3 vs gp2: gp3 has a 3,000 IOPS / 125 MiB/s baseline and lets you scale IOPS/throughput independently of size; gp2 IOPS is tied to volume size (3 IOPS/GiB, max 16,000 IOPS, bursts to 3,000 IOPS on small volumes)
- io1 max 64,000 PIOPS on Nitro instances (32,000 on others); io2 Block Express has sub-millisecond latency and max 256,000 PIOPS
- Multi-Attach (io1/io2 only) — up to 16 EC2 instances in the same AZ, each with full read/write access; requires a cluster-aware file system (not XFS/EXT4)
- Encryption uses KMS (AES-256); encrypting an existing unencrypted volume: snapshot it → copy the snapshot with encryption enabled → create a new volume from that snapshot → attach
