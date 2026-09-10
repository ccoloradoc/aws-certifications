# AWS Storage Gateway

## Overview

- Bridge between on-premises data and cloud data — replaces on-premises storage without changing existing workflows
- The reason it exists: S3 is a proprietary storage technology (unlike EFS/NFS), so Storage Gateway is what exposes S3 data on-premises
- Use cases: disaster recovery, backup & restore, tiered storage, on-premises cache & low-latency file access
- Stores data in S3, and provides a low-latency local cache (compared to going direct to EFS/EBS)
- **Types**:
  - **File Gateway** — NFS/SMB access
  - **Volume Gateway** — block storage (iSCSI)
  - **Tape Gateway** — virtual tape library for backup software
- Deployment options: VM (VMware, Hyper-V, KVM) or a hardware appliance

## File Gateway

- Configured S3 buckets are accessible using the NFS and SMB protocols
- Most recently used data is cached locally in the file gateway
- Supports S3 Standard, Standard-IA, One Zone-IA, and Intelligent-Tiering; transition to Glacier via an S3 Lifecycle policy
- Bucket access is granted via an IAM role per File Gateway
- SMB integrates with Active Directory (AD) for user authentication

## Volume Gateway

- Block storage using the iSCSI protocol, backed by S3
- Backed by EBS snapshots, which can help restore on-premises volumes
- **Cached volumes** — low-latency access to most-recently-used data; the primary dataset lives in AWS
- **Stored volumes** — the entire dataset stays on-premises, with scheduled backups to S3

## Tape Gateway

- For companies with existing physical-tape backup processes — Tape Gateway lets them keep the same workflows, but in the cloud
- Virtual Tape Library (VTL) backed by S3 and Glacier
- Uses an iSCSI interface, and works with leading backup software vendors

## Integrations

- **AWS Backup** supports Storage Gateway (Volume Gateway specifically), with cross-region and cross-account backups
- Appears in AWS's DR-tips guidance as one of the standard on-premises → AWS backup/replication bridges, alongside Snowball (see [snow-family.md](../09-migration-transfer/snow-family.md))

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> Storage Gateway framed as a bridge appliance (virtual or hardware) sitting in your on-prem data center, connecting local storage/backup workflows to AWS.
>
> — *Netec S3, 22:39-23:27, 25:07-25:34*

> File Gateway uses NFS/SMB-style protocols on the local side, backed by S3 as the destination — named use case: extending on-prem file-share/backup capacity into the cloud, and integration with FSx (e.g. Windows File Server).
>
> — *Netec S3, 25:40-26:03, 27:12-27:27*

> Tape Gateway simulates a virtual tape library (VTL) so you can replace physical backup tapes with cloud storage while keeping existing backup software workflows; backups can land in S3 Glacier, replacing physical tape rotation between offsite locations.
>
> — *Netec S3, 27:34-28:07*

> Volume Gateway modes flagged as a typical exam question — know which mode keeps data primarily where: Cached volumes (primary data lives in AWS, most-recently-used data cached locally) vs. Stored volumes (primary data stays on-prem, AWS holds backups).
>
> — *Netec S3, 28:16-29:52*

> General Storage Gateway use cases: hybrid backup/DR scenarios, corporate file sharing, migrating physical tape backups to the cloud, archiving historical data to the cloud. Practical advice: combine with S3 lifecycle policies to move Storage-Gateway-originated backups to cheaper storage classes over time.
>
> — *Netec S3, 30:23-30:56, 31:22-31:53*

Content sourced from slide deck, pages 361-390 and 781-810.
