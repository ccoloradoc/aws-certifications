# Instance Store & EBS

## Instance Store

- Block-level **ephemeral** storage, physically attached to the host
- Ideal for: temporary data, caches, buffers, replicated data
- Very high performance, low latency
- Cost included in instance pricing
- **Hibernation** preserves memory/EBS data; stop or terminate loses instance store data

> Exam-wording cue: a scenario emphasizing **high random I/O performance / very high IOPS** for **temporary, non-critical, or reproducible** data (buffer, cache, scratch space, temporary batch-processing output) → **Instance Store** — physically attached to the host, so it's faster than any network-attached volume, but disappears on stop/terminate. A scenario needing that same performance for data that **must persist and survive an instance stop/terminate/replacement** → **EBS Provisioned IOPS (io1/io2)** instead, which trades a bit of raw speed for durability/persistence. "High I/O" alone in the question isn't enough to pick Instance Store — check whether the data can be lost.

## EBS (Elastic Block Store)

### SSD Volumes

- **General Purpose (gp2, gp3)** — low latency, good default for development/most workloads
- **Provisioned IOPS (io1, io2)** — sub-millisecond latency, sustained high IOPS for critical workloads

### HDD Volumes

- **Cold HDD (sc1)** — infrequent access, lowest cost
- **Throughput Optimized (st1)** — frequent access, good for data warehousing/big data
- **Neither st1 nor sc1 can be used as a boot volume** — only SSD volume types (gp2/gp3/io1/io2) are supported for the root/boot volume

> Exam-wording cue: the two EBS families split by *what they're optimized to measure*. **SSD volumes** (gp2/gp3, io1/io2) are built for **transactional workloads with small, frequent read/write I/O**, where **IOPS** is the metric that matters — this is also why only SSD types are allowed as a **boot volume** (an OS needs fast small random reads at startup). **HDD volumes** (st1, sc1) are built for **large sequential/streaming workloads**, where **throughput (MiB/s)**, not IOPS, is the relevant measure — think big-data/data-warehousing scans, not booting an instance. A question ruling out an option because it "cannot be a boot volume" is almost always pointing at st1 or sc1.

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
