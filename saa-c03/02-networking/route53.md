# Amazon Route 53

## Overview

- Highly available, scalable, and Authoritative (you control the records); also a Domain Registrar; the only AWS service with a 100% availability SLA; named for DNS port 53
- DNS terminology: Domain Registrar (Route 53, GoDaddy, …), Zone File (holds DNS records), Name Server (resolves queries), TLD (.com, .org…), SLD (amazon.com…)
- Domain Registrar ≠ DNS Service — you can register a domain with one registrar (e.g. GoDaddy) and still use Route 53 to manage its DNS records by pointing the registrar's NS records at Route 53's name servers

## Hosted Zones

- Container for a domain's records; Public (internet-routable) vs Private (routes within one or more VPCs); $0.50/month per hosted zone
- TTL — high TTL means less Route 53 traffic but possibly stale records; low TTL means more traffic/cost but fresher records; TTL is mandatory on every record except Alias records

## Routing Policies

Define how Route 53 *responds* to DNS queries — don't confuse this with load balancer routing, which actually routes traffic; DNS never routes traffic, it only answers queries
- **Simple** — typically routes to a single resource; can specify multiple values in one record, in which case the client picks one at random; with Alias enabled, only one AWS resource can be specified; cannot be associated with Health Checks
- **Weighted** — splits traffic by relative weight (a record's share = its weight ÷ sum of all weights); weights don't need to add up to 100; all records for a weighted set must share the same name and type; can be associated with Health Checks; set a record's weight to 0 to stop sending it traffic (if every record has weight 0, they're all returned equally); use cases: load balancing across regions, canary-testing a new app version
- **Latency-based** — routes to the resource/region with the lowest latency for the user, based on measured latency between users and AWS Regions (not physical distance — different from Geolocation)
- **Failover** — active/passive routing based on health checks
- **Geolocation** — route based on user location, specified by Continent, Country, or US State; if locations overlap, the most specific match wins; should always define a "Default" record to catch unmatched queries; can be associated with Health Checks; use cases: website localization, restricting content distribution, load balancing
- **Geoproximity** — route based on the geographic location of users *and* resources, with an optional bias to shift traffic: expand a resource's reach (bias 1 to 99) or shrink it (bias -1 to -99); resources can be AWS (specify region) or non-AWS (specify latitude/longitude); requires Route 53 Traffic Flow
- **Multi-Value Answer** — returns up to 8 healthy records per query; can be associated with Health Checks (only healthy resources are returned); not a substitute for a real ELB
- **IP-based** — maps CIDR blocks of client IPs to specific endpoints; use cases: optimize performance, reduce network costs (e.g. route an ISP's users to a specific endpoint)

## Record Types

### Alias Records

- Route 53-specific extension layered on top of a normal **A** (or AAAA) record — not a separate DNS record type, so it's fully DNS-spec-compliant and *can* be used at the zone apex
- Work with: ELBs, APIs, CloudFront, S3, Elastic Beanstalk, VPC endpoints
- Can resolve the zone apex (naked/root domain)
- Return one or more IP addresses, free of charge

### CNAME Records

- Redirect to any DNS record
- Cannot resolve apex/root domains
- Don't require Route 53

### Alias vs. CNAME (common exam question)

| | Alias | CNAME |
|---|---|---|
| Zone apex (e.g. `example.com`) | ✅ Works | ❌ Not allowed |
| Points to | AWS resources only (ELB, CloudFront, S3, API GW, Elastic Beanstalk, VPC endpoint, another record in the same hosted zone) | Any DNS record, AWS or not |
| Cost | Free | Standard query charges |
| TTL | Set automatically by AWS, not configurable | You set it |
| Requires Route 53 | Yes | No |

- Rule of thumb: pointing the **root/naked domain** at an AWS resource → must use an Alias record (it behaves like an A record under the hood, so the DNS spec allows it at the apex); a CNAME can never be used at the apex, AWS target or not — a subdomain pointing anywhere else can use either

### PTR Records

- Reverse DNS lookup (IP → domain name)

## Health Checks

- Monitor resource health
- Only healthy resources are returned in DNS responses
- ~15 global checkers, default healthy/unhealthy threshold of 3, default interval 30s (10s costs more); HTTP/HTTPS/TCP supported; only 2xx/3xx counted healthy; can also check for text within the first 5120 bytes of the response; only for public resources
- Endpoint is considered Healthy if **>18%** of checkers report it healthy — otherwise Unhealthy; you can choose which checker locations Route 53 uses
- Calculated Health Checks — combine up to 256 child health checks with AND/OR/NOT logic into one parent check (useful for maintenance windows without failing everything)
- For private resources/VPCs: Route 53 checkers can't reach private endpoints directly — instead create a CloudWatch Alarm on the resource and health-check the alarm itself

## Configurations

- **Active/Passive** — failover with manual fail-back potential
- **Active/Active** — multiple resources serving traffic, automatic failover
- **Combination** — multiple routing policies combined in a tree structure

## Hybrid DNS

- Route 53 Resolver answers queries for local EC2 names and Private Hosted Zone records by default
- Resolver Endpoints extend this: Inbound (lets your on-prem resolvers query AWS-side names) and Outbound (forwards VPC queries to your on-prem resolvers) — needed for on-prem connectivity via Direct Connect or VPN

## Notes

<!-- Your own notes go here. -->
