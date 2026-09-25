# SQS & SNS

## Amazon SQS (Simple Queue Service)

- Durable, loosely-coupled messaging between services
- Pull-based (consumers poll for messages) — contrast with SNS's push model
- Consumers poll (up to 10 messages/call), process, then call `DeleteMessage`; can run many consumers in parallel (horizontally scalable), commonly behind an ASG or as a buffer in front of a database write

### Standard vs. FIFO

- **FIFO** — rigorous message ordering, deduplication; 300 msg/s (3,000 batched) throughput, exactly-once via Deduplication ID, ordered within a Message Group ID
- **Standard** — unlimited throughput/messages, best-effort ordering, at-least-once delivery (can duplicate); <10ms publish/receive latency

> Exam-wording cue: a scenario requiring messages **never be processed more than once** or **strict ordering** (e.g. financial transactions, sequential commands) → **FIFO** — deduplication (via Deduplication ID or content-based dedup) and ordering (via Message Group ID) are FIFO-only guarantees. **Standard** queues give **at-least-once** delivery, meaning the consumer application itself must tolerate/handle duplicate and out-of-order messages (e.g. by making processing idempotent) — Standard never prevents duplicates on its own.

### Polling

- **Short polling** (default) — checks a subset of servers, may return no messages even if some exist
- **Long polling** — consumer waits (1-20s, 20s preferred) for a message instead of returning empty immediately; reduces API call volume/cost vs. short polling; configurable at the queue or per-call (`WaitTimeSeconds`) level

### Batch Operations

- **SendMessageBatch / DeleteMessageBatch / ChangeMessageVisibilityBatch** — send, delete, or adjust visibility for up to **10 messages in a single API call** instead of one at a time; each batch call is billed/counted as a single request regardless of message count, reducing cost and increasing effective throughput
- This is the mechanism behind FIFO's "300 msg/s (3,000 batched)" figure above: the raw API call limit is 300 requests/sec, and batching 10 messages per call multiplies that to 3,000 messages/sec of actual throughput
- **Lambda consuming SQS** batches too, via its own separate settings: **BatchSize** (messages per invocation — up to 10 for FIFO, up to 10,000 for Standard when combined with a batching window), a **batching window** (how long to wait to fill a batch before invoking anyway), and **`ReportBatchItemFailures`** — lets the function report which *specific* messages in a batch failed, so only those are retried/sent to a DLQ instead of reprocessing the entire batch over one bad message

> Exam-wording cue: "reduce SQS costs/API calls" or "increase SQS throughput" → **batch API operations** (SendMessageBatch/DeleteMessageBatch), up to 10 messages per call. "Some messages in a Lambda-SQS batch fail and shouldn't cause the whole batch to be reprocessed" → **`ReportBatchItemFailures`**, not a bigger batch size or a separate queue.

### Security

- HTTPS for in-flight encryption, KMS for at-rest, optional client-side encryption
- **SQS Access Policies** (like S3 bucket policies) enable cross-account access and let other services (SNS, S3) write to the queue

### Other Details

- Max message size: 256KB (use S3 + a pointer for larger payloads)
- Max retention: 14 days (default 4 days)
- **Visibility Timeout** — default 30s; message stays invisible to other consumers until processing completes; becomes visible again if not deleted in time (risk of reprocessing); extend it with `ChangeMessageVisibility`; too high delays reprocessing after a crash, too low risks duplicate processing
- Use multiple queues for message prioritization

## Amazon SNS (Simple Notification Service)

- Fully managed push notification/pub-sub service
- Producer publishes once to a topic; broadcasts to multiple subscribers (SQS queues, Lambda, email, HTTP endpoints, etc.) — every subscriber receives every message (unless a filter policy is applied)
- Limits: up to 12,500,000 subscriptions/topic, 100,000 topics/account
- **Publish modes**: Topic Publish (SDK, for normal pub/sub) vs. Direct Publish (mobile push via platform application/endpoint — GCM, APNS, ADM)
- Security mirrors SQS: HTTPS in-flight, KMS at-rest, optional client-side encryption, and SNS Access Policies for cross-account/service-to-topic access

### Fan-out Pattern (SNS + SQS)

- One publish to SNS delivers to every subscribed SQS queue — fully decoupled, no data loss, each queue gets persistence/delayed processing/retries independently, works cross-region, and you can add subscriber queues over time (queue's access policy must allow the SNS topic to write)

### Message Filtering

- A JSON filter policy on a subscription limits which published messages it receives; a subscription with no filter policy gets everything

### SNS FIFO

- Same Message Group ID ordering + deduplication (by ID or content) as SQS FIFO
- Can fan out to both Standard and FIFO SQS queues; same throughput ceiling as SQS FIFO

## Amazon MQ

- Managed message broker (ActiveMQ or RabbitMQ) with both queue (~SQS) and topic (~SNS) semantics in one service
- For migrating on-prem brokers that use open protocols (MQTT, AMQP, STOMP, OpenWire, WSS) without re-architecting to SQS/SNS's proprietary APIs
- Runs on provisioned servers (not serverless like SQS/SNS), so it doesn't scale as elastically; supports Multi-AZ with failover

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 271-420.

> **Persistence**: SQS is durable — messages sit in the queue (up to 14 days retention) until a consumer polls and deletes them, so if no consumer is running, nothing is lost. SNS has no storage — it's push-based broadcast at publish time; if a subscriber isn't listening, that message is gone. This is exactly why the fan-out pattern (SNS → SQS) exists: the SQS queue behind each subscriber supplies the persistence/durability SNS itself lacks.

### From Netec live training (to review)

> SQS/SNS named directly as the decoupling mechanism for a failing downstream component (e.g. a payment service) not taking down the whole request pipeline — messages simply wait in the queue.
>
> — *Netec S4, 2:13:19-2:13:58*

> Concrete fan-out worked example (Lab 5): an image uploaded to S3 triggers an S3 event notification → SNS fans that out to two paths → two different Lambda functions each resize the image differently ("mobile" and "thumbnail") → each result is written back into a folder in the same S3 bucket → all Lambda execution activity is visible via CloudWatch Logs.
>
> — *Netec S4, 3:26:41-3:28:29*
