# Amazon FSx

- Replaces on-premises Windows file servers
- Multi-AZ capable
- Supports DFS protocol
- Active Directory integration

## FSx for Lustre

- Built for HPC workloads
- No Windows support

## Notes

<!-- Your own notes go here. -->

### From slides (pages 271-420)

- FSx for Windows: SMB protocol + Windows NTFS, AD integration, ACLs, user quotas; mountable on Linux EC2; supports DFS Namespaces; scales to 10s of GB/s and millions of IOPS; SSD (latency-sensitive) or HDD (general-purpose) storage; accessible on-prem via VPN/Direct Connect; Multi-AZ capable; daily backups to S3
- FSx for Lustre: parallel distributed file system for ML/HPC/video processing/financial modeling/EDA; scales to 100s of GB/s, millions of IOPS, sub-ms latency; SSD (low-latency, small random I/O) or HDD (throughput-heavy, large sequential I/O); reads S3 as a filesystem and can write results back to S3; accessible from on-prem via VPN/Direct Connect
  - Deployment options: **Scratch** (temporary, not replicated, high burst, cheapest — short-term processing) vs. **Persistent** (long-term, replicated within the AZ, fast failed-file replacement — sensitive/long-term workloads)
- FSx for NetApp ONTAP: managed NetApp ONTAP; NFS/SMB/iSCSI compatible; works with Linux/Windows/macOS/VMware Cloud/Workspaces/AppStream/EC2/ECS/EKS; auto-grows/shrinks storage; supports snapshots, replication, compression, dedup, and instant point-in-time cloning
- FSx for OpenZFS: managed OpenZFS; NFS v3/v4/v4.1/v4.2 compatible; same broad platform support as ONTAP; up to 1,000,000 IOPS at <0.5ms latency; snapshots, compression, instant cloning
