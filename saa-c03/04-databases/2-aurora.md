# Amazon Aurora

Aurora is one of the engine options under RDS, but different enough to warrant its own file. It shares RDS's Backups & Restore, Security, and Lambda/Event-Notification integration mechanics — see [rds.md](1-rds.md) for those; this file covers what's unique to Aurora.

## Amazon Aurora

- Proprietary, MySQL/Postgres-compatible (drivers work as-is); ~5x MySQL / ~3x Postgres performance claim; storage auto-grows in 10GB increments up to 256TB; ~20% more expensive than RDS
- Architecture: 6 copies of data across 3 AZs (4/6 needed to write, 3/6 to read), self-healing peer-to-peer replication, storage striped across 100s of volumes
- One writer instance + up to 15 read replicas; replica lag sub-10ms; automated master failover in <30s
- **Writer endpoint & Reader endpoint** — Aurora gives you two built-in cluster DNS endpoints instead of one:
  - **Writer endpoint** — always points at the current primary/writer instance; if a failover happens, this endpoint automatically repoints to the newly promoted writer, so the application doesn't need to change anything
  - **Reader endpoint** — load-balances read traffic across all available Aurora Replicas; adding/removing replicas is automatically reflected here
- **Aurora Auto Scaling** — automatically adds/removes Aurora Replicas in response to CloudWatch metrics (e.g. CPU utilization or connection count) and a target-tracking policy, up to a configured maximum; new replicas are automatically included in the Reader endpoint's load balancing
- **Custom Endpoints** — define a named subset of instances instead of using the default reader endpoint; lets you isolate specific replicas (which can be provisioned with a bigger instance size than the rest of the cluster) to absorb heavy/complex analytical queries without that load competing with regular read traffic on the other replicas
- **Aurora Database Cloning** — creates a new cluster from an existing one via copy-on-write (shares the original volume until writes diverge); much faster/cheaper than snapshot+restore; good for spinning up a staging DB from production
- **Aurora Machine Learning** — call SageMaker or Comprehend directly from SQL (fraud detection, sentiment analysis, recommendations) with no ML experience needed
- **Babelfish for Aurora PostgreSQL** — lets Aurora Postgres understand T-SQL so MS SQL Server apps can run against it with minimal code changes
- **Backtrack** — restore data to a point in time without restoring from backups

## Amazon Aurora Global Database

- Spans multiple regions: 1 primary (read/write) region + up to 10 secondary read-only regions (<1s replication lag, up to 16 replicas each)
- Promoting a secondary region for DR has RTO <1 minute
- **Aurora Serverless** auto-scales capacity, ideal for infrequent/unpredictable workloads, pay per second
  - **Exam scenario**: production runs on a full Aurora Cluster, and the dev team wants a scaled-down copy that can absorb occasional heavy workloads but sits unused most of the time — while minimizing cost. Fix: run the dev/test environment on **Aurora Serverless** — it auto-scales capacity up for the heavy workload and back down when idle, billed per-second, so an idle dev environment costs almost nothing, with no application changes needed since it's still Aurora-compatible

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> Aurora described as engine-compatible with MySQL and PostgreSQL but designed/optimized directly by AWS, separating compute from a distributed storage layer for efficiency/HA gains beyond standard Multi-AZ RDS. Noted that Aurora, like S3, leverages AZs under the hood automatically even when you don't explicitly pick one. Up to 15 read replicas confirmed, described as materially higher than "classic RDS." Aurora Serverless named as one of the deployment options.
>
> — *Netec S3, 1:14:51-1:16:04, 1:16:04-1:16:56, 1:17:39-1:18:03, 1:18:59-1:19:19*
