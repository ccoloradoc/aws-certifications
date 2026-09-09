# Amazon RDS

## RDS (Relational Database Service)

- Transactional (OLTP) database; a managed database *engine*, not a data store abstraction
- Supported engines: Postgres, MySQL, MariaDB, Oracle, SQL Server, IBM DB2, and Aurora (see [aurora.md](2-aurora.md) for Aurora specifics)
- Managed-service benefits over self-hosting on EC2: automated provisioning/OS patching, PITR, monitoring dashboards, read replicas, Multi-AZ DR, maintenance windows, vertical + horizontal scaling — but no SSH access to the instance (except RDS Custom, below)
- **RDS Custom** (Oracle & SQL Server only) — grants OS/DB-level access (SSH/SSM) for custom configuration/patches, unlike standard RDS; can deactivate Automation Mode to customize (snapshot first)

## RDS Storage Auto Scaling

- Scales storage automatically when free space is **<10% of allocated storage for 5+ minutes**, and at least **6 hours** have passed since the last storage modification
- Requires setting a **Maximum Storage Threshold** as a ceiling
- Supports all RDS engines
- Avoids manual storage scaling — useful for unpredictable growth patterns
- Not relevant to Aurora, which grows storage automatically as part of its own distributed-storage architecture (see [aurora.md](2-aurora.md))

## Read Replicas vs. Multi-AZ

- **Read Replicas** (async replication, for read scaling) — up to 15, within-AZ/cross-AZ/cross-region; eventually consistent; promotable to standalone DBs; read-only (SELECT only); no network cost for same-region replication; use case: run reporting/analytics without hitting the production DB
  - Each replica has its **own separate DNS endpoint** — the application must be explicitly configured/coded to send read traffic to it; there's no automatic request routing between primary and replica
- **Multi-AZ** (sync replication, for HA/failover) — single DNS name with automatic failover to a standby; purely for HA/DR, not scaling
  - The application always points at the **same** endpoint — failover silently repoints that DNS name to the new primary, with no application changes needed
- **Converting Single-AZ → Multi-AZ**: zero-downtime operation — no need to stop the DB, just click "Modify" on the instance. Internally:
  1. A snapshot of the Single-AZ instance is taken
  2. A new DB instance is restored from that snapshot into a different AZ
  3. Synchronization (sync replication) is established between the original and the new standby
- **Exam gotcha**: a Read Replica can *also* be configured as Multi-AZ — this combines read scaling with disaster recovery on the same replica, rather than being an either/or choice

## Backups & Restore

Applies to both RDS and Aurora.

- **Automated backups**: daily full backup + continuous transaction logs (5-minute granularity) enable **Point-in-Time Restore (PITR)** — restore to any point, typically up to 5 minutes ago; retention 1-35 days (RDS can disable with 0, Aurora cannot disable)
- **Manual snapshots** are kept indefinitely
- A stopped RDS instance still bills for storage — snapshot & restore instead if stopping long-term
- Restoring a backup/snapshot always creates a **new** database; can also restore an on-premises MySQL/Aurora backup uploaded to S3
- Encrypting an existing unencrypted DB requires snapshot → encrypt the copy → restore from the encrypted copy

## Security

Applies to both RDS and Aurora.

- At-rest encryption via KMS — must be set at launch; an unencrypted master means its replicas can't be encrypted either (see Backups & Restore above for the fix)
- In-flight is TLS-ready by default
- IAM Authentication as an alternative to username/password
- Security Groups control network access
- No SSH except RDS Custom
- Audit logs can stream to CloudWatch Logs

## Integrations

Applies to both RDS and Aurora.

- **Invoking Lambda from RDS/Aurora** — supported for RDS PostgreSQL and Aurora MySQL only; the DB instance needs outbound network access to reach Lambda (public, NAT Gateway, or VPC Endpoint) plus permission to invoke it (Lambda resource-based policy + IAM policy)
- **RDS Event Notifications** — near-real-time (up to 5 min) notifications about the *instance itself* (created/stopped/started, etc. — not the data), covering DB instance/snapshot/parameter group/security group/RDS Proxy/custom engine version categories; delivered via SNS or subscribed to through EventBridge

## Notes

<!-- Your own notes go here. -->
