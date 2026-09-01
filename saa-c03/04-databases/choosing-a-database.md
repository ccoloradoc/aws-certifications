# Choosing a Database & Specialized Databases

Not covered in the base cheat sheet — the slides frame database selection as a decision framework, then cover several specialized NoSQL databases with no other home in these notes.

## From slides (pages 421-570)

### Decision factors

- Workload shape: read-heavy / write-heavy / balanced? Throughput needs, and do they fluctuate?
- Data volume/growth, average object size, access pattern, retention needs
- Durability requirements and source-of-truth concerns
- Latency requirements and concurrent user count
- Data model: joins? structured vs. semi-structured? need a strong schema vs. flexibility?
- License costs — could a cloud-native DB (e.g. Aurora) reduce them?

### Database categories

- **RDBMS (SQL/OLTP)** — RDS, Aurora — best when you need joins
- **NoSQL** (no joins/SQL) — DynamoDB (JSON-like), ElastiCache (key/value), Neptune (graphs), DocumentDB (MongoDB-compatible), Keyspaces (Cassandra-compatible)
- **Object Store** — S3 (large objects), Glacier (backup/archive)
- **Data Warehouse (SQL analytics/BI)** — Redshift (OLAP), Athena, EMR
- **Search** — OpenSearch — free-text/unstructured search on JSON
- **Graph** — Neptune — relationships between data
- **Ledger** — Amazon Quantum Ledger Database (QLDB)
- **Time series** — Amazon Timestream

## Specialized Databases

### Amazon DocumentDB

- AWS's MongoDB-compatible database — same relationship to MongoDB that Aurora has to PostgreSQL/MySQL
- Stores/queries/indexes JSON data; fully managed, highly available across 3 AZs
- Storage auto-grows in 10GB increments; auto-scales to millions of requests/sec

### Amazon Neptune

- Fully managed graph database (e.g. social networks — users, posts, comments, likes)
- Highly available across 3 AZs, up to 15 read replicas, billions of relations queryable at millisecond latency
- Use cases: knowledge graphs (Wikipedia-style), fraud detection, recommendation engines, social networking
- Neptune Streams — real-time, strictly ordered, no-duplicate change feed over HTTP REST; use cases: change notifications, syncing to another store (S3, OpenSearch, ElastiCache), cross-region replication

### Amazon Keyspaces (for Apache Cassandra)

- Managed, serverless, Cassandra-compatible database using CQL (Cassandra Query Language)
- Tables replicated 3x across multiple AZs; auto-scales; single-digit ms latency at 1,000s of requests/sec
- Capacity: on-demand or provisioned with auto-scaling; encryption, backups, PITR up to 35 days
- Use cases: IoT device data, time-series data

### Amazon Timestream

- Fully managed, serverless time-series database; auto-scales; stores/analyzes trillions of events/day
- Claimed 1,000x faster and 1/10th the cost of relational DBs for this workload
- Tiered storage: recent data in memory, historical data cost-optimized; built-in time-series analytics functions; SQL-compatible; encrypted at rest and in transit
- Use cases: IoT apps, operational monitoring, real-time analytics

## Notes

<!-- Your own notes go here. -->
