# Streaming & Big Data

## Amazon Kinesis

- Real-time data ingestion (e.g., IoT sensors)
- Components: **shards** → **data records** (each with a sequence number) → **partition keys**; ordering guaranteed only within the same partition key
- Records up to 1MiB; KMS at-rest + HTTPS in-flight encryption
- Use the **Kinesis Producer Library (KPL)** and **Kinesis Client Library (KCL)** for optimized producers/consumers

### Kinesis Data Streams

- Retention up to 365 days; consumers can replay data; data isn't deletable until it expires
- **Provisioned mode** — pick shard count, each shard = 1MB/s in (1,000 records/s) and 2MB/s out; scale shards manually; billed per shard-hour
- **On-demand mode** — no capacity planning, default 4MB/s in (4,000 records/s), auto-scales off the last 30 days' observed peak; billed per stream-hour + data volume
- Standard consumers pull at 2MB/s/shard; **enhanced fan-out** consumers get a push model at 2MB/s/shard/consumer

### Amazon Data Firehose

- (formerly Kinesis Data Firehose) fully managed, serverless, auto-scaling near-real-time delivery into S3/Redshift/OpenSearch or 3rd parties (Splunk, MongoDB, Datadog, New Relic) or a custom HTTP endpoint
- Buffers by size/time; supports CSV/JSON/Parquet/Avro/raw text/binary with optional Parquet/ORC conversion, gzip/snappy compression, and Lambda-based transformation (e.g. CSV→JSON)
- Unlike Data Streams, does **not** store data or support replay

### Kinesis vs. Amazon MSK

- Both support 1MB messages (Kinesis configurable higher, e.g. 10MB) with TLS in-flight + KMS at-rest encryption
- Kinesis uses shards (splittable/mergeable); Kafka uses topic partitions (can only be added, not removed)

### Kinesis vs. SQS vs. SNS

- Kinesis is built for real-time big-data/analytics/ETL with shard-level ordering, replay capability, and a configurable expiration window
- See [sqs-sns.md](../06-application-integration/sqs-sns.md) for SQS (pull-based, deleted after consumption, ordering only on FIFO) and SNS (push pub/sub, fan-out) detail

## Amazon Managed Service for Apache Flink

- (formerly Kinesis Data Analytics for Apache Flink) run Flink (Java/Scala/SQL) stream-processing apps on a managed, auto-scaling cluster with checkpoint/snapshot-based backups
- Note: does **not** read directly from Amazon Data Firehose

## Amazon EMR (Elastic MapReduce)

- Managed Hadoop clusters (100s of EC2 instances) bundled with Spark/HBase/Presto/Flink, auto-scaling, Spot-integrated
- **Node types**: Master (long-running, coordinates), Core (long-running, runs tasks + stores data), Task (optional, usually Spot, compute only)
- **Purchasing options**: On-Demand (reliable), Reserved (1yr+ savings, used automatically when available), Spot (cheapest, less reliable)
- Clusters can be long-running or transient

## Amazon MSK (Managed Streaming for Apache Kafka)

- Fully managed Apache Kafka (brokers + Zookeeper managed for you), deployed in your VPC across up to 3 AZs for HA
- Auto-recovers from common Kafka failures; stores data on EBS for as long as you want
- **MSK Serverless** removes capacity management entirely

## Amazon QuickSight

- Serverless, ML-powered BI dashboards; scales automatically, embeddable, per-session pricing
- Integrates with RDS/Aurora/Athena/Redshift/S3
- Uses the in-memory **SPICE** engine when data is imported
- Enterprise edition adds column-level security
- Users/Groups exist only inside QuickSight, not IAM
- A published **dashboard** is a read-only, shareable snapshot of an analysis (viewers can see the underlying data)

## Amazon OpenSearch Service

- (formerly Amazon Elasticsearch Service) lets you search any field (including partial matches), unlike DynamoDB's key/index-only queries
- Managed or serverless cluster modes; no native SQL (plugin-enabled)
- Ingests from Kinesis Data Firehose, IoT, CloudWatch Logs
- Secured via Cognito/IAM/KMS/TLS
- Ships with OpenSearch Dashboards for visualization

## AWS Glue & Lake Formation

- **AWS Glue** — fully serverless, managed ETL; converts data to Parquet/ORC
  - **Glue Data Catalog** — indexes your datasets
  - **Job Bookmarks** — skip already-processed data
  - **DataBrew** — no-code data cleaning
  - **Glue Studio** — visual ETL job GUI
  - **Streaming ETL** — Spark Structured Streaming, compatible with Kinesis/Kafka/MSK
- **AWS Lake Formation** — built on top of Glue; automates data-lake setup (collect/cleanse/transform/catalog/dedupe via ML), combines structured + unstructured sources (S3, RDS, other DBs), and adds fine-grained (row/column-level) access control

## CloudSearch

- Managed site search engine

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 271-420 and 421-570.
