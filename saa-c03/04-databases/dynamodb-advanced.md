# DynamoDB — Advanced

Follow-up to [dynamodb.md](dynamodb.md) — covers features the base cheat sheet didn't mention.

## To research

- **DynamoDB Global Tables** — multi-region, multi-active replication
- **DynamoDB Streams** — change data capture, integration with Lambda
- **DAX** (DynamoDB Accelerator) — in-memory caching layer, when to use it
- Secondary indexes: Global Secondary Index (GSI) vs. Local Secondary Index (LSI)
- On-demand vs. provisioned capacity mode
- Transactions (`TransactWriteItems` / `TransactGetItems`)
- TTL (time-to-live) for automatic item expiration

## Answers (from slides, pages 421-570)

- **Global Tables** — active-active, multi-region: apps can read AND write in any participating region; requires DynamoDB Streams enabled as a prerequisite
- **DynamoDB Streams** — ordered log of item-level create/update/delete events, 24h retention, limited consumers, processed via Lambda triggers or the DynamoDB Streams Kinesis adapter; use cases: real-time reactions (welcome emails), analytics, populating derivative tables, cross-region replication. Kinesis Data Streams is the newer alternative for the same change-capture role: 1 year retention, many more consumers, and works with Lambda/Kinesis Data Analytics/Firehose/Glue Streaming ETL
- **DAX** — fully managed in-memory cache in front of DynamoDB, microsecond reads, no application code changes needed (same API), default 5-minute TTL; solves read congestion/"hot key" problems
- On-demand vs. provisioned — covered in dynamodb.md
- **TTL** — auto-deletes items past an expiry timestamp attribute; use cases: trimming stored data to only current items, regulatory data-retention limits, web session expiry
- Backups: continuous PITR (optional, up to 35 days, restore creates a new table) vs. on-demand full backups (kept until explicitly deleted, manageable via AWS Backup including cross-region copy) — neither affects live performance
- S3 integration: **Export to S3** needs PITR enabled, covers any point in the last 35 days, doesn't consume read capacity, outputs DynamoDB JSON or ION (good for analysis/ETL/audit snapshots); **Import from S3** accepts CSV/DynamoDB JSON/ION, doesn't consume write capacity, always creates a new table, and logs import errors to CloudWatch Logs

## Notes

<!-- Your own notes go here. -->
