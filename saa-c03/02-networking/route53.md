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
