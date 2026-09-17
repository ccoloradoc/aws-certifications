# DynamoDB

- Key/value store for high-throughput, millisecond-latency workloads
- Not compatible with relational data models
- Fully managed, multi-AZ replicated NoSQL database with transaction support; single-digit millisecond performance at massive scale (millions of requests/sec, trillions of rows, 100s of TB); IAM-integrated security; no maintenance/patching
- No native read replicas — use **DAX** (DynamoDB Accelerator) for caching
- Measures **RCUs** (read capacity units/sec) and **WCUs** (write capacity units/sec)
- **Capacity modes**: Provisioned (plan RCU/WCU ahead of time, optional auto-scaling via Application Auto Scaling, cheaper) vs. On-Demand (auto-scales with load, no planning, more expensive, best for unpredictable/spiky workloads)

## Table & Item Structure

- Tables have a Primary Key decided at creation; unlimited items (rows)
- Items have attributes (nullable, addable over time) up to 400KB total; supports scalar (String/Number/Binary/Boolean/Null), document (List/Map), and set (String/Number/Binary Set) types — schemas can evolve freely

## Best Practices

- Keep items < 400KB; use S3 with a pointer for larger data
- Separate frequently vs. infrequently accessed data into different tables
- Use separate tables for timestamp-based access patterns

## Global Tables & Streams

- **Global Tables** — active-active, multi-region: apps can read AND write in any participating region; requires DynamoDB Streams enabled as a prerequisite
- **DynamoDB Streams** — ordered log of item-level create/update/delete events, 24h retention, limited consumers, processed via Lambda triggers or the DynamoDB Streams Kinesis adapter; use cases: real-time reactions (welcome emails), analytics, populating derivative tables, cross-region replication. Kinesis Data Streams is the newer alternative for the same change-capture role: 1 year retention, many more consumers, and works with Lambda/Kinesis Data Analytics/Firehose/Glue Streaming ETL

## DAX (DynamoDB Accelerator)

- Fully managed in-memory cache in front of DynamoDB, microsecond reads, no application code changes needed (same API), default 5-minute TTL; solves read congestion/"hot key" problems

## TTL (Time-to-Live)

- Auto-deletes items past an expiry timestamp attribute; use cases: trimming stored data to only current items, regulatory data-retention limits, web session expiry

## Backups & S3 Integration

- Backups: continuous PITR (optional, up to 35 days, restore creates a new table) vs. on-demand full backups (kept until explicitly deleted, manageable via AWS Backup including cross-region copy) — neither affects live performance
- **Export to S3** — needs PITR enabled, covers any point in the last 35 days, doesn't consume read capacity, outputs DynamoDB JSON or ION (good for analysis/ETL/audit snapshots)
- **Import from S3** — accepts CSV/DynamoDB JSON/ION, doesn't consume write capacity, always creates a new table, and logs import errors to CloudWatch Logs

## To research

- Secondary indexes: Global Secondary Index (GSI) vs. Local Secondary Index (LSI)
- Transactions (`TransactWriteItems` / `TransactGetItems`)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 421-570.

### From Netec live training (to review)

> DynamoDB framed as the exam's primary NoSQL/non-relational focus service — a managed, table-based key-value database with automatic scaling and a flexible schema vs. a relational database's fixed schema. Table structure described with relational-adjacent vocabulary: a record is like a row, attributes are like flexible columns. Advantages: scalability, millisecond-order latency, automatic scaling, automatic replication across multiple AZs. IAM integration available, as with RDS. Use cases: mobile/web apps, e-commerce, online gaming (player profiles/scores/leaderboards queryable globally at low latency).
>
> — *Netec S3, 58:32-58:42, 1:19:41-1:20:28, 1:20:52-1:21:25, 1:21:25-1:22:05, 1:22:05-1:22:20, 1:22:38-1:23:13*

> Exam-pattern guidance: questions mentioning millions of requests per second, very low latency, or "serverless" (unless Aurora Serverless is explicitly named), or referencing key-value access, tend to point to DynamoDB.
>
> — *Netec S3, 1:23:27-1:24:21*

> Partition Key + Sort Key together form a Composite Primary Key, uniquely identifying an item and enabling efficient range/filter queries within a partition — e.g. partition key = user ID, sort key = order ID, enabling queries like "all orders for this user" or "orders within a specific range" rather than scanning the whole table. A key with only a partition key is called simple; partition key + sort key together is composite.
>
> — *Netec S3, 1:24:49-1:28:52*

> Capacity modes: On-Demand (AWS scales automatically, pay per request — best for unpredictable/spiky traffic) vs. Provisioned (fixed read/write capacity defined ahead of time — best for predictable traffic and cost optimization). Exam-wording heuristic: "unpredictable demand"/"sudden spikes"/"no capacity planning" → On-Demand; "predictable traffic"/"cost optimization"/"known workload" → Provisioned.
>
> — *Netec S3, 1:29:14-1:32:23, 1:31:45-1:32:15*

> Read consistency model: Eventually consistent (the table default) — replication across AZs can lag by milliseconds, so an immediate read after a write can return a stale value; cheaper, fine for logs/metrics/non-critical data. Strongly consistent reads (opt-in) guarantee the most recent write is returned immediately, consume more RCUs, recommended for financial/transactional data needing immediate accuracy. Exam-wording heuristic: "must reflect the latest write immediately"/"last write" → strongly consistent; "reduce cost"/tolerates staleness → eventually consistent (the default).
>
> — *Netec S3, 1:32:28-1:33:50, 1:35:48-1:37:29, 1:34:50-1:35:21, 1:36:10-1:36:26, 1:36:50-1:37:29*

> DynamoDB Global Tables: replicated automatically across multiple regions with no single primary region — every region can read AND write, changes sync automatically across all regions. Use cases: global low-latency gaming/e-commerce apps, disaster recovery with automatic failover across regions.
>
> — *Netec S3, 1:37:29-1:39:36*
