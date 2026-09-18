# Amazon API Gateway

Main scenario: acting as the "front door" for a serverless REST API — exposing one or more Lambda functions to clients (web/mobile apps, third parties) over HTTP without managing any servers. 

Typical flow: client → API Gateway → Lambda (Lambda integration) → response, with API Gateway handling auth, throttling, request validation/transformation, caching, versioning/staging, and SDK generation around that call. 

Manages versioning (v1/v2), environments (dev/test/prod), auth, API keys, throttling, request/response transformation & validation, SDK generation, and supports importing from Swagger/OpenAPI

## Throttling & Caching

- **Throttling limits**: server-side, per-method, per-client, account-level
- **API Caching**: configured per-stage, default TTL of 300 seconds

## Integration Types

- **Lambda** — invoke a function; easiest way to expose a serverless REST API
- **HTTP** — proxy an existing HTTP backend (e.g. on-prem or an ALB), adding auth/rate limiting/caching in front of it
- **AWS Service** — expose any AWS API directly (e.g. start a Step Functions execution or post to SQS) for public access + auth + rate control without writing glue code

## Endpoint Types

- **Edge-Optimized** (default) — routed through CloudFront edge locations for global clients, though the API itself still lives in one region
- **Regional** — for same-region clients; can still be manually paired with CloudFront for more cache control
- **Private** — VPC-only, reached via an interface VPC endpoint, access governed by a resource policy

## Authentication & Custom Domains

- Auth options: IAM roles (internal apps), Cognito (external/mobile user identity), or a custom authorizer (your own logic)
- Custom domain: the ACM cert must be in us-east-1 for Edge-Optimized endpoints, or in the API's own region for Regional endpoints — plus a Route 53 CNAME/A-alias record

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 421-570.
