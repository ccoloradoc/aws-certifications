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
