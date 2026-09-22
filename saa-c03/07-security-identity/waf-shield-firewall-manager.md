# WAF, Shield & Firewall Manager

Not covered in the base cheat sheet — notable security gap around DDoS/application-layer protection.

## AWS WAF

- Protects web applications from common web exploits at **Layer 7** (HTTP; Layer 4 is TCP/UDP)
- Deploys on: Application Load Balancer, API Gateway, CloudFront, AppSync GraphQL API, Cognito User Pool
- **Web ACL rules** can match:
  - IP sets — up to 10,000 IP addresses per rule; use multiple rules for more
  - HTTP headers, HTTP body, or URI strings — protects from SQL injection and Cross-Site Scripting (XSS)
  - Size constraints and geo-match (block countries)
  - Rate-based rules (count occurrences of events) — for DDoS protection
- Web ACLs are **Regional**, except for CloudFront (global)
- A **rule group** is a reusable set of rules you can add to a web ACL

> Exam-wording cue: "application needs a **fixed/static IP** *and* **WAF protection**" → **Global Accelerator + ALB with WAF attached**. The tempting wrong answer is an NLB (which gives static IPs per AZ), but WAF can't attach to an NLB — it's Layer 4. Also: NLB alone can't do L7 filtering, so "filter HTTP requests" rules it out.

### Blocking an IP Address (Scenarios)

- The slides cover four "blocking an IP address" architectures — ALB, NLB, ALB + WAF, and ALB + CloudFront + WAF — but only as diagrams, so the converted text has no detail; see [load-balancing.md](../02-networking/load-balancing.md) and [cloudfront-and-global-accelerator.md](../02-networking/cloudfront-and-global-accelerator.md) for related notes

## AWS Shield

- **DDoS** (Distributed Denial of Service) — many requests at the same time
- **Shield Standard** — free, activated for every AWS customer; protects from SYN/UDP floods, reflection attacks and other layer 3/layer 4 attacks
- **Shield Advanced** — optional DDoS mitigation, **$3,000/month per organization**:
  - Protects against more sophisticated attacks on EC2, ELB, CloudFront, Global Accelerator, and Route 53
  - 24/7 access to the AWS DDoS Response Team (DRT)
  - Protection against higher fees during usage spikes caused by a DDoS attack
  - **Automatic application-layer DDoS mitigation** — automatically creates, evaluates and deploys WAF rules to mitigate layer 7 attacks

## AWS Firewall Manager

- Manages rules in **all accounts of an AWS Organization**, through a **security policy** (a common set of security rules)
- A policy can cover:
  - WAF rules (Application Load Balancer, API Gateway, CloudFront)
  - Shield Advanced (ALB, CLB, NLB, Elastic IP, CloudFront)
  - Security groups for EC2, Application Load Balancer and ENI resources in a VPC
  - AWS Network Firewall (VPC level)
  - Amazon Route 53 Resolver DNS Firewall
- Policies are created at the **Region** level
- Rules are applied to new resources as they're created, across all current and future accounts in the Organization — good for compliance

## WAF vs. Firewall Manager vs. Shield

- WAF, Shield and Firewall Manager are used together for comprehensive protection
- Define your Web ACL rules in **WAF**; for granular protection of your resources, WAF alone is the correct choice
- To use WAF across accounts, speed up WAF configuration, or automate protection of new resources → add **Firewall Manager** with WAF
- **Shield Advanced** adds features on top of WAF, such as dedicated support from the Shield Response Team (SRT) and advanced reporting; consider it if you're prone to frequent DDoS attacks

> Exam-wording cue: "protect one ALB/API/CloudFront from SQL injection, XSS, or block by IP/geo" → **WAF**; "DDoS response team, cost protection during attack spikes" → **Shield Advanced**; "free, automatic DDoS protection" → **Shield Standard**; "enforce WAF/security-group/Shield policies across all accounts in the Organization, including new resources" → **Firewall Manager**; "filter traffic for an entire VPC (Layer 3-7) including Direct Connect/VPN" → **Network Firewall**.

## AWS Network Firewall

- Protects your **entire VPC** from Layer 3 to Layer 7, inspecting traffic in any direction: VPC-to-VPC, outbound to internet, inbound from internet, and to/from Direct Connect and Site-to-Site VPN
- Internally uses the **AWS Gateway Load Balancer**
- Rules can be managed centrally across accounts by Firewall Manager, applied to many VPCs
- **Fine-grained controls**:
  - Supports 1,000s of rules
  - IP and port filtering (10,000s of IPs)
  - Protocol filtering (e.g. block SMB for outbound traffic)
  - Stateful domain list rule groups (e.g. only allow outbound traffic to `*.mycorp.com` or a third-party software repo)
  - General pattern matching with regex
  - Actions: allow, drop, or alert on matching traffic
  - Active flow inspection with intrusion-prevention capabilities (like Gateway Load Balancer, but all managed by AWS)
  - Rule-match logs go to S3, CloudWatch Logs, or Kinesis Data Firehose

## AWS Best Practices for DDoS Resiliency

The slides label the practices **BP1–BP7** (BP = Best Practice, from the AWS whitepaper "AWS Best Practices for DDoS Resiliency"); the numbers are diagram labels, not a priority order. The slides never define them in text, so the mapping below is inferred from where each label appears, and BP2 is the least certain. The practices are grouped by layer below.

| # | Best practice | Technology it maps to |
|---|---|---|
| **BP1** | Use edge locations for scale | CloudFront, Global Accelerator |
| **BP2** | Filter malicious web requests | WAF, Shield Advanced (automatic WAF rules) |
| **BP3** | Resolve DNS at the edge | Route 53 |
| **BP4** | Protect API endpoints | API Gateway |
| **BP5** | Filter traffic in the VPC | Security groups, network ACLs |
| **BP6** | Scale and distribute traffic | Elastic Load Balancing |
| **BP7** | Scale compute | EC2 Auto Scaling |

### Edge Location Mitigation (BP1, BP3)

- **CloudFront (BP1)** — web application delivery at the edge; protects from common attacks (SYN floods, UDP reflection)
- **Global Accelerator (BP1)** — access your application from the edge; integrates with Shield for DDoS protection; helpful if your backend isn't compatible with CloudFront
- **Route 53 (BP3)** — domain name resolution at the edge, with DDoS protection mechanisms

### Infrastructure Layer Defense (BP1, BP3, BP6, BP7)

- Use Global Accelerator, Route 53, CloudFront and Elastic Load Balancing together
- **EC2 with Auto Scaling (BP7)** — helps scale for sudden traffic surges, including flash crowds or a DDoS attack
- **Elastic Load Balancing (BP6)** — scales with traffic increases and distributes traffic across many EC2 instances, protecting EC2 against high traffic

### Application Layer Defense (BP1, BP2)

- CloudFront caches static content and serves it from edge locations, protecting your backend
- **WAF** on top of CloudFront and ALB filters and blocks requests based on request signatures
- WAF **rate-based rules** can automatically block the IPs of bad actors
- Use **managed rules** on WAF to block attacks based on IP reputation, or block anonymous IPs
- CloudFront can block specific geographies
- **Shield Advanced (BP1, BP2, BP6)** — automatic application-layer DDoS mitigation creates, evaluates and deploys WAF rules for you

### Attack Surface Reduction (BP1, BP4, BP5, BP6)

- **Obfuscate AWS resources** — use CloudFront, API Gateway and Elastic Load Balancing to hide backend resources (Lambda functions, EC2 instances)
- **Security groups and Network ACLs (BP5)** — filter traffic by specific IP at the subnet or ENI level; Elastic IPs are protected by Shield Advanced
- **Protect API endpoints (BP4)** — hide EC2/Lambda behind API Gateway; use Edge-Optimized mode, or CloudFront + Regional mode (more control for DDoS); WAF + API Gateway gives burst limits, header filtering, and API keys

## Related Services

- **GuardDuty** and **Inspector** are covered in [monitoring-audit.md](monitoring-audit.md)
- **CloudHSM** (dedicated encryption hardware, contrast with KMS) has its own file: [cloudhsm.md](cloudhsm.md)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 661-690 (WAF, Shield, Firewall Manager, DDoS best practices), 691-720 (attack surface reduction), 751-780 (Network Firewall), and 811-840 (blocking an IP address).
