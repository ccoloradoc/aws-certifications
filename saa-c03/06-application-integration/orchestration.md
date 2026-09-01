# Orchestration

- **AWS Step Functions** — serverless workflow coordination between services (state machines)
- **AWS Simple Workflow Service (SWF)** — older task execution/coordination service, largely superseded by Step Functions

## Notes

<!-- Your own notes go here. -->

### From slides (pages 421-570)

- Step Functions build serverless visual workflows (state machines) to orchestrate Lambda and other services: sequences, parallel branches, conditionals, timeouts, error handling, and even human-approval steps
- Can integrate with EC2, ECS, on-premises servers, API Gateway, SQS, and more
- Use cases: order fulfillment, data processing pipelines, any multi-step workflow needing visibility/retries
