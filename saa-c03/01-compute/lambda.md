# AWS Lambda

Not covered in the base cheat sheet at all — serverless compute is core to SAA-C03, research this thoroughly.

## To research

- Execution model: event sources, triggers, synchronous vs. asynchronous invocation
- Concurrency: reserved vs. provisioned concurrency, throttling
- Cold starts and how to mitigate them
- Memory/CPU/timeout configuration and how they relate to cost
- Lambda layers
- VPC-attached Lambda (ENI cost, NAT requirements for internet access)
- Event source mappings (SQS, Kinesis, DynamoDB Streams)
- Lambda@Edge vs. CloudFront Functions (see [content-delivery.md](../02-networking/content-delivery.md))
- Step Functions integration (see [orchestration.md](../06-application-integration/orchestration.md))
- Pricing model (requests + duration)

## Answers (from slides, pages 421-570)

- Pricing: first 1M requests/month free, then $0.20/million; first 400,000 GB-seconds of compute/month free (e.g. 400,000s at 1GB RAM, or 3,200,000s at 128MB), then $1.00 per 600,000 GB-seconds — usually very cheap
- Per-region limits: memory 128MB-10GB (1MB increments, more RAM also scales CPU/network), max execution time 900s (15 min), env vars 4KB, `/tmp` disk 512MB-10GB, default concurrency limit 1,000 (raisable via support ticket)
- Deployment limits: 50MB compressed zip, 250MB uncompressed (code+deps); container images are also supported (must implement the Lambda Runtime API) — prefer ECS/Fargate for arbitrary Docker images instead of forcing them into Lambda
- Concurrency & throttling: exceeding the concurrency limit throttles new invocations — synchronous callers get a 429 `ThrottleError`; asynchronous invocations auto-retry (backing off exponentially from 1s up to 5 min, for up to 6 hours) before landing in a DLQ; reserved concurrency caps/guarantees concurrency per function
- Cold starts: happen when a new execution environment initializes (loads code + runs init code outside the handler) — the first request on that instance is slower; **Provisioned Concurrency** pre-warms environments so cold starts never happen (can be scaled by Application Auto Scaling on a schedule or target utilization)
- **Lambda SnapStart** (Java/Python/.NET) — up to 10x faster starts at no extra cost by invoking from a pre-initialized, cached snapshot (memory+disk state) taken when you publish a new version
- VPC-attached Lambda: by default Lambda runs outside your VPC and can't reach RDS/ElastiCache/internal ELBs; attaching it requires specifying VPC/subnets/security groups, and Lambda creates an ENI in your subnet to reach those resources
- Lambda + RDS Proxy: putting RDS Proxy in front of a database avoids connection exhaustion from many concurrent Lambda invocations; the Lambda function must itself be VPC-attached since RDS Proxy is never public
- Lambda@Edge vs CloudFront Functions — see [content-delivery.md](../02-networking/content-delivery.md) for the full comparison
- Step Functions — see [orchestration.md](../06-application-integration/orchestration.md)

## Notes

<!-- Your own notes go here. -->
