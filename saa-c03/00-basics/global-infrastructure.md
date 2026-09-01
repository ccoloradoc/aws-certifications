# AWS Global Infrastructure

From slides, pages 1-30 — moved here from `02-networking/vpc.md` since this is foundational vocabulary the whole course (not just VPC/networking) assumes, even though Region/AZ placement does drive networking decisions directly.

## Building blocks

Four building blocks: **Regions**, **Availability Zones**, **Data Centers**, and **Edge Locations / Points of Presence** ([infrastructure.aws](https://infrastructure.aws/))

## Region

- A cluster of data centers in one geographic area, named like `us-east-1` or `eu-west-3`; most AWS services are region-scoped
- How to choose a Region:
  - Compliance/data-governance requirements — data never leaves a region without explicit permission
  - Proximity to customers — lower latency
  - Which services/features are actually available there — not every service launches everywhere at once
  - Pricing — varies region to region

## Availability Zone (AZ)

- One or more discrete data centers with redundant power, networking, and connectivity
- Each Region has multiple AZs (usually 3, minimum 3, maximum 6), named like `ap-southeast-2a`/`-2b`/`-2c`
- AZs are physically separate for disaster isolation, but interconnected with high-bandwidth, ultra-low-latency links — this is what makes Multi-AZ architectures both resilient *and* fast

## Edge Locations / Points of Presence

- 400+ locations (400+ Edge Locations + 10+ Regional Caches) across 90+ cities in 40+ countries
- Used to deliver content to end users with lower latency — what CloudFront/Global Accelerator ride on

## Global vs. Region-scoped services

- **Global**: IAM, Route 53, CloudFront, WAF
- **Region-scoped**: EC2, Elastic Beanstalk, Lambda, Rekognition, and most other services
- This distinction is why IAM users/roles work identically no matter which region you're viewing the console in, but an EC2 instance only exists in the region it was launched in

## See also

- [02-networking/vpc.md](../02-networking/vpc.md) — VPCs map 1:1 to a Region, and subnets map 1:1 to an AZ; this is where Region/AZ choice becomes a concrete networking decision

## Notes

<!-- Your own notes go here. -->
