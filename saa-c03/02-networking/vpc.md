# VPC

## Core Concepts

- VPCs map to Regions; subnets map 1-to-1 with Availability Zones
- Route tables determine traffic direction
- Internet Gateways connect VPCs to the internet
- VPC Endpoints provide private connections to AWS services
- VPC Peering routes traffic directly between two VPCs
- Transit Gateway acts as a central hub for many VPCs (see [hybrid-connectivity.md](hybrid-connectivity.md))

## Subnet Types

### Public Subnets

- Route to an Internet Gateway
- Auto-assign public IP/ENI to EC2 instances
- Allow SSH (port 22) for administration

### Private Subnets

- Route outbound traffic through NAT devices
- **NAT Gateway** (AWS-managed) vs. **NAT Instance** (manually managed, can double as bastion host)
- Use a bastion host for SSH access into private subnets

## VPC Endpoints

### Interface Endpoints

- Private connections via AWS PrivateLink
- Apply to many services (API Gateway, CloudFormation, CloudWatch, S3, etc.)
- Backed by an ENI with a private IP, traffic directed via DNS

### Gateway Endpoints

- Direct private-IP access to S3 or DynamoDB only
- Traffic managed via route tables
- Protected by VPC endpoint policies

## Network Adapters

- **ENI** — basic elastic network interface
- **ENA** — enhanced networking adapter
- **EFA** — elastic fabric adapter, for HPC

## Security Groups vs. Network ACLs

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance | Subnet |
| Rules | Allow only; stateful | Allow/Deny; stateless |
| Evaluation | All rules together | In order |
| Default inbound | Deny (custom); Allow (default) | Deny |
| Default outbound | Allow all | Allow all (default VPC) |

## Notes

<!-- Your own notes go here. -->

### From slides (pages 1-120)

- Public IP — unique across the whole internet, easily geo-located
- Private IP — unique only within its own private network (two different private networks can reuse the same range); reaches the internet via NAT + internet gateway
- Elastic IP — a public IPv4 you own until you release it; attach to one instance at a time; masks instance failure by remapping to a healthy instance; capped at 5 per account (increasable)
  - Generally avoid overusing them — often a sign of poor architecture; prefer a DNS name or a Load Balancer instead of a static public IP

### CIDR & VPC sizing (from slides, pages 571-720)

- CIDR = a base IP + a subnet mask (`/0`-`/32`) defining a range; quick memo: `/32` = 1 IP, `/24` = last octet varies (256 IPs), `/16` = last 2 octets vary (65,536 IPs), `/8` = last 3 octets vary, `/0` = every IP
- Private IPv4 ranges (IANA): `10.0.0.0/8`, `172.16.0.0/12` (AWS's default VPC range), `192.168.0.0/16` (typical home networks) — everything else is public
- Per region: up to 5 VPCs (soft limit); each VPC can have up to 5 CIDR blocks, each sized between `/28` (16 IPs) and `/16` (65,536 IPs); VPC CIDRs must come from the private ranges above and shouldn't overlap other networks you'll connect to (e.g. corporate)
- Every subnet reserves 5 IPs AWS keeps for itself (network address, VPC router, DNS mapping, future use, broadcast) — e.g. for `10.0.0.0/24`: `.0`, `.1`, `.2`, `.3`, `.255` are unusable. Exam tip: needing 29 usable IPs requires at least a `/26` (64 - 5 = 59 ≥ 29), since a `/27` (32 - 5 = 27) falls short
- Internet Gateway: horizontally scaled, HA by design, created separately from the VPC, 1:1 with a VPC — but attaching it alone does nothing until route tables are also updated to point at it
- Default VPC exists in every new account; instances launched without a specified subnet land here, get a public IPv4 by default, and get both public and private DNS names

### Bastion Hosts & NAT (from slides, pages 571-720)

- Bastion Host: sits in a public subnet to let you SSH into private-subnet instances; its SG allows inbound 22 from a restricted CIDR (e.g. your corporate IP); the target instances' SG must allow the bastion's SG (or private IP) in turn
- NAT Instance (legacy, still testable): an EC2 instance in a public subnet with Source/Destination Check disabled and an Elastic IP attached; private-subnet route tables point traffic at it; bandwidth is capped by instance type, and HA requires your own ASG + failover scripting — you also manage its security groups (allow HTTP/S inbound from private subnets, SSH from home, HTTP/S outbound)
- NAT Gateway: AWS-managed, no admin overhead, 5Gbps auto-scaling to 100Gbps, lives in one AZ tied to an Elastic IP, needs an IGW (private subnet → NAT GW → IGW), and — unlike a NAT instance — needs no security groups and can't be used as a bastion. Resilient only within its own AZ, so deploy one NAT Gateway per AZ for full fault tolerance (no cross-AZ failover needed since an AZ outage removes the need for its own NAT anyway)
- Regional NAT Gateway (RNAT) — a newer HA NAT variant associated with the whole VPC rather than one AZ: shares route tables across AZs (no per-AZ NAT GW needed), doesn't require a public subnet to host it, and auto-expands to new AZs as resources appear there

### NACLs in depth (from slides, pages 721-870)

- One NACL per subnet (new subnets get the Default NACL, which allows everything in/out — don't modify it, create custom NACLs instead); newly created custom NACLs deny everything by default
- Rules are numbered 1-32766, lower number = higher precedence, first match wins; the final implicit rule (`*`) denies anything unmatched; AWS recommends numbering in increments of 100 for room to insert rules later
- Because NACLs are stateless, you must explicitly allow the **ephemeral port range** for return traffic (varies by OS — e.g. 32768-60999 on many Linux kernels, 49152-65535 on Windows/IANA) — this is the most common NACL gotcha
- Great for a quick, subnet-wide block of a specific malicious IP, which a security group can't easily do as cleanly

### VPC Peering, Endpoints & Flow Logs (from slides, pages 721-870)

- **VPC Peering**: private connection over AWS's network between two VPCs (even cross-account/cross-region) that must have non-overlapping CIDRs; NOT transitive — a direct peering connection is required between every pair of VPCs that need to talk; route tables in every involved subnet must be updated; you can reference a security group in a peered VPC (same-region only)
- **VPC Endpoints (PrivateLink)**: reach AWS services over the private AWS network instead of the public internet — horizontally scaled, redundant, removes the need for an IGW/NAT to reach AWS services
  - **Interface Endpoint** — an ENI with a private IP (needs a security group), supports most services, billed per-hour + per-GB
  - **Gateway Endpoint** — a route-table target (no security group), supports only S3 and DynamoDB, free — prefer this over an Interface Endpoint for S3/DynamoDB unless you need access from on-prem, another VPC, or another region, in which case Interface is required
  - Example: a VPC-attached Lambda calling DynamoDB should use a Gateway Endpoint (free, private) rather than routing out through a NAT Gateway + IGW to reach the public DynamoDB endpoint
- **VPC Flow Logs**: capture IP traffic at the VPC/subnet/ENI level (including AWS-managed ENIs behind ELB, RDS, ElastiCache, Redshift, WorkSpaces, NAT GW, Transit Gateway); destinations: S3, CloudWatch Logs, Kinesis Data Firehose; record fields include srcaddr/dstaddr, srcport/dstport, and an ACCEPT/REJECT action — query with Athena (S3) or Logs Insights (CloudWatch)
  - Troubleshooting with the ACTION field: inbound REJECT → NACL or SG; inbound ACCEPT + outbound REJECT → NACL; outbound REJECT → NACL or SG; outbound ACCEPT + inbound REJECT → NACL (a rejected *return* leg always points at the stateless NACL, never the stateful SG)
  - The IAM role behind Flow Logs needs `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents` to publish to CloudWatch Logs

### Traffic Mirroring, IPv6 & Egress-Only IGW (from slides, pages 721-870)

- **VPC Traffic Mirroring** — clones traffic from a source ENI to a target ENI or NLB (same VPC or peered VPC) for inspection by security appliances you manage; can capture everything or a filtered/truncated subset
- **IPv6**: every IPv6 address in AWS is public/internet-routable (no private range); IPv4 cannot be disabled on a VPC/subnet — IPv6 just adds dual-stack support, so if you can't launch an instance it's an IPv4-exhaustion problem, not IPv6 (fix: add another IPv4 CIDR)
- **Egress-Only Internet Gateway** — the IPv6 analog of a NAT Gateway: lets instances make outbound IPv6 connections while blocking the internet from initiating inbound IPv6 connections; still requires a route table update

### Networking cost tips (from slides, pages 721-870)

- Use private IPs over public IPs for both performance and cost savings; same-AZ traffic is cheapest (but sacrifices multi-AZ resilience)
- Egress (outbound) traffic is the expensive direction — ingress is typically free; keep traffic inside AWS where possible, and co-locate Direct Connect in the same region as your resources to cut egress costs
- A Gateway VPC Endpoint (free) vs. a NAT Gateway (hourly + per-GB) for reaching S3/DynamoDB is a common cost-optimization exam scenario — prefer the Gateway Endpoint
