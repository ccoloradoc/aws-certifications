# CloudFront and Global Accelerator

## Amazon CloudFront

- Distributes content from an origin (S3, EC2, ELB, Route 53, or an external origin)

### CloudFront Distribution

- A **distribution** is the CDN configuration you create in CloudFront — ties one or more origins to a set of edge behaviors (caching, access control, functions) and gets its own domain name (or a custom one via an alias)
- Content Delivery Network (CDN): improves read performance by caching content at the edge, close to users, across hundreds of Points of Presence (edge locations + regional edge caches) worldwide
- **Global service** — like IAM, Route 53, and WAF, a distribution isn't tied to a single AWS Region (unlike most other services, e.g. EC2, Lambda, Elastic Beanstalk)
- Built-in DDoS resilience from its edge footprint, with native integration into AWS Shield and AWS WAF for request filtering/blocking

### Origins

- CloudFront origin types: S3 bucket (secured via Origin Access Control — OAC, the newer replacement for OAI), VPC Origin (private ALB/NLB/EC2 in a private subnet, no internet exposure needed), or Custom HTTP origin (an S3 static website, or any public HTTP backend like a public ALB)

### CloudFront + S3

- S3 static website hosting lacks native HTTPS — front it with CloudFront for HTTPS
- **Origin Access Identity (OAI)** prevents direct access to the S3 bucket, forcing traffic through CloudFront (superseded by OAC — see Origins above)

> **Q:** You have a static website hosted on an S3 bucket. You have created a CloudFront Distribution that points to your S3 bucket to better serve your requests and improve performance. After a while, you noticed that users can still access your website directly from the S3 bucket. You want to enforce users to access the website only through CloudFront. How would you achieve that?
>
> **A:** Configure your CloudFront Distribution and create an Origin Access Control (OAC), then update your S3 Bucket Policy to only accept requests from your CloudFront Distribution.

- CloudFront vs. S3 Cross-Region Replication: CloudFront = global edge cache with a TTL (e.g. a day), best for static content needed everywhere; CRR = per-region setup, near-real-time updates, read-only, best for dynamic content needing low latency in a few specific regions

### Cache Management

- Cache Invalidations — force a full (`*`) or partial (`/images/*`) cache refresh instead of waiting for the TTL to expire after updating the origin

### Other Features

- **Field-level encryption** — extra encryption at the edge for sensitive fields (e.g., PII)
- Custom error page handling
- Streaming content support
- **Geo restriction** — whitelist/blacklist by country
- **Price class** configuration — choose which edge regions to use, trades cost for coverage
- **Access control** — signed URLs/cookies, optionally restricted by IP

### Edge Functions: CloudFront Functions vs. Lambda@Edge

- Both let you run code attached to a CloudFront distribution, close to users, fully serverless, pay-per-use — for CDN customization: security/privacy, dynamic content at the edge, SEO, origin routing, bot mitigation, real-time image transforms, A/B testing, auth, user tracking
- **CloudFront Functions** — lightweight JavaScript, sub-ms startup, millions of requests/sec, native to CloudFront; can only touch Viewer Request/Viewer Response; max 2MB memory, 10KB package, <1ms execution; no network/filesystem/body access; cheaper (1/6th the price) with a free tier — best for cache-key normalization, header manipulation, URL rewrites/redirects, token-based auth
- **Lambda@Edge** — Node.js or Python, thousands of requests/sec; can hook all 4 points (Viewer Request/Response + Origin Request/Response); 128MB-10GB memory, 1-50MB package, 5-10s execution; has network/filesystem/request-body access, so it can call other AWS services (e.g. via the SDK); authored in us-east-1 and replicated globally by CloudFront; no free tier, billed per request+duration

## AWS Global Accelerator

- Increases availability and performance for global applications
- Runs over the AWS global network (not the public internet)
- Directs traffic to the optimal endpoint across regions
- Provides 2 static anycast IP addresses (existing IPv4 /24 ranges can be migrated in)

### Mechanics

- Uses Anycast IP (2 static IPs shared across edge locations, client routed to the nearest one) vs. Unicast IP (one server, one IP); traffic enters at the nearest edge location and is carried over AWS's internal backbone to the app
- Works with Elastic IP, EC2, ALB, NLB (public or private)
- Performs health checks and fails over to a healthy region in under a minute — useful for DR
- Only 2 external IPs need whitelisting; integrates with AWS Shield for DDoS protection

## CloudFront vs. Global Accelerator

- Both ride the AWS global network + edge locations and integrate with Shield
- CloudFront serves content at the edge (great for cacheable + dynamic HTTP content)
- Global Accelerator proxies packets at the edge to your regional apps over TCP/UDP (better for non-HTTP cases like gaming/UDP, IoT/MQTT, VoIP, or HTTP cases needing static IPs / fast deterministic regional failover)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 1-30 and 271-570.
