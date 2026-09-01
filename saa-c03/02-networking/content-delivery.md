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
