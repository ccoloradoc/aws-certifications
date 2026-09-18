# Streaming & Big Data

| Section | Summary |
|---|---|
| [Amazon Kinesis](#amazon-kinesis) | Real-time data ingestion; Data Streams (shards, replay) vs. Data Firehose (serverless delivery, no replay), plus comparisons to MSK and SQS/SNS |
| [Amazon Managed Service for Apache Flink](#amazon-managed-service-for-apache-flink) | Managed Flink stream-processing apps (Java/Scala/SQL) |
| [Amazon EMR](#amazon-emr-elastic-mapreduce) | Managed Hadoop/Spark clusters for big-data processing |
| [Amazon MSK](#amazon-msk-managed-streaming-for-apache-kafka) | Managed Apache Kafka (brokers + Zookeeper) |
| [Amazon QuickSight](#amazon-quicksight) | Serverless, ML-powered BI dashboards |
| [Amazon OpenSearch Service](#amazon-opensearch-service) | Managed search/log analytics (formerly Elasticsearch Service) |
| [AWS Glue & Lake Formation](#aws-glue--lake-formation) | Serverless ETL (Glue) and data-lake setup/governance (Lake Formation) |
| [CloudSearch](#cloudsearch) | Managed site search engine |

## Amazon Kinesis

- A family of services for collecting and processing real-time streaming data (e.g. IoT sensors, clickstreams, app/infra logs) — the umbrella term covers Data Streams, Data Firehose, and Managed Service for Apache Flink below
- Components: **shards** → **data records** (each with a sequence number) → **partition keys**; ordering guaranteed only for records sharing the same partition key (same key → same shard)
- KMS at-rest + HTTPS in-flight encryption across the family
- Use the **Kinesis Producer Library (KPL)** to write an optimized producer app, and the **Kinesis Client Library (KCL)** to write an optimized consumer app

### Kinesis Data Streams

- **Collects and stores** streaming data in real-time for consumers to process — this is the piece of the family that actually retains data, unlike Firehose
- Retention up to 365 days; consumers can replay/reprocess data any time within that window; data isn't deletable until it expires
- Record size up to 1MiB by default, configurable higher (e.g. 10MB) on request; typical use case is a high volume of small real-time records rather than few large ones
- **Provisioned mode** — pick shard count, each shard = 1MB/s in (1,000 records/s) and 2MB/s out; scale shards manually; billed per shard-hour
- **On-demand mode** — no capacity planning, default 4MB/s in (4,000 records/s), auto-scales off the last 30 days' observed peak; billed per stream-hour + data volume
- **Standard consumers** — pull model, 2MB/s per shard, shared across all consumers of that shard
- **Enhanced fan-out consumers** — push model, 2MB/s per shard *per consumer*, so each consumer gets its own dedicated throughput instead of sharing

### Amazon Data Firehose

- (formerly Kinesis Data Firehose) fully managed, serverless, auto-scaling **delivery** service — it loads streaming data *into* a destination rather than storing/holding it itself: S3, Redshift, OpenSearch, a 3rd party (Splunk, MongoDB, Datadog, New Relic), or a custom HTTP endpoint
- Near-real-time, not real-time — buffers by size or time before flushing to the destination
- Supports CSV/JSON/Parquet/Avro/raw text/binary; can convert to Parquet/ORC and compress with gzip/snappy in flight; supports custom transformation via Lambda (e.g. CSV→JSON)
- Unlike Data Streams: does **not** store data and does **not** support replay — once buffered data is flushed downstream, it's gone from Firehose

### Kinesis Data Streams vs. Amazon Data Firehose

| | Kinesis Data Streams | Amazon Data Firehose |
|---|---|---|
| **Role** | Streaming data collection — you write producer & consumer code | Delivery — fully managed, no consumer code to write |
| **Timing** | Real-time | Near-real-time (buffered by size/time) |
| **Capacity** | Provisioned or On-Demand mode | Fully automatic scaling |
| **Storage** | Up to 365 days | None — no data storage |
| **Replay** | Yes | No |

> Exam-wording cue: need to write custom consumer logic, replay data, or retain it for a window → Data Streams. Need to just land streaming data into S3/Redshift/OpenSearch/a 3rd party with zero infra to manage → Data Firehose.

### Kinesis vs. SQS vs. SNS

- Kinesis is built for real-time big-data/analytics/ETL with shard-level ordering, replay capability, and a configurable expiration window
- See [sqs-sns.md](../06-application-integration/sqs-sns.md) for SQS (pull-based, deleted after consumption, ordering only on FIFO) and SNS (push pub/sub, fan-out) detail

## Amazon Managed Service for Apache Flink

- (formerly Kinesis Data Analytics for Apache Flink) the **processing** layer of the pipeline — where Data Streams/Data Firehose collect and deliver, Flink is what you use to actually transform/analyze the data mid-stream (e.g. running aggregations, windowed computations, or anomaly detection in real time)
- **Apache Flink** — a framework (Java/Scala/SQL) for processing data streams; this service runs any Flink application for you on a managed cluster, with provisioned compute, parallel computation, and automatic scaling — no cluster ops to manage yourself
- **Backups** — implemented as checkpoints and snapshots, so a failed/restarted job resumes processing rather than reprocessing from scratch
- Reads from **Kinesis Data Streams** or **MSK/Kafka** as its streaming source
- Note: does **not** read directly from Amazon Data Firehose — Firehose only delivers to destinations, it doesn't expose a stream Flink can consume from

> Exam-wording cue: "real-time stream processing/transformation using SQL or a Flink app" → Managed Service for Apache Flink. If the question is only about collecting/delivering the stream (not processing it), that's Data Streams/Data Firehose instead.

## Amazon EMR (Elastic MapReduce)

- Managed Hadoop clusters (100s of EC2 instances) bundled with Spark/HBase/Presto/Flink, auto-scaling, Spot-integrated
- **Node types**: Master (long-running, coordinates), Core (long-running, runs tasks + stores data), Task (optional, usually Spot, compute only)
- **Purchasing options**: On-Demand (reliable), Reserved (1yr+ savings, used automatically when available), Spot (cheapest, less reliable)
- Clusters can be long-running or transient

## Amazon MSK (Managed Streaming for Apache Kafka)

- **Alternative to Kinesis** — same real-time streaming role, but running actual open-source Apache Kafka rather than an AWS-proprietary service; reach for MSK when migrating an existing Kafka workload, or when you need Kafka-specific APIs/ecosystem tooling (Kafka Connect, ksqlDB, existing Kafka producers/consumers) that Kinesis doesn't speak
- Fully managed Apache Kafka — MSK creates and manages the Kafka broker nodes and Zookeeper nodes for you; you still create/update/delete clusters yourself
- Deployed inside your VPC, Multi-AZ across up to 3 AZs for HA
- Automatic recovery from common Apache Kafka failures
- Data stored on EBS volumes for as long as you want — no automatic expiration window like Kinesis's retention setting
- **MSK Serverless** — run Kafka on MSK without managing capacity; automatically provisions and scales both compute and storage
- **Common consumers**: AWS Lambda, AWS Glue (Streaming ETL), Amazon Managed Service for Apache Flink (formerly Kinesis Data Analytics)

### Kinesis Data Streams vs. Amazon MSK

| | Kinesis Data Streams | Amazon MSK |
|---|---|---|
| **Message size** | 1MB default, configurable higher (e.g. 10MB) | 1MB message size limit |
| **Partitioning unit** | Shards — can be split & merged | Kafka topics with partitions — can only add partitions, never remove |
| **In-flight encryption** | TLS | PLAINTEXT or TLS |
| **At-rest encryption** | KMS | KMS |

> Exam-wording cue: "migrating an existing Kafka workload" or "need Kafka-specific APIs/ecosystem" → MSK. "Real-time streaming, AWS-native, no existing Kafka investment" → Kinesis. If the question emphasizes needing to **shrink capacity back down elastically**, that favors Kinesis (shards can split *and* merge); genuine Kafka/MSK partitions can only ever be added, never removed.

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

### Common Usage Patterns

- **CloudWatch Logs / VPC Flow Logs → Kinesis Data Firehose → OpenSearch → OpenSearch Dashboards** — log analytics / operational monitoring: ingest application, infrastructure, or VPC Flow Logs and search/visualize them in near-real-time; the most common exam scenario
- **DynamoDB or RDS (system of record) + DynamoDB Streams/CDC → OpenSearch** — full-text/free-text search: source data stays in DynamoDB/RDS, OpenSearch holds a synced searchable index for partial-match/free-text queries that DynamoDB's key/index-only lookups can't do
- **CloudTrail + GuardDuty + WAF logs → OpenSearch** — security/SIEM-style analytics: centralize and correlate security event data for threat hunting and anomaly-detection dashboards
- **IoT / clickstream events → Kinesis Data Firehose → OpenSearch** — clickstream/application analytics: ingest high-volume event streams for near-real-time aggregation and visualization
- **OpenSearch replacing SQL `LIKE`** — website/e-commerce search: typo-tolerant, relevance-ranked search with faceting/filtering that a bespoke SQL `LIKE` query can't scale or rank well

> Exam-wording cue: "search," "free-text search on unstructured/JSON data," or "log analytics dashboard" → OpenSearch, rather than DynamoDB/RDS/Redshift. If the question also names the ingestion path, look for **Kinesis Data Firehose** (streaming ingestion) or **DynamoDB Streams/CDC** (keeping a search index in sync).

## AWS Glue & Lake Formation

### AWS Glue

- Fully serverless, managed **ETL** (Extract, Transform, Load) service — the tool that prepares/cleans/transforms raw data *before* it's analyzed elsewhere (Athena, Redshift, EMR, QuickSight); converts data to columnar Parquet/ORC to speed up and cut the cost of those downstream queries
- **Glue Data Catalog** — central metadata store (schemas, table definitions, partitions) built by crawling your data sources; shared as the catalog backing Athena, Redshift Spectrum, and EMR, so a table cataloged once is queryable from all three without redefining it
- **Job Bookmarks** — tracks what's already been processed so re-running a job only picks up new data (prevents re-processing old data)
- **DataBrew** — no-code data cleaning/normalization using pre-built transformations, for users who don't want to write ETL scripts
- **Glue Studio** — visual GUI to build, run, and monitor ETL jobs without writing Spark code by hand
- **Streaming ETL** — built on Apache Spark Structured Streaming; consumes continuously from Kinesis Data Streams, Kafka, or MSK instead of a batch S3 source

> Exam-wording cue: "serverless ETL," "prepare/clean/transform data for analytics," or "convert data to Parquet" → Glue. If the question is instead about *organizing a whole S3-based repository* of that transformed data with catalog + fine-grained permissions, that's Lake Formation (below), which is built on top of Glue.

### AWS Lake Formation

- **Data lake** — a central repository holding all your data (structured + unstructured, from any source) for analytics purposes, as-is, at any scale
- Built on top of AWS Glue; a fully managed service that sets up a production-ready data lake in days rather than the months a manual build takes
- Automates the complex manual steps of building one: discover, cleanse, transform, and ingest data, plus de-duplication via ML Transforms
- **Source blueprints** — out-of-the-box ingestion from S3, RDS, and other relational/NoSQL databases
- **Fine-grained access control** — row- and column-level permissions, centrally managed and applied consistently across the analytics services querying the lake (e.g. Athena, Redshift Spectrum, EMR)

> Exam-wording cue: "single/central repository for all data (structured + unstructured) for analytics" → data lake / Lake Formation, not a data warehouse (Redshift) or a single database.

## CloudSearch

- Managed site search engine

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 271-420 and 421-570.
