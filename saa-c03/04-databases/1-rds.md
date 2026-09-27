# Amazon RDS

## RDS (Relational Database Service)

- Transactional (OLTP) database; a managed database *engine*, not a data store abstraction
- Supported engines: Postgres, MySQL, MariaDB, Oracle, SQL Server, IBM DB2, and Aurora (see [aurora.md](2-aurora.md) for Aurora specifics)
- Managed-service benefits over self-hosting on EC2: automated provisioning/OS patching, PITR, monitoring dashboards, read replicas, Multi-AZ DR, maintenance windows, vertical + horizontal scaling — but no SSH access to the instance (except RDS Custom, below)
- **RDS Custom** (Oracle & SQL Server only) — grants OS/DB-level access (SSH/SSM) for custom configuration/patches, unlike standard RDS; can deactivate Automation Mode to customize (snapshot first)
  - Use case: third-party applications that require **privileged access** to the database host/OS — e.g. applying special patches or changing database software settings that standard (fully-managed, no-SSH) RDS wouldn't allow — with minimal infrastructure maintenance effort compared to self-hosting on EC2
  - **Must be explicitly configured for Multi-AZ** for high availability — it isn't automatic/default like it can be made to be on standard RDS

> Exam-wording cue: "need privileged OS/database-host access to support a third-party app's special configuration or patching requirements, on Oracle or SQL Server" → **RDS Custom** — standard RDS deliberately blocks this kind of access as part of being fully managed. If the scenario also emphasizes HA, remember Multi-AZ isn't a given here; it must be set up explicitly, unlike the "just toggle it" experience on standard RDS.

> Exam-wording cue: migrating a **SQL Server** workload while wanting to **enhance security** and **minimize database management/operational burden** → standard, fully-managed **Amazon RDS for SQL Server** — not **RDS Custom** (grants privileged OS access, which *increases* operational burden, the opposite of what's asked), and not **Aurora** (doesn't support SQL Server as an engine at all, so it isn't a valid option regardless of appeal). Reach for RDS Custom only when the scenario explicitly needs privileged host-level access for a third-party app's requirements.

## RDS Storage Auto Scaling

- Scales storage automatically when free space is **<10% of allocated storage for 5+ minutes**, and at least **6 hours** have passed since the last storage modification
- Requires setting a **Maximum Storage Threshold** as a ceiling
- Supports all RDS engines
- Avoids manual storage scaling — useful for unpredictable growth patterns
- Not relevant to Aurora, which grows storage automatically as part of its own distributed-storage architecture (see [aurora.md](2-aurora.md))

> Exam-wording cue: "database might run out of **storage**" + "**urgent**, **minimum development/administration effort**" → **enable RDS Storage Auto Scaling** on the existing instance — a single native setting, no migration, no cutover, no compatibility testing. Migrating to **Aurora** also solves storage growth long-term (its distributed storage scales automatically), but it's the wrong answer *here* since a migration itself requires real planning/testing/cutover effort — disproportionate to a narrow, already-solved-by-a-toggle problem, and directly conflicting with "urgent" + "minimum effort." Reach for Aurora when the question is instead about performance, read scaling, or multi-region reach — not when it's purely "storage might run out, fix it now with minimal effort."

## Read Replicas vs. Multi-AZ

- **Read Replicas** (async replication, for read scaling) — up to 15, within-AZ/cross-AZ/cross-region; eventually consistent; promotable to standalone DBs; read-only (SELECT only); no network cost for same-region replication; use case: run reporting/analytics without hitting the production DB
  - Each replica has its **own separate DNS endpoint** — the application must be explicitly configured/coded to send read traffic to it; there's no automatic request routing between primary and replica
  - **Data transfer charges**: replication traffic between the primary and a **same-region** read replica is **free** (no data transfer charge at all); a **cross-region** read replica incurs standard **inter-region data transfer pricing** on that replication traffic

> Exam-wording cue: "single-region app, latency complaints from one specific distant region (e.g. Aurora in us-east-1, complaints from Europe), select two" → create a **Read Replica in the nearby region** (e.g. eu-west-1), paired with a **regional web-tier fleet + Route 53 Latency routing** on the compute side (see [route53.md](../02-networking/route53.md)). This gives European users local, low-latency reads instead of round-tripping every query across the Atlantic — a more literal fix than accelerating the path to a single origin region (which is what Aurora Global Database / Global Accelerator would do instead, also valid but a different architecture).

> Exam-wording cue: "RDS read replica data transfer charges" → **free within the same region**, **charged across regions** (standard cross-region data transfer rate) — not "always free" or "always charged" regardless of location. Mirrors the general AWS pattern (same-region/same-AZ traffic is cheapest, cross-region/egress is where cost shows up).
- **Multi-AZ** (sync replication, for HA/failover) — single DNS name with automatic failover to a standby; purely for HA/DR, not scaling
  - The application always points at the **same** endpoint — failover silently repoints that DNS name to the new primary, with no application changes needed
- **Converting Single-AZ → Multi-AZ**: zero-downtime operation — no need to stop the DB, just click "Modify" on the instance. Internally:
  1. A snapshot of the Single-AZ instance is taken
  2. A new DB instance is restored from that snapshot into a different AZ
  3. Synchronization (sync replication) is established between the original and the new standby
- **Exam gotcha**: a Read Replica can *also* be configured as Multi-AZ — this combines read scaling with disaster recovery on the same replica, rather than being an either/or choice

> Exam-wording cue: **synchronous** (Multi-AZ) means a write is only acknowledged back to the app once *both* the primary and standby have confirmed it — that round-trip is what makes automatic, zero-data-loss failover possible, at the cost of added write latency; it's why Multi-AZ is framed around **"no data loss"** and **"automatic failover."** **Asynchronous** (Read Replica) means the primary acknowledges the write immediately and pushes it to the replica afterward — faster writes, but the replica can lag ("eventually consistent"), and a failure right after a write but before it replicates means that data is **not** on the replica; that lag is why promoting a Read Replica is always a **manual** action, never automatic. So: "no data loss"/"automatic failover" → Multi-AZ (sync); "scale reads"/"heavy read load"/"improve read performance" → Read Replica (async) — sync vs. async is the mechanical reason those two framings map the way they do.

## Backups & Restore

Applies to both RDS and Aurora.

- **Automated backups**: daily full backup + continuous transaction logs (5-minute granularity) enable **Point-in-Time Restore (PITR)** — restore to any point, typically up to 5 minutes ago; retention 1-35 days (RDS can disable with 0, Aurora cannot disable)
  - The daily full backup runs during a configured **backup window**; if it needs more time than the window allows, it simply continues past the window until finished — it doesn't get cut off. The backup window can't overlap the weekly maintenance window
  - Despite the "daily" backup, the underlying mechanism is continuous/incremental — that's what makes the **~5-minute-ago latest restorable time** possible, not just restoring to yesterday's snapshot
  - **No performance impact on the live database** while backup data is being written — this is what makes restoring from an automated backup a safe way to spin up a separate copy (e.g. a dev/staging database) without competing for I/O with production, unlike a manual full logical copy
- **Manual snapshots** are kept indefinitely
- A stopped RDS instance still bills for storage — snapshot & restore instead if stopping long-term

> Exam-wording cue: "**heavy read load** on RDS/Aurora" + "**a recurring full logical copy** of production (e.g. to populate a dev database) is causing latency/I/O contention" + "open to migrating engines" → migrate to **Aurora** for read-scaling/HA via Read Replicas, and replace the manual full-copy process with **restoring a new cluster from Aurora's automated backups** (or **Aurora Database Cloning** for a faster, copy-on-write alternative) — either way, the fix is to stop taking a manual logical copy and instead use Aurora's built-in, zero-production-impact copy mechanisms.
- Restoring a backup/snapshot always creates a **new** database; can also restore an on-premises MySQL/Aurora backup uploaded to S3
- Encrypting an existing unencrypted DB requires snapshot → encrypt the copy → restore from the encrypted copy

> Exam-wording cue: "how do I encrypt an **existing, already-running** unencrypted RDS instance" → always **snapshot → copy with encryption enabled (specify a KMS key during the copy) → restore a new instance from that encrypted copy → cut over**, never "enable encryption on the existing resource" — that option doesn't exist. Whether a DB instance's storage is encrypted is fixed at creation time, the same immutable-property pattern as a KMS key's single- vs. multi-Region setting; the only path is create-new-from-an-encrypted-copy, never an in-place conversion.

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

### From Netec live training (to review)

> Managed-service framing restated for databases: self-hosting a DB engine on EC2 means owning all scaling/HA/security concerns and the full operational burden; a managed service like RDS provides the config/update layer so you don't administer the DB engine at a low level — called a likely exam pattern (scenario mentions a managed service vs. self-managed instance → pick the managed service). Engine compatibility restated: MySQL, PostgreSQL, Oracle (and others). Deployment options: traditional provisioned instances vs. serverless (auto-provisions capacity).
>
> — *Netec S3, 58:50-1:00:25, 1:02:08-1:02:15, 1:02:23-1:02:50*

> Multi-AZ deployment explained mechanically: a primary instance plus a standby in another AZ; every write is confirmed only once both instances have the data (synchronous replication) — this is why failover can happen almost instantly with effectively no data loss (barring edge cases: a failure at the exact commit moment, an app-level bug, human error). If the primary fails, the standby is automatically promoted — a fast, simple mechanism enabled with a single toggle. Explicitly tied to RTO improvement.
>
> — *Netec S3, 1:05:09-1:07:18, 1:05:09-1:07:33, 1:05:32-1:05:50*

> Read Replicas explained mechanically: asynchronous replication (some lag exists), replicas are read-only, and — critically — promoting a read replica to standalone/primary is a manual action (or requires your own automation), unlike Multi-AZ's automatic failover. Purpose is explicitly read-scaling/performance, not high availability — can be cross-region, e.g. for reporting/analytics or reducing latency for distributed read traffic.
>
> — *Netec S3, 1:07:51-1:08:02, 1:11:19-1:11:52, 1:08:10-1:10:47*

> Direct comparison flagged as a common exam question: Multi-AZ = synchronous, automatic failover, HA/no-data-loss framing; Read Replica = asynchronous, manual promotion, read-scaling/performance framing — and they're not mutually exclusive (a read replica can itself be Multi-AZ). Exam-wording heuristic: "no data loss"/"automatic failover" → Multi-AZ; "scale reads"/"heavy read load"/"improve read performance" → Read Replica.
>
> — *Netec S3, 1:10:54-1:13:50, 1:12:38-1:13:18*

> RDS runs inside a VPC, protected via Security Groups, with a public-access toggle, and supports encryption via KMS with different keys per instance.
>
> — *Netec S3, 1:13:50-1:14:51*
