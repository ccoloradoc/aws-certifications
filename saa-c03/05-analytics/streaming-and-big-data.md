# Streaming & Big Data

## Amazon Kinesis

- Real-time data ingestion (e.g., IoT sensors)
- Components: **shards** → **data records** (each with a sequence number) → **partition keys**

## Other Big Data / Analytics Services

- **Amazon EMR** — Hadoop-based processing on EC2/S3
- **Amazon MSK** — managed Apache Kafka streaming service
- **Amazon QuickSight** — business intelligence dashboards
- **Amazon Elasticsearch Service** — operational analytics and search
- **CloudSearch** — managed site search engine

### Expanded detail (from slides, pages 421-570)

- **EMR** ("Elastic MapReduce") — managed Hadoop clusters (100s of EC2 instances) bundled with Spark/HBase/Presto/Flink, auto-scaling, Spot-integrated. Node types: Master (long-running, coordinates), Core (long-running, runs tasks + stores data), Task (optional, usually Spot, compute only). Purchasing: On-Demand (reliable), Reserved (1yr+ savings, used automatically when available), Spot (cheapest, less reliable). Clusters can be long-running or transient
- **Amazon QuickSight** — serverless, ML-powered BI dashboards; scales automatically, embeddable, per-session pricing; integrates with RDS/Aurora/Athena/Redshift/S3; uses the in-memory **SPICE** engine when data is imported; Enterprise edition adds column-level security; Users/Groups exist only inside QuickSight, not IAM; a published **dashboard** is a read-only, shareable snapshot of an analysis (viewers can see the underlying data)
- **Amazon Elasticsearch Service** is now branded **Amazon OpenSearch Service** — lets you search any field (including partial matches), unlike DynamoDB's key/index-only queries; managed or serverless cluster modes; no native SQL (plugin-enabled); ingests from Kinesis Data Firehose, IoT, CloudWatch Logs; secured via Cognito/IAM/KMS/TLS; ships with OpenSearch Dashboards for visualization
- **AWS Glue** — fully serverless, managed ETL; converts data to Parquet/ORC; **Glue Data Catalog** indexes your datasets; also includes Job Bookmarks (skip already-processed data), DataBrew (no-code data cleaning), Glue Studio (visual ETL job GUI), and Streaming ETL (Spark Structured Streaming, compatible with Kinesis/Kafka/MSK)
- **AWS Lake Formation** — built on top of Glue; automates data-lake setup (collect/cleanse/transform/catalog/dedupe via ML), combines structured + unstructured sources (S3, RDS, other DBs), and adds fine-grained (row/column-level) access control
- **Amazon Managed Service for Apache Flink** (formerly Kinesis Data Analytics for Apache Flink) — run Flink (Java/Scala/SQL) stream-processing apps on a managed, auto-scaling cluster with checkpoint/snapshot-based backups; note it does **not** read directly from Amazon Data Firehose
- **Amazon MSK** — fully managed Apache Kafka (brokers + Zookeeper managed for you), deployed in your VPC across up to 3 AZs for HA, auto-recovers from common Kafka failures, stores data on EBS for as long as you want; MSK Serverless removes capacity management entirely
- Kinesis Data Streams vs. MSK: both support 1MB messages (Kinesis configurable higher, e.g. 10MB) with TLS in-flight + KMS at-rest encryption; Kinesis uses shards (splittable/mergeable), Kafka uses topic partitions (can only be added, not removed)

## Notes

<!-- Your own notes go here. -->

### From slides (pages 271-420)

- Kinesis Data Streams: retention up to 365 days, consumers can replay data, data isn't deletable until it expires, records up to 1MiB, ordering guaranteed only within the same partition key, KMS at-rest + HTTPS in-flight encryption; use the Kinesis Producer Library (KPL) and Kinesis Client Library (KCL) for optimized producers/consumers
  - Provisioned mode: pick shard count, each shard = 1MB/s in (1,000 records/s) and 2MB/s out; scale shards manually; billed per shard-hour
  - On-demand mode: no capacity planning, default 4MB/s in (4,000 records/s), auto-scales off the last 30 days' observed peak; billed per stream-hour + data volume
- Amazon Data Firehose (formerly Kinesis Data Firehose): fully managed, serverless, auto-scaling near-real-time delivery into S3/Redshift/OpenSearch or 3rd parties (Splunk, MongoDB, Datadog, New Relic) or a custom HTTP endpoint; buffers by size/time; supports CSV/JSON/Parquet/Avro/raw text/binary with optional Parquet/ORC conversion, gzip/snappy compression, and Lambda-based transformation (e.g. CSV→JSON) — unlike Data Streams, it does **not** store data or support replay
- SQS vs SNS vs Kinesis: SQS = pull, deleted after consumption, any number of workers, ordering only on FIFO; SNS = push pub/sub, up to 12.5M subscribers/100K topics, undelivered data is lost, integrates with SQS for fan-out; Kinesis = built for real-time big-data/analytics/ETL with shard-level ordering, replay capability, and a configurable expiration window (standard consumers pull at 2MB/s/shard, enhanced fan-out consumers get a push model at 2MB/s/shard/consumer)
