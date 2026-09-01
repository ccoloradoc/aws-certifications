# Redshift & Athena

## Amazon Redshift

- Columnar data warehouse for petabyte-scale queries
- Not serverless — requires running EC2-based cluster instances
- **Redshift Spectrum** queries S3 data directly with parallel processing, without loading into Redshift

### Update from slides (pages 421-570)

- Redshift now also offers a **Serverless** cluster mode alongside the classic Provisioned mode (which lets you pick/reserve instance types for cost savings) — the "not serverless" note above reflects the older model
- Based on Postgres but built for OLAP, not OLTP; columnar storage + parallel query engine gives ~10x the performance of row-based warehouses; has a SQL interface and integrates with QuickSight/Tableau; faster than Athena for joins/aggregations thanks to indexes
- Cluster architecture: leader node (query planning/result aggregation) + compute nodes (execute queries)
- Snapshots: point-in-time, incremental, stored in S3; automated every 8h/5GB/schedule (1-35 day retention) or manual (kept until deleted); can auto-copy snapshots to another region for DR; some cluster configurations support Multi-AZ
- Loading data: bulk/large inserts perform much better than row-by-row — load via the `COPY` command from S3, or stream in through Kinesis Data Firehose
- Redshift Spectrum requires an active Redshift cluster to submit the query, which then fans out to many Spectrum nodes to read S3 data directly

## Amazon Athena

- Serverless SQL queries directly on S3 data
- Inexpensive, pay-per-query option
- Native encryption support (client/server-side)

### Additional detail from slides (pages 421-570)

- Built on Presto; supports CSV/JSON/ORC/Avro/Parquet; priced at $5.00 per TB scanned; commonly paired with QuickSight for dashboards; also good for querying VPC Flow Logs, ELB logs, CloudTrail trails
- Performance tips: use columnar formats (Parquet/ORC — convert with Glue) to cut scan volume, compress data (bzip2/gzip/lz4/snappy/zlib/zstd), partition S3 data by virtual columns (e.g. `year=1991/month=1/day=1/`) for query pruning, and prefer larger files (>128MB) to reduce overhead
- Federated Query — run SQL across relational/non-relational/object/on-prem sources via Lambda-based Data Source Connectors (e.g. CloudWatch Logs, DynamoDB, RDS), writing results back to S3

## S3 Select

- Run SQL analysis directly on S3 objects without spinning up a data warehouse

## Notes

<!-- Your own notes go here. -->
