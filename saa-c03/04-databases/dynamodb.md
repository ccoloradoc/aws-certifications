# DynamoDB

- Key/value store for high-throughput, millisecond-latency workloads
- Not compatible with relational data models
- No native read replicas — use **DAX** (DynamoDB Accelerator) for caching
- Measures **RCUs** (read capacity units/sec) and **WCUs** (write capacity units/sec)
- Auto-scaling via Application Auto Scaling

## Best Practices

- Keep items < 400KB; use S3 with a pointer for larger data
- Separate frequently vs. infrequently accessed data into different tables
- Use separate tables for timestamp-based access patterns

## Notes

<!-- Your own notes go here. -->

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

### From slides (pages 421-570)

- Fully managed, multi-AZ replicated NoSQL database with transaction support; single-digit millisecond performance at massive scale (millions of requests/sec, trillions of rows, 100s of TB); IAM-integrated security; no maintenance/patching
- Tables have a Primary Key decided at creation; unlimited items (rows); items have attributes (nullable, addable over time) up to 400KB total; supports scalar (String/Number/Binary/Boolean/Null), document (List/Map), and set (String/Number/Binary Set) types — schemas can evolve freely
- Capacity modes: Provisioned (plan RCU/WCU ahead of time, optional auto-scaling, cheaper) vs. On-Demand (auto-scales with load, no planning, more expensive, best for unpredictable/spiky workloads)
