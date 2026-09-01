# WAF, Shield & Firewall Manager

Not covered in the base cheat sheet — notable security gap around DDoS/application-layer protection.

## To research

- **AWS WAF** — layer 7 (HTTP) filtering, rule groups, rate-based rules, attaches to ALB/CloudFront/API Gateway
- **AWS Shield Standard** (free, automatic) vs. **Shield Advanced** (paid, enhanced DDoS protection + cost protection)
- **AWS Firewall Manager** — centrally manage WAF rules and Shield Advanced across multiple accounts/resources via Organizations
- **AWS Network Firewall** — VPC-level traffic filtering, stateful/stateless rules

## Answers (from slides, pages 571-720)

- **AWS WAF**: Layer 7 firewall, deployable on ALB, API Gateway, CloudFront, AppSync GraphQL, and Cognito User Pools. Web ACL rules can match IP sets (up to 10K IPs/rule), HTTP headers/body/URI (SQLi, XSS protection), size constraints, geo-match, or rate-based counting (DDoS mitigation); rule groups are reusable rule bundles; Web ACLs are regional except for CloudFront's global scope. WAF does **not** support NLB (Layer 4) — pair Global Accelerator (for a fixed IP) with WAF-on-ALB instead
- **Shield Standard** — free, automatic for every account, covers common layer 3/4 attacks (SYN/UDP floods, reflection attacks)
- **Shield Advanced** — $3,000/month/org, adds protection for EC2/ELB/CloudFront/Global Accelerator/Route 53, 24/7 DDoS Response Team (DRT) access, cost protection against usage spikes caused by an attack, and automatic layer-7 mitigation by generating/deploying WAF rules for you
- **AWS Firewall Manager** — centrally manages WAF rules, Shield Advanced, EC2/ALB/ENI security groups, AWS Network Firewall (VPC-level), and Route 53 Resolver DNS Firewall across every account in an Organization; policies are regional and auto-apply to new resources as they're created (compliance-friendly)
- **When to use which**: WAF alone for granular single-resource protection; add Firewall Manager when you need cross-account WAF rollout/automation for new resources; add Shield Advanced specifically for dedicated DDoS response support and advanced reporting if you're a frequent target
- **AWS Network Firewall** (from slides, pages 721-870): protects an entire VPC at Layer 3-7, in any direction (VPC-to-VPC, outbound to internet, inbound from internet, to/from Direct Connect/Site-to-Site VPN); built internally on the Gateway Load Balancer; centrally manageable across accounts via Firewall Manager. Supports 1,000s of rules: IP/port filtering, protocol blocking (e.g. block outbound SMB), stateful domain allowlists (only allow `*.mycorp.com`), regex pattern matching, allow/drop/alert actions, and active-flow intrusion-prevention inspection; logs rule matches to S3/CloudWatch Logs/Firehose
- **CloudHSM** (contrast with KMS): AWS provisions dedicated tamper-resistant hardware (FIPS 140-2 Level 3) and you manage your own keys entirely — single-tenant, no free tier, requires the CloudHSM client software, HA via multiple HSMs across AZs, integrates with KMS via a Custom Key Store; good fit for SSE-C-style "you hold the key" requirements or Redshift encryption
- **DDoS resiliency best practices**: use CloudFront + Global Accelerator + Route 53 at the edge to absorb/deflect attacks before they reach your infra; scale with ASG + ELB to absorb traffic surges; hide backend resources behind CloudFront/API Gateway/ELB rather than exposing them directly; use WAF rate-based rules + managed IP-reputation rule sets + CloudFront geo-blocking for application-layer filtering; enable Shield Advanced's automatic layer-7 mitigation for defense in depth

## Notes

<!-- Your own notes go here. -->
