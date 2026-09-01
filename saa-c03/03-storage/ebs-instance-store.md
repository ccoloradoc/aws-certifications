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
