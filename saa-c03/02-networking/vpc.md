# VPC

A Virtual Private Cloud (VPC) is an isolated, private network hosted within a public cloud that lets you run resources safely.

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

AWS Global Infrastructure (Regions, Availability Zones, Edge Locations, global vs. region-scoped services, how to choose a Region) has moved to [00-basics/global-infrastructure.md](../00-basics/global-infrastructure.md) — it's foundational vocabulary the whole course assumes, not VPC-specific, even though it directly drives the "VPCs map to Regions, subnets map to AZs" rule in Core Concepts above.

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

### Security Groups fundamentals (from slides, pages 1-120 — previously missed)

- The fundamental unit of network security in AWS; control inbound/outbound traffic to/from an EC2 instance; contain only rules (no explicit deny); rules can reference an IP range or another security group
- Regulate: which ports are open, which IP ranges (IPv4/IPv6) are authorized, and inbound vs. outbound direction independently
- Can attach to multiple instances; locked to a region/VPC combination; live "outside" the instance — if a security group blocks traffic, the instance never sees it at all
- Good practice: keep a dedicated security group just for SSH access, separate from application traffic rules
- Troubleshooting rule of thumb: application times out → security group issue; application gives "connection refused" → the app itself has an error or isn't running (not a security group problem)
- Defaults: all inbound traffic blocked, all outbound traffic allowed
- Classic ports to know: 22 = SSH (Linux login), 21 = FTP, 22 = SFTP (file transfer over SSH), 80 = HTTP, 443 = HTTPS, 3389 = RDP (Windows login)

#### What can have a security group?

The unifying rule: a security group attaches to an **ENI**, so anything that provisions an ENI inside your VPC can have one. That covers far more than just EC2.

| Component | Has its own SG? | Notes |
|---|---|---|
| Elastic Network Interface (ENI) | Yes | This is the actual attachment point — every row below only has a SG *because* it provisions one of these; a standalone ENI can also be created and attached to an instance independently |
| EC2 instances | Yes | The classic case — attached via the instance's primary ENI |
| RDS / Aurora DB instances | Yes | Controls who can reach the DB port |
| ElastiCache clusters (Redis/Memcached) | Yes | — |
| Redshift clusters | Yes | — |
| Lambda functions | Yes, if VPC-attached | Default (non-VPC) Lambda has no ENI, so no SG; see [lambda.md](../01-compute/lambda.md) |
| Application Load Balancer (ALB) | Yes | — |
| Classic Load Balancer (CLB) | Yes | In EC2-VPC platform |
| Network Load Balancer (NLB) | No (traditionally) | NLB passes traffic through preserving source IP with no SG of its own — the **targets'** SGs do the filtering; note AWS added optional NLB-level SG support more recently, but the exam still tests the classic "NLB has no SG" fact |
| Gateway Load Balancer (GWLB) | No | Operates below the SG layer (Layer 3) |
| Interface VPC Endpoints (PrivateLink) | Yes | Backed by an ENI |
| Gateway VPC Endpoints (S3/DynamoDB) | No | Route-table target, not ENI-based |
| NAT Gateway | No | AWS-managed, no SG to configure (unlike a NAT Instance) |
| NAT Instance | Yes | It's just an EC2 instance under the hood |
| EFS Mount Targets | Yes | One per AZ, each with its own ENI |
| RDS Proxy | Yes | — |
| FSx file systems (Windows/Lustre/ONTAP/OpenZFS) | Yes | — |
| Amazon MSK brokers | Yes | — |
| DocumentDB / Neptune clusters | Yes | — |
| AWS Directory Service (Managed Microsoft AD) | Yes | — |
| Amazon WorkSpaces | Yes | — |
| SageMaker notebooks/endpoints | Yes, if VPC-attached | — |
| Client VPN endpoints | Yes | — |
| Internet Gateway | No | No ENI of its own |
| Elastic IP | No | An IP address, not a network interface itself (it's attached *to* something that has an ENI/SG) |

#### Elastic Network Interface (ENI) fundamentals (from slides, pages 61-90)

- A logical component representing a virtual network card inside a VPC — this is the actual object a security group attaches to, and the thing every row in both tables above is really being checked for
- Attributes an ENI can carry:
  - One primary private IPv4 address, plus one or more secondary private IPv4 addresses
  - Up to one Elastic IP (IPv4) per private IPv4 address it holds
  - One public IPv4 address
  - One or more security groups
  - A MAC address
- Can be created independently of any instance, then attached/detached and moved between EC2 instances on the fly — a common pattern for building failover (move the ENI, and its IPs/MAC/SG move with it, to a standby instance)
- Bound to a specific Availability Zone — an ENI created in one AZ cannot be attached to an instance in a different AZ

#### What can be assigned an ENI?

The flip side of the table above: whether a component provisions an ENI at all is the real gate — SG support just follows from it. Most rows match the SG table exactly, but a few components get an ENI **without** exposing a configurable SG on it, and a couple of "networking" constructs are NOT ENI-based despite sounding like they should be.

| Component | Gets an ENI? | Notes |
|---|---|---|
| EC2 instances | Yes | Primary ENI at launch; additional ENIs (secondary, different subnet/AZ within the same AZ as the instance) can be attached/detached on the fly |
| RDS / Aurora, ElastiCache, Redshift, RDS Proxy, FSx, MSK brokers, DocumentDB/Neptune, WorkSpaces | Yes | Same as the SG table — managed services provision an ENI per node/instance in your VPC/subnet |
| Lambda functions | Yes, if VPC-attached | The ENI is how a VPC-attached Lambda reaches RDS/ElastiCache/internal resources; see [lambda.md](../01-compute/lambda.md) |
| SageMaker notebooks/endpoints | Yes, if VPC-attached | — |
| Client VPN endpoints | Yes | Associates an ENI per subnet it's targeted at |
| ALB / CLB | Yes | One or more ENIs per AZ/subnet the load balancer spans |
| **NAT Gateway** | **Yes** | Gets an ENI with a private IP (and an associated Elastic IP) — but unlike a NAT Instance, AWS manages it and exposes **no configurable SG** on it (hence "No" in the SG table above despite having an ENI) |
| Interface VPC Endpoints (PrivateLink) | Yes | The ENI is the entry point that gets the private IP |
| EFS Mount Targets | Yes | One ENI per AZ |
| AWS Directory Service (Managed Microsoft AD) | Yes | Creates ENIs directly in your VPC subnets |
| **Route 53 Resolver Endpoints** (Inbound/Outbound) | Yes | Each endpoint provisions ENIs in your chosen subnets — see the Hybrid DNS notes below |
| **Transit Gateway VPC attachments** | Yes | One ENI per subnet/AZ the attachment uses |
| Network Load Balancer (NLB) | Yes | Gets an ENI per AZ (with a static IP) even though it traditionally has no SG of its own — traffic passes through to the targets |
| Gateway Load Balancer (GWLB) | Yes | ENI-backed like NLB, operating below the SG layer |
| Gateway VPC Endpoints (S3/DynamoDB) | **No** | Route-table target only — no ENI, no IP in your subnet |
| Internet Gateway / Egress-Only Internet Gateway | **No** | Horizontally scaled, redundant logical construct — not attached as an ENI |
| Virtual Private Gateway (VGW) | **No** | Terminates the VPN tunnel at the VPC edge, not as an ENI inside a subnet |
| VPC Peering connection | **No** | Pure route-table construct between two VPCs — no ENI on either side |
| Direct Connect Virtual Interface (VIF) | **No** | Operates at the DX connection level, outside any single VPC's ENIs |
| Elastic IP | **No** | An IP address that gets *attached to* an ENI, not an ENI itself |

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
