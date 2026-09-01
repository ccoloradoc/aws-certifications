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
