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

## Notes

<!-- Your own notes go here. -->
