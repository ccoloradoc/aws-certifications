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

> Exam-wording cue: "move **Windows file server** workloads off-prem, need **highly reliable file storage** accessible over **SMB**, Windows-compatible — **select two**" → **Amazon FSx for Windows File Server** (native SMB/NTFS, AD/ACL integration, purpose-built Windows file share — see [fsx.md](fsx.md)) **and** **AWS Storage Gateway's File Gateway** (SMB access to S3-backed storage, also AD-integrated). **EFS** is the standard wrong third option here — Linux-only, no SMB support at all, despite also being "cloud file storage."

> Exam-wording cue: "**preserve access from local file systems**," "**optimize bandwidth during migration**," "**avoid retrieval fees or delays**," "**minimal application reconfiguration**," "**frequent local access**" → **AWS Storage Gateway — File Gateway**, backed by **S3 Standard/Standard-IA** (never Glacier, since Glacier's retrieval fees/delays directly conflict with "avoid retrieval fees"). File Gateway's local caching of hot data is what gives fee-free, delay-free access to frequently-used records, while the NFS/SMB interface means existing on-prem applications keep using ordinary file paths — no rewrite to call an S3 API. This is the specific combination that rules out both a raw S3 migration (would need app changes) and any Glacier-backed tier (retrieval fees/delays), leaving File Gateway as the only option satisfying every requirement simultaneously.

## Volume Gateway

- Block storage using the iSCSI protocol, backed by S3
- Backed by EBS snapshots, which can help restore on-premises volumes
- **Cached volumes** — low-latency access to most-recently-used data; the primary dataset lives in AWS
- **Stored volumes** — the entire dataset stays on-premises, with scheduled backups to S3

> Exam-wording cue: "hybrid DR, data available on AWS **and** on-premises must be **uniform**" (the full dataset, not just hot data) → **Stored Volumes** — the entire dataset stays on-prem for low-latency local access, while being asynchronously backed up to S3 in full. "Primary data should live in AWS, only cache hot data locally" → **Cached Volumes** instead — the primary/authoritative copy is in S3, on-prem only holds a subset.

> Exam-wording cue: "**frequently/most-accessed data cached locally**, full dataset **backed up to S3**" describes **both** File Gateway and Volume Gateway (Cached) almost identically — caching behavior alone doesn't disambiguate them. Disambiguate on **access pattern** instead: **file-level** access (NFS/SMB — individual files land as real, independently-readable S3 objects) → **File Gateway**. **Block-level** access (iSCSI — the app mounts a raw disk/volume) → **Volume Gateway (Cached)**, whose S3-backed data is an EBS-snapshot-backed volume internally, not browsable S3 objects. The tell is usually in the answer text itself: "**the full volume**" → Volume Gateway; "**the files/objects**" → File Gateway.

## Tape Gateway

- For companies with existing physical-tape backup processes — Tape Gateway lets them keep the same workflows, but in the cloud
- Virtual Tape Library (VTL) backed by S3 and Glacier
- Uses an iSCSI interface, and works with leading backup software vendors

> Exam-wording cue: "**petabytes** of data on **physical tapes**," "**without changing** current tape backup **workflows**," "**cost-optimized**" → **AWS Storage Gateway — Tape Gateway**, presenting a **Virtual Tape Library (VTL)** that existing backup software (NetBackup, Veeam, etc.) treats exactly like physical tape infrastructure — no changes to backup schedules, jobs, or tooling. The virtual tapes are backed by **S3**, and can transition to **S3 Glacier/Deep Archive** for the lowest-cost long-term archival storage, matching "cost-optimized" at petabyte scale. This is the tell whenever a question emphasizes preserving an **existing tape-based** process specifically — not a general "move backups to S3" migration, which would instead point to something like DataSync or a direct S3 upload.

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
