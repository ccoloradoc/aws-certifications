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

### From slides (pages 271-420)

- Use cases: DR, backup & restore, tiered storage, on-prem cache/low-latency access — the bridge for exposing S3's proprietary storage on-premises (unlike EFS/NFS)
- S3 File Gateway: exposes S3 buckets over NFS/SMB, caches most-recently-used data locally; supports Standard, Standard-IA, One Zone-IA, Intelligent-Tiering (transition to Glacier via lifecycle policy); IAM role per gateway for bucket access; SMB integrates with Active Directory
- Volume Gateway: block storage over iSCSI, backed by S3 (via EBS snapshots, so on-prem volumes are restorable) — Cached volumes (low-latency access to recent data, rest in S3) vs. Stored volumes (full dataset kept on-prem, scheduled S3 backups)
- Tape Gateway: Virtual Tape Library backed by S3 + Glacier, iSCSI interface, integrates with existing tape backup software/workflows
- Deployment options: VM (VMware, Hyper-V, KVM) or hardware appliance
