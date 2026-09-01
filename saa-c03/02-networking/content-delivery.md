# Content Delivery

## Amazon CloudFront

- Distributes content from an origin (S3, EC2, ELB, Route 53, or an external origin)

### CloudFront + S3

- S3 static website hosting lacks native HTTPS — front it with CloudFront for HTTPS
- **Origin Access Identity (OAI)** prevents direct access to the S3 bucket, forcing traffic through CloudFront

### Other Features

- **Lambda@Edge** — run code at edge locations
- **Field-level encryption** — extra encryption at the edge for sensitive fields (e.g., PII)
- Custom error page handling
- Streaming content support
- **Geo restriction** — whitelist/blacklist by country
- **Price class** configuration — choose which edge regions to use, trades cost for coverage
- **Access control** — signed URLs/cookies, optionally restricted by IP

## AWS Global Accelerator

- Increases availability and performance for global applications
- Runs over the AWS global network (not the public internet)
- Directs traffic to the optimal endpoint across regions
- Provides 2 static anycast IP addresses (existing IPv4 /24 ranges can be migrated in)

## Notes

<!-- Your own notes go here. -->

### From slides (pages 271-420)

- CloudFront origin types: S3 bucket (secured via Origin Access Control — OAC, the newer replacement for OAI), VPC Origin (private ALB/NLB/EC2 in a private subnet, no internet exposure needed), or Custom HTTP origin (an S3 static website, or any public HTTP backend like a public ALB)
- CloudFront vs. S3 Cross-Region Replication: CloudFront = global edge cache with a TTL (e.g. a day), best for static content needed everywhere; CRR = per-region setup, near-real-time updates, read-only, best for dynamic content needing low latency in a few specific regions
- Cache Invalidations — force a full (`*`) or partial (`/images/*`) cache refresh instead of waiting for the TTL to expire after updating the origin
- Global Accelerator mechanics: uses Anycast IP (2 static IPs shared across edge locations, client routed to the nearest one) vs. Unicast IP (one server, one IP); traffic enters at the nearest edge location and is carried over AWS's internal backbone to the app
  - Works with Elastic IP, EC2, ALB, NLB (public or private)
  - Performs health checks and fails over to a healthy region in under a minute — useful for DR
  - Only 2 external IPs need whitelisting; integrates with AWS Shield for DDoS protection
- Global Accelerator vs. CloudFront: both ride the AWS global network + edge locations and integrate with Shield; CloudFront serves content at the edge (great for cacheable + dynamic HTTP content); Global Accelerator proxies packets at the edge to your regional apps over TCP/UDP (better for non-HTTP cases like gaming/UDP, IoT/MQTT, VoIP, or HTTP cases needing static IPs / fast deterministic regional failover)

### Edge Functions: CloudFront Functions vs. Lambda@Edge (pages 421-570)

- Both let you run code attached to a CloudFront distribution, close to users, fully serverless, pay-per-use — for CDN customization: security/privacy, dynamic content at the edge, SEO, origin routing, bot mitigation, real-time image transforms, A/B testing, auth, user tracking
- **CloudFront Functions** — lightweight JavaScript, sub-ms startup, millions of requests/sec, native to CloudFront; can only touch Viewer Request/Viewer Response; max 2MB memory, 10KB package, <1ms execution; no network/filesystem/body access; cheaper (1/6th the price) with a free tier — best for cache-key normalization, header manipulation, URL rewrites/redirects, token-based auth
- **Lambda@Edge** — Node.js or Python, thousands of requests/sec; can hook all 4 points (Viewer Request/Response + Origin Request/Response); 128MB-10GB memory, 1-50MB package, 5-10s execution; has network/filesystem/request-body access, so it can call other AWS services (e.g. via the SDK); authored in us-east-1 and replicated globally by CloudFront; no free tier, billed per request+duration
