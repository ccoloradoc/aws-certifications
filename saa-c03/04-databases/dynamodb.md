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

### From slides (pages 421-570)

- Fully managed, multi-AZ replicated NoSQL database with transaction support; single-digit millisecond performance at massive scale (millions of requests/sec, trillions of rows, 100s of TB); IAM-integrated security; no maintenance/patching
- Tables have a Primary Key decided at creation; unlimited items (rows); items have attributes (nullable, addable over time) up to 400KB total; supports scalar (String/Number/Binary/Boolean/Null), document (List/Map), and set (String/Number/Binary Set) types — schemas can evolve freely
- Capacity modes: Provisioned (plan RCU/WCU ahead of time, optional auto-scaling, cheaper) vs. On-Demand (auto-scales with load, no planning, more expensive, best for unpredictable/spiky workloads)
