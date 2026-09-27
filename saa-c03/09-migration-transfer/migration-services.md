# Migration & Transfer Services

## File & Data Transfer

- **AWS DataSync** — copies data to S3 (any storage class, incl. Glacier), EFS, or FSx (Windows/Lustre/ONTAP/OpenZFS); needs an agent for on-prem/other-cloud → AWS transfers (source: NFS, SMB, HDFS, S3 API) — no agent needed for AWS-to-AWS transfers; schedulable (hourly/daily/weekly), one agent task can push up to 10Gbps (or a set bandwidth cap).
  > Frequent exam point: DataSync preserves file permissions/metadata (POSIX, SMB).

> Exam-wording cue: "move on-prem data into **Amazon S3, Amazon EFS, and/or Amazon FSx** easily/quickly/cost-effectively," "**automate and accelerate online data transfers**" → **AWS DataSync** — the one service natively targeting all three destinations from a single tool, over the network (**"online"** rules out the physical **Snow Family**), **schedulable** (automate) with up to **10Gbps per agent task** (accelerate). Naming multiple AWS storage destinations together in the same question is the specific tell for DataSync over a narrower, single-target tool.

- **AWS Transfer Family** — fully managed FTP/FTPS/SFTP endpoint in front of S3 or EFS; scalable, reliable, Multi-AZ; pay per provisioned endpoint-hour + data transfer; stores user credentials itself or integrates with AD/LDAP/Okta/Cognito/custom auth; use cases: file sharing, public datasets, CRM/ERP integration

> Exam-wording cue: "legacy vendors/partners need **SFTP/FTP/FTPS** (not S3 API), fully managed, no infrastructure, **federated per-user access scoped to specific S3 buckets/prefixes**" → **AWS Transfer Family** with a **custom identity provider** (or AD/LDAP/Okta/Cognito) mapping each user to a scoped IAM role/session policy. If a scenario doesn't require the SFTP/FTP protocol specifically (the vendor could call an API or use a pre-signed URL instead), Transfer Family is unnecessary overhead — the protocol requirement is the specific tell that points here.

> Exam-wording cue: "SFTP file arrives, must be **processed immediately** — not periodic polling, not a separately-configured S3 event trigger" → **AWS Transfer Family Managed Workflows** — attached directly to the Transfer Family server, triggers the moment an upload completes, and can invoke a **Lambda custom step** (which reports back via `SendWorkflowStepState`) to process the file, chain further steps, or branch to exception handling on failure. This is the native, purpose-built mechanism — distinct from generic **S3 Event Notifications**, which would also work but isn't what Transfer Family itself exposes as its own first-class trigger.
- **AWS Import/Export** — physically ship storage devices (HDDs) to import large datasets into S3; legacy predecessor to the [Snow Family](snow-family.md)
- Large one-time transfer rule of thumb: 200TB over a 100Mbps internet/VPN link ≈ 185 days; over 1Gbps Direct Connect ≈ 18.5 days (DX setup itself often takes a month+); via Snowball ≈ about a week end-to-end, combinable with DMS for the cutover; for ongoing replication, prefer Site-to-Site VPN or DX paired with DMS/DataSync rather than one-off transfers

### Transfer Comparison

| Service | Summary |
|---|---|
| Transfer Family | FTP/FTPS/SFTP interface on top of S3 or EFS |
| DataSync | Schedules data sync from on-premises to AWS, or AWS to AWS |
| Import/Export | Ship physical HDDs to import large datasets into S3 (legacy, superseded by [Snow Family](snow-family.md)) |
| [Snowcone](snow-family.md) | Smallest Snow Family device, edge computing + data transfer (~8TB) |
| [Snowball Edge](snow-family.md#device-specs) | Storage Optimized (210TB) or Compute Optimized (28TB, dedicated edge compute) |
| [Snowmobile](snow-family.md) | Exabyte-scale data transfer via shipping container |

## Database Migration

- **AWS DMS** (Database Migration Service) — copy/migrate databases with minimal downtime; source stays available/running throughout; supports homogeneous (Oracle→Oracle) and heterogeneous (SQL Server→Aurora) migrations, plus ongoing Change Data Capture (CDC) replication; supports a full one-time load, ongoing change replication, or both; requires an EC2 instance to host the replication task; Multi-AZ deployment gives a synchronous standby replica (redundancy, no I/O freezes, fewer latency spikes)
  - Sources: on-prem/EC2-hosted Oracle/SQL Server/MySQL/MariaDB/PostgreSQL/MongoDB/SAP/DB2, Azure SQL Database, any RDS engine incl. Aurora, S3, DocumentDB
  - Targets: on-prem/EC2-hosted Oracle/SQL Server/MySQL/MariaDB/PostgreSQL/SAP, RDS, Redshift, DynamoDB, S3, OpenSearch, Kinesis Data Streams, Kafka, DocumentDB, Neptune, Redis, Babelfish
- **AWS SCT** (Schema Conversion Tool) — converts database schemas between engine types (e.g. SQL Server/Oracle → MySQL/PostgreSQL/Aurora for OLTP, Teradata/Oracle → Redshift for OLAP); **not needed** when the engine doesn't change (e.g. on-prem PostgreSQL → RDS PostgreSQL is still PostgreSQL, just a different platform)
- **SCT + DMS together, for a heterogeneous migration**: SCT converts the schema first (tables, views, stored procedures translated to the target engine's dialect, before any data moves), then DMS migrates the actual data (full load, plus ongoing CDC replication if configured) into that converted schema. Neither is sufficient alone here — DMS moves data but doesn't translate schema, and SCT translates schema but doesn't move or replicate data

> Exam-wording cue: source and target are **different database engines** (e.g. Oracle → Aurora PostgreSQL) → **SCT (schema) + DMS (data/CDC)** together, in that order. Source and target are the **same engine**, just a different platform (e.g. on-prem MySQL → RDS MySQL) → **DMS alone** is sufficient; SCT is unnecessary since there's no schema translation needed.

- RDS/Aurora same-engine migration paths (MySQL and PostgreSQL both): (1) snapshot-and-restore into Aurora, or (2) create an Aurora Read Replica from the RDS source and promote it once replication lag hits zero (slower/costlier but near-zero downtime); for external (non-RDS) sources: dump to S3 and import (faster), or use DMS if both DBs are live

## Server & VM Migration

- **AWS Server Migration Service (SMS)** — migrate on-premises VMs to AWS (superseded by AWS Application Migration Service / MGN, below)
- **AWS Application Migration Service (MGN)** — the current lift-and-shift/rehost service (AWS's evolution of CloudEndure Migration), replacing SMS; converts physical/virtual/cloud servers to run natively on AWS with minimal downtime, broad OS/DB/platform support
- **VM Import/Export** — bring existing VMware/KVM/VirtualBox/Hyper-V VMs into EC2, or export EC2 instances back out (e.g. to build an on-prem DR repository); can also download an Amazon Linux 2 AMI as a portable VM image
- **VMware Cloud on AWS** — run actual VMware vSphere software on AWS infrastructure, for shops standardized on VMware that want to extend into AWS or build a DR strategy without changing tooling

## Discovery & Planning

- **AWS Application Discovery Service** — inventories on-prem servers to plan a migration: Agentless mode (VM inventory/config/perf via AWS Agentless Discovery Connector) or Agent-based mode (system config/perf/processes/network connections via AWS Application Discovery Agent); results feed into **AWS Migration Hub**

## Resource Sharing

- **AWS Resource Access Manager (RAM)** — lets one "owner" account share specific resources with other AWS accounts, so those accounts can use them directly instead of duplicating or connecting to them separately
  - Shareable resources include: **VPC subnets** (VPC Sharing), Transit Gateways, License Manager configurations, Route 53 Resolver rules, and others
  - **Sharing is free** — RAM itself adds no cost; you only pay for the underlying resource as usual
  - **Within the same AWS Organization**: shares are accepted automatically, no manual invitation step needed
  - **Outside an Organization** (an explicitly invited external account): the recipient account must manually accept the resource share invitation
- **VPC Sharing** (RAM's most commonly-tested use case): the owner account shares one or more **subnets** of its VPC with other accounts; those accounts can then launch their own resources (EC2, RDS, etc.) **directly into the shared subnets**. Since every participant's resources are actually sitting in the **same VPC**, they communicate as ordinary same-VPC traffic — no VPC Peering, Transit Gateway, or any other connecting construct is needed, and there's no inter-VPC data transfer charge, since there's no "inter-VPC" traffic to begin with

> Exam-wording cue: "multiple accounts **within an AWS Organization**, single region, need to communicate privately, **cheapest**" → **VPC Sharing via AWS RAM** — sharing subnets puts everyone in one VPC, avoiding both the full-mesh complexity of VPC Peering *and* the inter-VPC data transfer cost either Peering or Transit Gateway would still carry. Reach for **VPC Peering** instead only when each account must keep its **own separate VPC** (e.g. for isolation/compliance reasons) rather than sharing one; reach for **Transit Gateway** when you need centralized routing/transitive connectivity across many *separate* VPCs, accepting its extra hourly + per-GB cost for that operational simplicity.

> Exam-wording cue: RAM isn't a universal replacement for cross-account sharing — it specifically fills the gap for resource types with **no resource-based policy of their own** (VPC subnets, Transit Gateway, License Manager configurations, Route 53 Resolver rules). "Share a resource across accounts" + the resource is one of these → **AWS RAM**. If the resource is **S3/SQS/SNS/KMS** (which *do* support resource-based policies), a **resource-based policy** is the more direct, native mechanism — RAM isn't typically the answer there. VPC Sharing specifically only displaces **VPC Peering/Transit Gateway** in the narrow case where accounts don't actually need separate VPCs — it doesn't replace them as concepts, since Peering/TGW remain necessary whenever genuine per-account VPC isolation is required.

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 271-420 and 721-870.

### From Netec live training (to review)

> AWS DataSync: online (network-based) data-transfer tool; requires installing an agent (on-prem or in the cloud) that reads from a source (e.g. NFS, SMB) and transfers encrypted data to AWS; supports incremental sync, verifies data integrity during transfer, supports scheduling/automation. Source/destination pairs named: on-prem NFS/SMB → S3, EFS, or FSx. Contrasted with Storage Gateway: Storage Gateway uses an appliance as the bridge, DataSync uses an agent configured with an explicit source and destination and discrete transfer tasks.
>
> — *Netec S3, 32:52-34:41, 34:09-34:26, 34:50-35:05*

> AWS Transfer Family named directly: transfers files into/out of AWS using traditional protocols like FTP/SFTP, functioning like a managed FTP server in front of AWS storage.
>
> — *Netec S3, 23:52-24:11, 35:14-35:36*

> AWS DMS keeps the source database available/running during migration — called out as a key benefit; supports continuous replication to keep source and target in sync during a cutover window, works across different or the same database engines, and supports a full one-time load, ongoing change replication, or a combination of both.
>
> — *Netec S3, 1:51:11-1:53:41*

> AWS Schema Conversion Tool (SCT) named directly: converts schemas between different/heterogeneous database engines (e.g. on-prem SQL Server → RDS) as part of a migration.
>
> — *Netec S3, 1:54:13-1:55:14*
