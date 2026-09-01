# Amazon Route 53

## Routing Policies

- **Geolocation** — route based on user location
- **Geoproximity** — route based on resource proximity (with bias)
- **Weighted** — split traffic by percentage
- **Latency-based** — route to the lowest-latency region
- **Failover** — active/passive routing based on health checks

## Record Types

### Alias Records

- Route 53-specific DNS extension
- Work with: ELBs, APIs, CloudFront, S3, Elastic Beanstalk, VPC endpoints
- Can resolve the zone apex (naked/root domain)
- Return one or more IP addresses, free of charge

### CNAME Records

- Redirect to any DNS record
- Cannot resolve apex/root domains
- Don't require Route 53

### PTR Records

- Reverse DNS lookup (IP → domain name)

## Health Checks

- Monitor resource health
- Only healthy resources are returned in DNS responses

## Configurations

- **Active/Passive** — failover with manual fail-back potential
- **Active/Active** — multiple resources serving traffic, automatic failover
- **Combination** — multiple routing policies combined in a tree structure

## Notes

<!-- Your own notes go here. -->

### From slides (pages 121-270)

- DNS terminology: Domain Registrar (Route 53, GoDaddy, …), Zone File (holds DNS records), Name Server (resolves queries), TLD (.com, .org…), SLD (amazon.com…)
- Route 53 is highly available, scalable, and Authoritative (you control the records); it's also a Domain Registrar; the only AWS service with a 100% availability SLA; named for DNS port 53
- Domain Registrar ≠ DNS Service — you can register a domain with one registrar (e.g. GoDaddy) and still use Route 53 to manage its DNS records by pointing the registrar's NS records at Route 53's name servers
- Hosted Zones — container for a domain's records; Public (internet-routable) vs Private (routes within one or more VPCs); $0.50/month per hosted zone
- TTL — high TTL means less Route 53 traffic but possibly stale records; low TTL means more traffic/cost but fresher records; TTL is mandatory on every record except Alias records
- Additional routing policies not yet in the base notes:
  - Simple — routes to a single resource (or a random one of several values); can't attach Health Checks when Alias is enabled with a single resource
  - Multi-Value Answer — returns up to 8 healthy records per query; not a substitute for a real ELB
  - IP-based — maps CIDR blocks of client IPs to specific endpoints (e.g. route an ISP's users to a specific endpoint)
- Health Checks: ~15 global checkers, default healthy/unhealthy threshold of 3, default interval 30s (10s costs more); HTTP/HTTPS/TCP supported; only 2xx/3xx counted healthy; can also check for text within the first 5120 bytes of the response; only for public resources
  - Calculated Health Checks — combine up to 256 child health checks with AND/OR/NOT logic into one parent check (useful for maintenance windows without failing everything)
  - For private resources/VPCs: Route 53 checkers can't reach private endpoints directly — instead create a CloudWatch Alarm on the resource and health-check the alarm itself
- Hybrid DNS — Route 53 Resolver answers queries for local EC2 names and Private Hosted Zone records by default; Resolver Endpoints extend this: Inbound (lets your on-prem resolvers query AWS-side names) and Outbound (forwards VPC queries to your on-prem resolvers) — needed for on-prem connectivity via Direct Connect or VPN
