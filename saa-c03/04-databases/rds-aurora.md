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
