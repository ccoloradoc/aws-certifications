# Amazon API Gateway

Main scenario: acting as the "front door" for a serverless REST API — exposing one or more Lambda functions to clients (web/mobile apps, third parties) over HTTP without managing any servers. 

Typical flow: client → API Gateway → Lambda (Lambda integration) → response, with API Gateway handling auth, throttling, request validation/transformation, caching, versioning/staging, and SDK generation around that call. 

Manages versioning (v1/v2), environments (dev/test/prod), auth, API keys, throttling, request/response transformation & validation, SDK generation, and supports importing from Swagger/OpenAPI

## Throttling & Caching

- **Throttling limits**: server-side, per-method, per-client, account-level
- **API Caching**: configured per-stage, default TTL of 300 seconds

> Exam-wording cue: "**REST API on API Gateway + Lambda + Aurora**," workload is **read-heavy**, data **rarely changes**, staleness of up to **~24 hours is acceptable**, reduce database cost "**easily**" / "**with minimal changes**" → **enable API Gateway response caching** on the stage (a pure configuration toggle, no code changes to Lambda or Aurora). Cached responses are served directly from API Gateway for repeat identical requests, never reaching Lambda or Aurora at all. Caveat worth knowing: API Gateway's cache TTL has a **hard maximum of 3600 seconds (1 hour)** — it can't reach a full 24 hours, but 1 hour of staleness still falls comfortably within a 24-hour tolerance window, so it satisfies the requirement even without using the full budget. If a question instead needs to actually **use** the full stated staleness window (or caches data outside of what API Gateway fronts), that points to **ElastiCache** in front of Aurora instead — but that requires adding cache-aside logic to the Lambda function, a real code change, not just a config toggle, making it the *less* "minimal changes" option of the two.

> Exam-wording cue: "**rate limiting and throttling on a per-client basis**," "**usage quotas**," "**different limits to different API consumers**" → **API Gateway Usage Plans + API Keys**. Each client is issued an **API Key**, and a **Usage Plan** attached to that key defines both **throttling** (steady-state rate + burst limit) and a **quota** (e.g. requests per day/week/month) — different consumers get mapped to different Usage Plans (e.g. free tier vs. premium tier), giving per-client differentiation natively at the API layer, with no custom rate-limiting logic needed in application code.

## Integration Types

- **Lambda** — invoke a function; easiest way to expose a serverless REST API
- **HTTP** — proxy an existing HTTP backend (e.g. on-prem or an ALB), adding auth/rate limiting/caching in front of it
- **AWS Service** — expose any AWS API directly (e.g. start a Step Functions execution or post to SQS) for public access + auth + rate control without writing glue code

## Endpoint Types

- **Edge-Optimized** (default) — routed through CloudFront edge locations for global clients, though the API itself still lives in one region
- **Regional** — for same-region clients; can still be manually paired with CloudFront for more cache control
- **Private** — VPC-only, reached via an interface VPC endpoint, access governed by a resource policy

## Resource Policies & IP Restriction

- **Resource Policy** — an IAM-style JSON policy attached directly to a **REST API** (not a role/user, the API resource itself), controlling who can invoke it at the API Gateway layer, before any request reaches the backend integration
- Restrict by IP using the `aws:SourceIp` condition key with `IpAddress`/`NotIpAddress` operators — e.g. `Deny` on `NotIpAddress` for everything except a trusted CIDR range (an explicit allow-list)
- **REST APIs only** — HTTP APIs (the newer, cheaper API Gateway type) do not support resource policies at all
- **Security Groups don't apply here**: a public API Gateway endpoint has no ENI of its own, so there's nothing to attach a Security Group to. The only place an SG becomes relevant is a **Private API's Interface VPC Endpoint** ENI — and even then it only gates VPC-internal traffic, not arbitrary internet-source IPs

> Exam-wording cue: "restrict an API Gateway REST API to specific IP ranges, nothing more" → **Resource Policy with IpAddress/NotIpAddress** — simplest, native, no extra cost. "Restrict by IP **and** need broader web-attack protection (SQLi/XSS/rate-limiting/geo-block)" → **WAF Web ACL** instead. If the API is an **HTTP API** (not REST), resource policies aren't an option at all.

## Authentication & Custom Domains

- Auth options: IAM roles (internal apps), Cognito (external/mobile user identity), or a custom authorizer (your own logic)
- Custom domain: the ACM cert must be in us-east-1 for Edge-Optimized endpoints, or in the API's own region for Regional endpoints — plus a Route 53 CNAME/A-alias record

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 421-570.
