# SQS & SNS

## Amazon SQS (Simple Queue Service)

- Durable, loosely-coupled messaging between services
- Pull-based (consumers poll for messages) — contrast with SNS's push model

### Standard vs. FIFO

- **FIFO** — rigorous message ordering, deduplication
- **Standard** — unlimited throughput, best-effort ordering

### Polling

- **Short polling** (default) — checks a subset of servers, may return no messages even if some exist
- **Long polling** — waits for messages to arrive, reduces empty responses and cost

### Other Details

- Max message size: 256KB (use S3 + a pointer for larger payloads)
- Max retention: 14 days
- **Visibility timeout** — message stays invisible to other consumers until processing completes
- Use multiple queues for message prioritization
- **Amazon MQ** — managed Apache ActiveMQ, an alternative when migrating from existing message brokers

## Amazon SNS (Simple Notification Service)

- Fully managed push notification/pub-sub service
- Broadcasts messages to multiple subscribers (SQS queues, Lambda, email, HTTP endpoints, etc.)

## Notes

<!-- Your own notes go here. -->

### From slides (pages 271-420)

- SQS Standard: unlimited throughput/messages, default retention 4 days (max 14), <10ms publish/receive latency, at-least-once delivery (can duplicate), best-effort ordering only
- Consumers poll (up to 10 messages/call), process, then call `DeleteMessage`; can run many consumers in parallel (horizontally scalable), commonly behind an ASG or as a buffer in front of a database write
- Security: HTTPS for in-flight encryption, KMS for at-rest, optional client-side encryption; SQS Access Policies (like S3 bucket policies) enable cross-account access and let other services (SNS, S3) write to the queue
- Visibility Timeout — default 30s; a message is invisible to other consumers after being polled, and becomes visible again if not deleted in time (risk of reprocessing); extend it with `ChangeMessageVisibility`; too high delays reprocessing after a crash, too low risks duplicate processing
- Long Polling — consumer waits (1-20s, 20s preferred) for a message instead of returning empty immediately; reduces API call volume/cost vs. short polling; configurable at the queue or per-call (`WaitTimeSeconds`) level
- FIFO queues: 300 msg/s (3,000 batched) throughput, exactly-once via Deduplication ID, ordered within a Message Group ID

## Amazon SNS — additional detail (from slides, pages 271-420)

- Producer publishes once to a topic; all subscribers receive every message (unless a filter policy is applied); up to 12,500,000 subscriptions/topic, 100,000 topics/account
- Publish modes: Topic Publish (SDK, for normal pub/sub) vs. Direct Publish (mobile push via platform application/endpoint — GCM, APNS, ADM)
- Security mirrors SQS: HTTPS in-flight, KMS at-rest, optional client-side encryption, and SNS Access Policies for cross-account/service-to-topic access
- **Fan-out pattern (SNS + SQS)**: one publish to SNS delivers to every subscribed SQS queue — fully decoupled, no data loss, each queue gets persistence/delayed processing/retries independently, works cross-region, and you can add subscriber queues over time (queue's access policy must allow the SNS topic to write)
- SNS FIFO topics: same Message Group ID ordering + deduplication (by ID or content) as SQS FIFO; can fan out to both Standard and FIFO SQS queues; same throughput ceiling as SQS FIFO
- Message Filtering — a JSON filter policy on a subscription limits which published messages it receives; a subscription with no filter policy gets everything

## Amazon MQ (from slides, pages 271-420)

- For migrating on-prem brokers that use open protocols (MQTT, AMQP, STOMP, OpenWire, WSS) without re-architecting to SQS/SNS's proprietary APIs
- Managed message broker (ActiveMQ or RabbitMQ) with both queue (~SQS) and topic (~SNS) semantics in one service
- Runs on provisioned servers (not serverless like SQS/SNS), so it doesn't scale as elastically; supports Multi-AZ with failover
