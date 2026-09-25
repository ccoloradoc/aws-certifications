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
- Lambda@Edge vs. CloudFront Functions (see [cloudfront-and-global-accelerator.md](../02-networking/cloudfront-and-global-accelerator.md))
- Step Functions integration (see [orchestration.md](../06-application-integration/orchestration.md))
- Pricing model (requests + duration)

## Answers (from slides, pages 421-570)

- Pricing: first 1M requests/month free, then $0.20/million; first 400,000 GB-seconds of compute/month free (e.g. 400,000s at 1GB RAM, or 3,200,000s at 128MB), then $1.00 per 600,000 GB-seconds — usually very cheap
- Per-region limits: memory 128MB-10GB (1MB increments, more RAM also scales CPU/network), max execution time 900s (15 min), env vars 4KB, `/tmp` disk 512MB-10GB, default concurrency limit 1,000 (raisable via support ticket)
- Deployment limits: 50MB compressed zip, 250MB uncompressed (code+deps); container images are also supported (must implement the Lambda Runtime API) — prefer ECS/Fargate for arbitrary Docker images instead of forcing them into Lambda
- Concurrency & throttling: exceeding the concurrency limit throttles new invocations — synchronous callers get a 429 `ThrottleError`; asynchronous invocations auto-retry (backing off exponentially from 1s up to 5 min, for up to 6 hours) before landing in a DLQ; reserved concurrency caps/guarantees concurrency per function

> Exam-wording cue: the concurrency limit (1,000 by default) is a **pool shared across every function in the account/region**, not per-function — a burst of invocations from **any** trigger (SNS, S3 events, EventBridge, etc.) competes for that same shared pool, so one noisy function can throttle an unrelated one. SNS (and S3/EventBridge) invoke Lambda **asynchronously**, so throttled invocations there aren't dropped immediately — they auto-retry for up to 6 hours before hitting a DLQ, meaning symptoms show up as **delayed processing**, not outright failure. "Invocations from [any async trigger] are being delayed/dropped under high volume" → fix with **Reserved Concurrency** on the affected function (guarantees it a slice of the pool) or an account concurrency limit increase — not something fixable on the trigger's side.
- Cold starts: happen when a new execution environment initializes (loads code + runs init code outside the handler) — the first request on that instance is slower; **Provisioned Concurrency** pre-warms environments so cold starts never happen (can be scaled by Application Auto Scaling on a schedule or target utilization)
- **Lambda SnapStart** (Java/Python/.NET) — up to 10x faster starts at no extra cost by invoking from a pre-initialized, cached snapshot (memory+disk state) taken when you publish a new version
- VPC-attached Lambda: by default Lambda runs outside your VPC and can't reach RDS/ElastiCache/internal ELBs; attaching it requires specifying VPC/subnets/security groups, and Lambda creates an ENI in your subnet to reach those resources
- Lambda + RDS Proxy: putting RDS Proxy in front of a database avoids connection exhaustion from many concurrent Lambda invocations; the Lambda function must itself be VPC-attached since RDS Proxy is never public
- Lambda@Edge vs CloudFront Functions — see [cloudfront-and-global-accelerator.md](../02-networking/cloudfront-and-global-accelerator.md) for the full comparison
- Step Functions — see [orchestration.md](../06-application-integration/orchestration.md)

## Notes

<!-- Your own notes go here. -->

### From Netec live training (to review)

> Lambda framed via contrast with EC2: "serverless" doesn't mean no infrastructure exists, it means you don't manage/administer the servers — AWS runs the uploaded code and you pay only for execution time, not continuous uptime. Described as event-driven/reactive rather than continuously running.
>
> — *Netec S2, 2:37:16-2:38:15, 2:38:22-2:38:50*

> Execution limits restated live: up to 15 minutes max execution, up to 10GB memory.
>
> — *Netec S2, 2:39:41-2:39:56*

> Event sources named directly as examples: CloudWatch alarms/monitoring, error detection, a DynamoDB table query, a security-related event, an object uploaded to S3.
>
> — *Netec S2, 2:40:41-2:41:14*

> Use-case framing: backend processing for websites/mobile apps, data processing, AI/ML-adjacent workflows. Deployment options: write code directly (multiple languages) or deploy via container image.
>
> — *Netec S2, 2:41:28-2:41:46, 2:41:56-2:42:10*

> Integration example: a Lambda function connecting to an AI model (e.g. Amazon Bedrock) to process a user query and return a response, optionally authenticated via API Gateway or Cognito, integrated with a frontend built in Amplify.
>
> — *Netec S2, 2:42:36-2:43:22*

> Function configuration includes verifying/testing performance, monitoring via CloudWatch logs for errors, defining trigger origins, and defining permissions — explicitly tied back to IAM roles (a Lambda calling Bedrock needs a role permitting that specific access).
>
> — *Netec S2, 2:43:33-2:44:35*

> Lab-derived console UI mechanic: when creating a Lambda function, the "use a default execution role" option is offered first; choosing a specific pre-existing IAM role instead requires expanding a collapsed "Change default execution role" section — easy to miss.
>
> — *Netec S4, 3:46:50-3:49:07*
