# RDS & Aurora

## RDS (Relational Database Service)

- Transactional (OLTP) database
- **Read Replicas** (async replication, for read scaling) vs. **Multi-AZ** (sync replication, for HA/failover)
- A managed database engine, not a data store abstraction
- Encryption: existing unencrypted DBs require snapshot → encrypt copy → restore
- **Point-in-Time Restore (PITR)** — restore to any point, typically up to 5 minutes ago

## Amazon Aurora Global Database

- Spans multiple regions
- **Aurora Replicas** for read scaling and standby
- **Aurora Serverless** auto-scales capacity, ideal for infrequent/unpredictable workloads

## Notes

<!-- Your own notes go here. -->

### From slides (pages 121-270)

- RDS supports Postgres, MySQL, MariaDB, Oracle, SQL Server, IBM DB2, and Aurora
- Managed-service benefits over self-hosting on EC2: automated provisioning/OS patching, PITR, monitoring dashboards, read replicas, Multi-AZ DR, maintenance windows, vertical + horizontal scaling — but no SSH access to the instance
- Storage Auto Scaling — scales storage automatically when free space is <10% of allocated for 5+ minutes (and 6+ hours since the last modification); requires setting a Maximum Storage Threshold; supports all engines
- Read Replicas — up to 15, within-AZ/cross-AZ/cross-region, async (eventually consistent) replication, promotable to standalone DBs, read-only (SELECT only); no network cost for same-region replication; use case: run reporting/analytics without hitting the production DB
- Multi-AZ — sync replication, single DNS name with automatic failover to standby, purely for HA/DR (not for scaling); enabling it on an existing DB is zero-downtime (snapshot → restore in new AZ → sync)
- RDS Custom (Oracle & SQL Server only) — grants OS/DB-level access (SSH/SSM) for custom configuration/patches, unlike standard RDS; can deactivate Automation Mode to customize (snapshot first)
- Aurora: proprietary, MySQL/Postgres-compatible, ~5x MySQL / ~3x Postgres performance claim, storage auto-grows in 10GB increments up to 256TB, ~20% more expensive than RDS
  - 6 copies of data across 3 AZs (4/6 needed to write, 3/6 to read), self-healing peer-to-peer replication, storage striped across 100s of volumes
  - One writer instance + up to 15 read replicas; replica lag sub-10ms; automated master failover in <30s
  - Custom Endpoints — define a named subset of instances (e.g. for analytical queries) instead of using the default reader endpoint
  - Aurora Serverless — auto-scales capacity based on usage, good for infrequent/unpredictable workloads, pay per second
  - Global Aurora — 1 primary (read/write) region + up to 10 secondary read-only regions (<1s replication lag, up to 16 replicas each); promoting a secondary region for DR has RTO <1 minute
  - Aurora Machine Learning — call SageMaker or Comprehend directly from SQL (fraud detection, sentiment analysis, recommendations) with no ML experience needed
  - Babelfish for Aurora PostgreSQL — lets Aurora Postgres understand T-SQL so MS SQL Server apps can run against it with minimal code changes
  - Backtrack — restore data to a point in time without restoring from backups
- Backups (RDS & Aurora): automated daily full backup + continuous transaction logs (5 min granularity) enable PITR; retention 1-35 days (RDS can disable with 0, Aurora cannot disable); manual snapshots kept indefinitely; a stopped RDS instance still bills for storage — snapshot & restore instead if stopping long-term
- Restoring a backup/snapshot always creates a **new** database; can also restore an on-premises MySQL/Aurora backup uploaded to S3
- Aurora Database Cloning — creates a new cluster from an existing one via copy-on-write (shares the original volume until writes diverge); much faster/cheaper than snapshot+restore; good for spinning up a staging DB from production
- Security: at-rest encryption via KMS (must be set at launch; unencrypted master ⇒ replicas can't be encrypted either — see the encryption note above for the fix); in-flight is TLS-ready by default; IAM Authentication as an alternative to username/password; Security Groups control network access; no SSH except RDS Custom; audit logs can stream to CloudWatch Logs

### More from slides (pages 421-570)

- Invoking Lambda from RDS/Aurora: supported for RDS PostgreSQL and Aurora MySQL only; the DB instance needs outbound network access to reach Lambda (public, NAT Gateway, or VPC Endpoint) plus permission to invoke it (Lambda resource-based policy + IAM policy)
- RDS Event Notifications: near-real-time (up to 5 min) notifications about the *instance itself* (created/stopped/started, etc. — not the data), covering DB instance/snapshot/parameter group/security group/RDS Proxy/custom engine version categories; delivered via SNS or subscribed to through EventBridge
