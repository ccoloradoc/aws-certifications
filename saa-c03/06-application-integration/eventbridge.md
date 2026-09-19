# Amazon EventBridge

- (formerly CloudWatch Events) the service for reacting to events and automating responses, across AWS services and 3rd-party/SaaS applications, without custom integration code for native services
- **Schedule rules** — cron-like (e.g. trigger a Lambda function every hour)
- **Event Pattern rules** — react to a service doing something (e.g. an EC2 instance state-change notification), filter the event, then route it to a target
- Broad target list: Lambda, ECS Task, SQS, Step Functions, SSM, SNS, CodePipeline, CodeBuild, EC2 actions, Kinesis Data Streams, AWS Batch, and 3rd parties (Zendesk, Datadog)
- Event buses can be accessed cross-account via **resource-based policies** (`events:PutEvents`) — use case: aggregate all events from your AWS Organization into a single account or region
- Events can be archived (all or filtered, indefinitely or for a set period), with the ability to replay them later
- **Schema Registry** — EventBridge can analyze events on a bus and infer/version their schema, and generate code bindings so your application knows in advance how event data is structured
- **Security** — a rule invoking a target needs permission on it: a resource-based policy (Lambda, SNS, SQS, S3, API Gateway) or an IAM role (EC2 Auto Scaling, SSM Run Command, ECS task)

> Exam-wording cue: compare against [sqs-sns.md](sqs-sns.md) — SQS/SNS move messages between producers and consumers; EventBridge reacts to *events* (state changes, schedules) and routes them to targets, with native filtering/schema support that SNS's plain pub/sub doesn't have.

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 571-720.

### From Netec live training (to review)

> Amazon EventBridge introduced as the service for reacting to events and automating responses across integrated AWS services and external/third-party applications, without needing custom integration code for native services — including forwarding AWS events to an external security event-management system. Components: event bus (default + custom buses), rules (filter events by type/origin), targets (SNS, Lambda, another AWS service). Worked example: creating an EC2 instance is itself an event EventBridge can detect and react to (trigger Lambda, send an alert, kick off a workflow) — framed as useful for automating security corrections. Also supports custom events from your own applications/SaaS integrations.
>
> — *Netec S3, 3:12:30-3:13:28, 3:13:35-3:14:23, 3:14:28-3:15:30, 3:15:36-3:16:15, 3:16:25-3:16:50, 3:17:05-3:17:22, 3:19:06-3:19:28*
