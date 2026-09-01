# Migration & Transfer Services

- **AWS DataSync** — copy data to S3, EFS, FSx, NFS, SMB, Snowcone
- **AWS DMS** (Database Migration Service) — copy/migrate databases with minimal downtime
- **AWS SCT** (Schema Conversion Tool) — converts database schemas between engine types
- **AWS Server Migration Service (SMS)** — migrate on-premises VMs to AWS (superseded by **AWS Application Migration Service / MGN**, see below)
- **AWS Import/Export** — physically ship storage devices (HDDs) to import large datasets into S3
- **AWS Resource Access Manager** — share resources (e.g., Transit Gateway) across accounts

## Notes

<!-- Your own notes go here. -->

### From slides (pages 271-420)

- **AWS Transfer Family** — fully managed FTP/FTPS/SFTP endpoint in front of S3 or EFS; scalable, reliable, Multi-AZ; pay per provisioned endpoint-hour + data transfer; stores user credentials itself or integrates with AD/LDAP/Okta/Cognito/custom auth; use cases: file sharing, public datasets, CRM/ERP integration
- **AWS DataSync** detail: needs an agent for on-prem/other-cloud → AWS transfers (NFS, SMB, HDFS, S3 API), no agent needed for AWS-to-AWS transfers; can target S3 (any class, including Glacier), EFS, or FSx (Windows/Lustre/ONTAP/OpenZFS); schedulable (hourly/daily/weekly); preserves file permissions/metadata (POSIX, SMB); one agent task can push up to 10Gbps (or a set bandwidth cap)

### More detail (from slides, pages 721-870)

- **AWS DMS**: source stays available during migration; supports homogeneous (Oracle→Oracle) and heterogeneous (SQL Server→Aurora) migrations, plus ongoing Change Data Capture (CDC) replication; you must run an EC2 instance to host the replication task; Multi-AZ deployment gives a synchronous standby replica (redundancy, no I/O freezes, fewer latency spikes)
  - Sources: on-prem/EC2-hosted Oracle/SQL Server/MySQL/MariaDB/PostgreSQL/MongoDB/SAP/DB2, Azure SQL Database, any RDS engine incl. Aurora, S3, DocumentDB
  - Targets: on-prem/EC2-hosted Oracle/SQL Server/MySQL/MariaDB/PostgreSQL/SAP, RDS, Redshift, DynamoDB, S3, OpenSearch, Kinesis Data Streams, Kafka, DocumentDB, Neptune, Redis, Babelfish
- **AWS SCT**: converts schemas between engines (e.g. SQL Server/Oracle → MySQL/PostgreSQL/Aurora for OLTP, Teradata/Oracle → Redshift for OLAP); **not needed** when the engine doesn't change (e.g. on-prem PostgreSQL → RDS PostgreSQL is still PostgreSQL, just a different platform)
- RDS/Aurora same-engine migration paths (MySQL and PostgreSQL both): (1) snapshot-and-restore into Aurora, or (2) create an Aurora Read Replica from the RDS source and promote it once replication lag hits zero (slower/costlier but near-zero downtime); for external (non-RDS) sources: dump to S3 and import (faster) or use DMS if both DBs are live
- **AWS Application Discovery Service** — inventories on-prem servers to plan a migration: Agentless mode (VM inventory/config/perf via AWS Agentless Discovery Connector) or Agent-based mode (system config/perf/processes/network connections via AWS Application Discovery Agent); results feed into **AWS Migration Hub**
- **AWS Application Migration Service (MGN)** — the current lift-and-shift/rehost service (AWS's evolution of CloudEndure Migration), replacing the older AWS Server Migration Service (SMS); converts physical/virtual/cloud servers to run natively on AWS with minimal downtime, broad OS/DB/platform support
- **VM Import/Export** — bring existing VMware/KVM/VirtualBox/Hyper-V VMs into EC2, or export EC2 instances back out (e.g. to build an on-prem DR repository); can also download an Amazon Linux 2 AMI as a portable VM image
- **VMware Cloud on AWS** — run actual VMware vSphere software on AWS infrastructure, for shops standardized on VMware that want to extend into AWS or build a DR strategy without changing tooling
- Large one-time transfer rule of thumb: 200TB over a 100Mbps internet/VPN link ≈ 185 days; over a 1Gbps Direct Connect ≈ 18.5 days (but DX setup itself often takes a month+); via Snowball ≈ about a week end-to-end, combinable with DMS for the cutover; for ongoing replication, prefer Site-to-Site VPN or DX paired with DMS/DataSync rather than one-off transfers
