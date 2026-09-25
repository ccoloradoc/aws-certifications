# Amazon FSx

## Overview

- Launch third-party high-performance file systems on AWS as a fully managed service
- Family: **FSx for Windows File Server**, **FSx for Lustre**, **FSx for NetApp ONTAP**, **FSx for OpenZFS**

### Quick Comparison

| | **Windows File Server** | **Lustre** | **NetApp ONTAP** | **OpenZFS** |
|---|---|---|---|---|
| **Protocol** | SMB, Windows NTFS | Parallel distributed FS (Lustre) | NFS, SMB, iSCSI | NFS (v3/v4/v4.1/v4.2) |
| **Best for** | Windows workloads needing a native file share | HPC / ML / video processing / financial modeling | Teams already on NetApp/NAS wanting broad OS + protocol compatibility | Teams on ZFS wanting high IOPS at low latency |
| **OS support** | Windows natively, mountable on Linux too | Linux only — no Windows | Linux, Windows, macOS, VMware Cloud, EC2/ECS/EKS | Linux, Windows, macOS, VMware Cloud, EC2/ECS/EKS |
| **Standout trait** | AD integration, DFS Namespaces, daily S3 backups | Reads/writes S3 directly as a filesystem; Scratch vs. Persistent deployment | Auto-grows/shrinks storage, instant point-in-time cloning | Up to 1,000,000 IOPS at <0.5ms latency, instant cloning |

- **FSx for Windows File Server** — a fully managed Windows file share (SMB/NTFS) as a drop-in replacement for on-prem Windows file servers, with AD/ACL integration
- **FSx for Lustre** — a parallel, high-throughput HPC filesystem, tightly integrated with S3, purpose-built for compute-intensive workloads like ML training and simulations
- **FSx for NetApp ONTAP** — managed NetApp ONTAP storage for teams migrating existing ONTAP/NAS workloads, valued for its protocol flexibility (NFS/SMB/iSCSI) and platform reach
- **FSx for OpenZFS** — managed OpenZFS for teams wanting ZFS semantics (snapshots, cloning, compression) with very high IOPS at sub-millisecond latency

## FSx for Windows File Server

- Replaces on-premises Windows file servers — fully managed Windows file system share drive
- Supports SMB protocol + Windows NTFS
- Microsoft Active Directory integration, ACLs, user quotas
- Can be mounted on Linux EC2 instances
- Supports Microsoft's Distributed File System (DFS) Namespaces — groups files across multiple file systems
- Scales up to 10s of GB/s, millions of IOPS, 100s of PB of data
- Storage options: SSD (latency-sensitive workloads — databases, media processing, data analytics) or HDD (broad-spectrum workloads — home directories, CMS)
- Accessible from on-premises infrastructure via VPN or Direct Connect
- Multi-AZ capable (high availability)
- Data is backed up daily to S3

## FSx for Lustre

- Built for HPC workloads — no Windows support
- Lustre is a parallel distributed file system for large-scale computing (the name derives from "Linux" + "cluster")
- Use cases: machine learning, HPC, video processing, financial modeling, electronic design automation
- Scales up to 100s of GB/s, millions of IOPS, sub-ms latency
- Storage options: SSD (low-latency, IOPS-intensive workloads, small/random file operations) or HDD (throughput-intensive workloads, large/sequential file operations)
- Seamless S3 integration — can read S3 as a file system through FSx, and write computation output back to S3
- Accessible from on-premises servers via VPN or Direct Connect

> Exam-wording cue: Lustre itself has **no cold storage tier** — its own storage (SSD/HDD, Scratch/Persistent) is always active/hot, built for compute. But because a Lustre file system can be **linked to an S3 bucket** as its data repository, data can flow out of Lustre into S3, and once there, normal **S3 Lifecycle rules** can transition it into IA/Glacier — so Lustre *enables a path* to cold storage via S3 rather than providing cold storage itself. "Parallel/distributed high-throughput HPC filesystem" → **FSx for Lustre**. "Cold/archival storage for rarely-accessed data" alone → **S3 Glacier** (or Lifecycle rules), not Lustre. A scenario combining fast HPC compute now with eventual archiving later is valid as **Lustre linked to S3, with S3 Lifecycle handling the cold tiering**.

### Deployment Options

- **Scratch** — temporary storage, data is not replicated (doesn't persist if the file server fails), high burst (6x faster, 200MBps per TiB); use for short-term processing to optimize cost
- **Persistent** — long-term storage, data is replicated within the same AZ, failed files replaced within minutes; use for long-term processing and sensitive data

## FSx for NetApp ONTAP

- Managed NetApp ONTAP on AWS — move workloads running on ONTAP or NAS to AWS
- File system compatible with NFS, SMB, and iSCSI protocols
- Works with Linux, Windows, macOS, VMware Cloud on AWS, Amazon WorkSpaces & AppStream 2.0, and EC2/ECS/EKS
- Storage automatically shrinks or grows
- Supports snapshots, replication, low-cost compression and data deduplication
- Point-in-time instantaneous cloning — helpful for testing new workloads

## FSx for OpenZFS

- Managed OpenZFS file system on AWS — move workloads running on ZFS to AWS
- File system compatible with NFS (v3, v4, v4.1, v4.2)
- Same broad platform support as ONTAP: Linux, Windows, macOS, VMware Cloud on AWS, WorkSpaces & AppStream 2.0, EC2/ECS/EKS
- Up to 1,000,000 IOPS at <0.5ms latency
- Supports snapshots, compression, and instant point-in-time cloning

## Integrations

- **ECS/Fargate** — supports FSx for Lustre and FSx for NetApp ONTAP as task volumes (alongside EBS and EFS)
- **AWS Backup** — supports FSx (Lustre and Windows File Server), with cross-region and cross-account backups
- **AWS DataSync** — can target any FSx file system (Windows, Lustre, NetApp ONTAP, OpenZFS) as a sync destination, alongside S3 and EFS

## Notes

<!-- Your own notes go here. -->

### Protocol Landscape

Two independent layers, not one hierarchy:

```
                    NETWORK LAYER                    ON-DISK LAYER
                 (how the client and           (how bytes are physically
                  server talk to each              arranged on the
                    other over TCP)                    storage)

  Windows  →         SMB                    →         NTFS
  Linux    →         NFS                    →         ext4 (or similar)
```

These two axes are independent. The network layer is about *requesting files by name* — "give me `report.docx`," "write these bytes to `log.txt`." The on-disk layer is about *how the server itself stores those bytes* once it has them — completely invisible to the client, which never sees NTFS or ext4 directly. FSx pairs SMB with NTFS because that's the natural Windows stack, but the pairing is a convention, not a requirement.

**DFS** lives entirely inside the network layer, one level above SMB — it's a namespace/redirect table mapping logical paths to real SMB shares. No bytes move because of DFS; it just tells the client which SMB share to actually talk to.

**iSCSI breaks out of this model entirely.** SMB, NFS, and DFS are all *file-level*: the client asks for something by filename and the server's filesystem resolves it. iSCSI is *block-level*: the server exposes a raw range of storage blocks — no files, no names, no directory structure — and the client mounts it as if it were a local unformatted disk, then lays down its own filesystem (NTFS, ext4, whatever) on top itself. iSCSI doesn't replace SMB/NFS with something fancier — it replaces the entire file-level abstraction and pushes the on-disk-format decision onto the client instead of the server.

| | Layer | What it does |
|---|---|---|
| SMB / NFS | Network (file-level) | Request/serve files by name |
| NTFS | On-disk | How the server physically stores bytes (invisible to the client) |
| DFS | Network (routing, above SMB) | Maps a logical path to the right SMB share |
| iSCSI | Network (block-level) | Serves raw disk blocks; client owns the filesystem, not the server |

### NAS, ZFS, and WAFL

**NAS (Network Attached Storage)** — a category, not a specific protocol: a dedicated storage device/appliance that sits on the network and serves files to multiple clients, typically over NFS or SMB. Contrasted with a **SAN (Storage Area Network)**, which serves raw blocks (over iSCSI or Fibre Channel) rather than files. When the FSx slides say "move workloads running on ONTAP or NAS to AWS," they mean: a physical NetApp (or similar) box on-prem serving files to the network, which FSx for ONTAP replicates in AWS.

**ZFS (Zettabyte File System)** — an on-disk filesystem, in the same category as NTFS or ext4 (not a network protocol). Originally built by Sun Microsystems for Solaris, now widely used on Linux/BSD. Notable vs. something like ext4:
- **Copy-on-write** — never overwrites data in place, which is what makes instant snapshots and clones cheap and safe
- **Built-in volume management** — pools multiple disks together without needing a separate RAID layer
- **Checksums everything** — detects (and with redundancy, repairs) silent data corruption
- **Native compression and deduplication**

FSx for OpenZFS takes that on-disk format and serves it over NFS — same layering split as above: ZFS is the on-disk layer, NFS is the network layer on top of it.

**WAFL (Write Anywhere File Layout)** — NetApp's proprietary on-disk filesystem, sitting at the exact same layer as ZFS/NTFS. It's what NetApp storage arrays have run since the 90s, and what FSx for NetApp ONTAP runs on in AWS. Same copy-on-write philosophy as ZFS — never overwrite in place — which is why ONTAP and OpenZFS both headline "auto-grow/shrink storage, instant point-in-time cloning": two independent implementations of the same architectural idea.

That completes the on-disk picture for all four FSx variants:

| FSx variant | On-disk filesystem | Network protocol |
|---|---|---|
| Windows File Server | NTFS | SMB |
| Lustre | Lustre (purpose-built, combines both layers) | Lustre |
| NetApp ONTAP | WAFL | NFS / SMB / iSCSI |
| OpenZFS | ZFS | NFS |

Content sourced from slide deck, pages 331-360, 361-390, 421-450, and 781-810.

## Acronyms

| Acronym | Meaning |
|---|---|
| DFS | Distributed File System |
| iSCSI | Internet Small Computer Systems Interface |
| NAS | Network Attached Storage |
| NFS | Network File System |
| NTFS | New Technology File System |
| SAN | Storage Area Network |
| SMB | Server Message Block |
| WAFL | Write Anywhere File Layout |
| ZFS | Zettabyte File System |
