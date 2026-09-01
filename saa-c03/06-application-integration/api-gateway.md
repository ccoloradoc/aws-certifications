# Amazon API Gateway

- **Throttling limits**: server-side, per-method, per-client, account-level
- **API Caching**: configured per-stage, default TTL of 300 seconds

## Notes

<!-- Your own notes go here. -->

### From slides (pages 421-570)

- Manages versioning (v1/v2), environments (dev/test/prod), auth, API keys, throttling, request/response transformation & validation, SDK generation, and supports importing from Swagger/OpenAPI
- Integration types: Lambda (invoke a function — easiest way to expose a serverless REST API), HTTP (proxy an existing HTTP backend, e.g. on-prem or an ALB, adding auth/rate limiting/caching in front of it), and AWS Service (expose any AWS API directly, e.g. start a Step Functions execution or post to SQS, for public access + auth + rate control without writing glue code)
- Endpoint types: Edge-Optimized (default; routed through CloudFront edge locations for global clients, though the API itself still lives in one region), Regional (for same-region clients; can still be manually paired with CloudFront for more cache control), Private (VPC-only, reached via an interface VPC endpoint, access governed by a resource policy)
- Auth options: IAM roles (internal apps), Cognito (external/mobile user identity), or a custom authorizer (your own logic); for a custom domain, the ACM cert must be in us-east-1 for Edge-Optimized endpoints, or in the API's own region for Regional endpoints — plus a Route 53 CNAME/A-alias record
