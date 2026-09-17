# SQS & SNS

## Amazon SQS (Simple Queue Service)

- Durable, loosely-coupled messaging between services
- Pull-based (consumers poll for messages) — contrast with SNS's push model
- Consumers poll (up to 10 messages/call), process, then call `DeleteMessage`; can run many consumers in parallel (horizontally scalable), commonly behind an ASG or as a buffer in front of a database write

### Standard vs. FIFO

- **FIFO** — rigorous message ordering, deduplication; 300 msg/s (3,000 batched) throughput, exactly-once via Deduplication ID, ordered within a Message Group ID
- **Standard** — unlimited throughput/messages, best-effort ordering, at-least-once delivery (can duplicate); <10ms publish/receive latency

### Polling

- **Short polling** (default) — checks a subset of servers, may return no messages even if some exist
- **Long polling** — consumer waits (1-20s, 20s preferred) for a message instead of returning empty immediately; reduces API call volume/cost vs. short polling; configurable at the queue or per-call (`WaitTimeSeconds`) level

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

### From Netec live training (to review)

> SQS/SNS named directly as the decoupling mechanism for a failing downstream component (e.g. a payment service) not taking down the whole request pipeline — messages simply wait in the queue.
>
> — *Netec S4, 2:13:19-2:13:58*

> Concrete fan-out worked example (Lab 5): an image uploaded to S3 triggers an S3 event notification → SNS fans that out to two paths → two different Lambda functions each resize the image differently ("mobile" and "thumbnail") → each result is written back into a folder in the same S3 bucket → all Lambda execution activity is visible via CloudWatch Logs.
>
> — *Netec S4, 3:26:41-3:28:29*
