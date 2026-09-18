# Redshift & Athena

## Amazon Redshift

- Columnar data warehouse for petabyte-scale queries; based on Postgres but built for OLAP, not OLTP — columnar storage + parallel query engine gives ~10x the performance of row-based warehouses
- Has a SQL interface and integrates with QuickSight/Tableau; faster than Athena for joins/aggregations thanks to indexes
- **Cluster modes**: classic **Provisioned** (pick/reserve instance types in advance for cost savings) or **Serverless** (no capacity to manage)
- Cluster architecture (Provisioned): leader node (query planning/result aggregation) + compute nodes (execute queries, send results to the leader)
- Loading data: bulk/large inserts perform much better than row-by-row — load via the `COPY` command from S3 (`copy customer from 's3://mybucket/mydata' iam_role '...'`), or stream in through Kinesis Data Firehose
- Snapshots: point-in-time, incremental, stored internally in S3; automated every 8h/5GB/schedule (1-35 day retention) or manual (kept until deleted); can auto-copy snapshots (automated or manual) to another region for DR; some cluster configurations support Multi-AZ

### Redshift Spectrum

- Queries data sitting directly in **S3** using the Redshift cluster's SQL engine, without loading it into Redshift first
- Still requires an **active Redshift cluster** — that's what parses/plans the query and submits it; the cluster then fans the query out to many separate, AWS-managed **Spectrum nodes** (outside the cluster's own compute) which read/scan/filter the raw S3 data in parallel
- Results are sent back to the cluster to be joined/aggregated with any local Redshift tables if needed, then returned to the client
- Use case: skip the `COPY`-into-cluster step for data that's cheaper to leave in S3 (cold/infrequently-queried data, datasets too large to duplicate into the cluster), while still being able to `JOIN` it against regular Redshift tables in one query

> Exam-wording cue: Spectrum is Redshift's answer to Athena's query-in-place-on-S3 model, but callable from inside the warehouse and joinable with existing warehouse data. Key distinction from Athena — Spectrum still requires a running Redshift cluster; Athena is fully serverless with zero infrastructure.

## Amazon Athena

- Serverless query service to analyze data stored in S3; built on Presto, uses standard SQL; supports CSV/JSON/ORC/Avro/Parquet
- Pricing: $5.00 per TB of data scanned
- Commonly used with QuickSight for reporting/dashboards; use cases include BI/analytics/reporting and querying VPC Flow Logs, ELB logs, CloudTrail trails
- **Performance tips**: use columnar formats (Parquet/ORC — convert with Glue) to cut scan volume and cost, compress data (bzip2/gzip/lz4/snappy/zlib/zstd), partition S3 data by virtual columns (e.g. `s3://bucket/path/year=1991/month=1/day=1/`) for query pruning, and prefer larger files (>128MB) to minimize overhead
- **Federated Query** — run SQL across relational/non-relational/object/on-prem sources via Lambda-based Data Source Connectors (e.g. CloudWatch Logs, DynamoDB, RDS), writing results back to S3
- Native encryption support (client/server-side)

## Redshift vs. Athena

| | Redshift | Athena |
|---|---|---|
| **Model** | Data warehouse — cluster-based (Provisioned) or Serverless | Fully serverless — no infrastructure at all |
| **Data location** | Data is loaded/copied *into* Redshift storage (via `COPY` from S3, or streamed via Kinesis Firehose) | Queries data *in place* in S3 — nothing to load |
| **Pricing** | Pay for cluster/compute time (Provisioned) or Serverless compute-seconds, regardless of query volume | Pay per query — $5.00 per TB scanned |
| **Performance** | Faster for complex joins/aggregations — has indexes, a query optimizer, leader+compute node architecture built for OLAP | Slower for heavy joins; built on Presto, scans raw files each time (no persistent indexes) |
| **Best for** | Repeated, complex analytical workloads on a defined, curated dataset (BI dashboards hitting the same warehouse constantly) | Ad-hoc, infrequent queries directly on S3 data — logs (VPC Flow Logs, ELB, CloudTrail), data lakes, one-off analysis |
| **Bridge between them** | **Redshift Spectrum** — query S3 data directly from a Redshift cluster without loading it in, when you need Redshift's engine against data still sitting in S3 | — |

> Exam-wording cue: "data warehouse," "petabyte-scale," or "frequent complex queries" → Redshift. "Query S3 directly," "serverless," "ad-hoc," or "pay per query" → Athena. This is a direct callout in the deck itself: "analyze data in S3 using serverless SQL → use Athena."

## S3 Select

- Run SQL analysis directly on S3 objects without spinning up a data warehouse

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 421-570.
