# AWS Storage Gateway

- Replaces on-premises storage without changing existing workflows
- **Types**:
  - **File Gateway** — NFS/SMB access
  - **Volume Gateway** — block storage (iSCSI)
  - **Tape Gateway** — virtual tape library for backup software
- Stores data in S3
- Provides a low-latency local cache (compared to going direct to EFS/EBS)

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

### From slides (pages 271-420)

- Use cases: DR, backup & restore, tiered storage, on-prem cache/low-latency access — the bridge for exposing S3's proprietary storage on-premises (unlike EFS/NFS)
- S3 File Gateway: exposes S3 buckets over NFS/SMB, caches most-recently-used data locally; supports Standard, Standard-IA, One Zone-IA, Intelligent-Tiering (transition to Glacier via lifecycle policy); IAM role per gateway for bucket access; SMB integrates with Active Directory
- Volume Gateway: block storage over iSCSI, backed by S3 (via EBS snapshots, so on-prem volumes are restorable) — Cached volumes (low-latency access to recent data, rest in S3) vs. Stored volumes (full dataset kept on-prem, scheduled S3 backups)
- Tape Gateway: Virtual Tape Library backed by S3 + Glacier, iSCSI interface, integrates with existing tape backup software/workflows
- Deployment options: VM (VMware, Hyper-V, KVM) or hardware appliance
