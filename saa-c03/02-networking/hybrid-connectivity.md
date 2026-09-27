# Hybrid Connectivity

## AWS Direct Connect (DX)

- Dedicated, private connection from your on-premises DC to an AWS Direct Connect location, extending to one or multiple VPCs; requires a VGW on your VPC; carries both public (S3) and private (EC2) traffic over the same connection; supports IPv4 and IPv6
- Setup time: often > 1 month, regardless of connection type
- Use cases: higher/cheaper bandwidth for large data sets, more consistent network performance for real-time feeds, hybrid environments
- Data in transit over DX is private but **not encrypted** — pair DX with a Site-to-Site VPN for IPsec encryption, at the cost of added complexity

### Connection Types

- **Dedicated** — 1Gbps-400Gbps, a physical port requested via an AWS Direct Connect Partner
- **Hosted** — 50Mbps-25Gbps, requested through a partner, capacity adjustable on demand

### Virtual Interfaces

- **Private VIF** — access to a VPC via private IP
- **Public VIF** — access to AWS public services via public IP
- **Transit VIF** — access to VPCs via Transit Gateway
- **Hosted VIF** — access shared across multiple accounts

### Direct Connect Gateway

- Needed when connecting one DX link to VPCs across multiple regions in the same account

> Exam-wording cue: Direct Connect Gateway's job is on-prem ↔ VPC — it bridges one DX connection to VPCs in **multiple regions**, but does **not** route between those VPCs. A question needing VPCs (or VPNs) to route **to each other**, in addition to reaching on-prem, points to a **Transit Gateway** instead (attached to the DX connection via a Transit VIF if DX is also involved) — Transit Gateway's core purpose is VPC-to-VPC routing, with DX/VPN attachment as an add-on, the reverse of Direct Connect Gateway's design.

### Resiliency

- For resilience: add a 2nd DX connection, or use a Site-to-Site IPSec VPN as a cheaper backup instead of paying for a second DX connection
- **High Resiliency** — one connection at multiple locations
- **Maximum Resiliency** — separate connections terminating on separate devices at more than one location

### Data Transfer Tools

- **AWS DataSync** — copy data to S3, EFS, FSx, NFS, SMB, Snowcone
- **AWS DMS** (Database Migration Service) — copy/migrate databases
- **AWS SCT** (Schema Conversion Tool) — convert schemas between database engine types

## AWS Site-to-Site VPN

- Must enable **Route Propagation** on the VGW in the subnets' route table, and open ICMP inbound in security groups if you need to ping EC2 instances from on-prem
- A Site-to-Site VPN is a common (cheaper) **backup** for a Direct Connect connection, instead of paying for a second DX connection

> Exam-wording cue: a question about **backup/failover connectivity for an existing Direct Connect** connection, with cost or speed-of-setup emphasized, points to a **Site-to-Site VPN** — not a second DX connection. A second DX connection is the answer only when the question explicitly demands DX-level bandwidth/consistency for the backup path too (i.e. "Maximum Resiliency," see above).

> Exam-wording cue: "**dedicated private connection**" to AWS, "**guarantee uptime**" via a **backup** that's allowed to use the **public internet** as long as it's **encrypted** — **select two** → **AWS Direct Connect** (the dedicated private link) **+ AWS Site-to-Site VPN** (the encrypted, internet-based failover). Direct Connect has no built-in redundancy of its own; VPN is the standard, cheaper backup path specifically because it's already an encrypted tunnel over the public internet — no need for a second, costly DX connection just to handle failure scenarios.

### Virtual Private Gateway (VGW)

- The AWS-side VPN concentrator, attached to the VPC you're connecting; ASN is customizable

### Customer Gateway (CGW)

- The on-prem side (hardware or software); needs a public, internet-routable IP (or the public IP of a NAT-T-capable NAT device in front of it)

> Exam-wording cue: "**correct configuration** for an AWS Managed **Site-to-Site IPSec VPN**" → **one Customer Gateway (CGW)** — a *resource* representing your on-prem device's public IP/ASN, not the device itself — connected to **one Virtual Private Gateway (VGW)** attached to the VPC, with the connection always provisioning **two IPSec tunnels** for redundancy, each terminating at a separate AWS-managed endpoint in a different Availability Zone. A wrong-answer option describing only a single tunnel, or swapping which side is the CGW vs. VGW, is the standard distractor pattern here.

### AWS VPN CloudHub

- Low-cost hub-and-spoke model over multiple VPN connections terminating on the same VGW, for secure communication between multiple on-prem sites; still travels over the public internet; needs dynamic routing + route table config

## AWS Transit Gateway

- Central hub connecting on-premises networks and multiple VPCs
- Reduces operational complexity vs. mesh VPC peering
- Offers features beyond plain VPC peering (e.g., transitive routing)
- Access via a transit virtual interface
- Common pattern: **Direct Connect → Transit Gateway → Multiple VPCs**
- Regional resource, but Transit Gateways can be peered across regions; share cross-account via Resource Access Manager (RAM)
- Route tables on the TGW itself limit which attached VPCs can reach each other (segmentation)
- Works with Direct Connect Gateway and VPN connections; is the only AWS networking construct that supports IP Multicast
- Can also be used to share a single Direct Connect connection across multiple accounts

> Exam-wording cue: "**simple solution**" to connect **VPCs and on-premises networks** "**through a central hub**," "**LEAST operational overhead**" → **AWS Transit Gateway**. "Central hub" is the literal tell — it rules out VPC Peering outright, since Peering has no hub concept at all (point-to-point only, non-transitive). Transit Gateway is the one managed resource that unifies both connection types at once: VPCs attach directly for transitive VPC-to-VPC routing, and on-premises reaches the same hub via a Direct Connect Gateway or VPN attachment — replacing what would otherwise be a full mesh of Peering connections plus separately-managed VPN/DX Gateway setups per VPC.
- **Centralizing PrivateLink access**: when multiple VPCs/accounts are already hub-and-spoke connected via TGW and all need private access to the same AWS service (Interface VPC Endpoint), deploy that endpoint **once in a single "shared services" VPC** and route every spoke VPC to it through the existing TGW — instead of duplicating the endpoint (and its per-AZ, per-VPC hourly cost) in every spoke individually
- **ECMP (Equal-Cost Multi-Path)** — spreads traffic across multiple Site-to-Site VPN tunnels to multiply bandwidth (e.g. combining tunnels for 2.5/5.0/7.5 Gbps); billed per-GB of TGW-processed data on top of the VPN cost

> Exam-wording cue: need more bandwidth out of a **Site-to-Site VPN** beyond a single tunnel's cap (~1.25 Gbps) → **ECMP** across multiple VPN tunnels via Transit Gateway. Need more bandwidth out of **Direct Connect** itself → provision a **LAG (Link Aggregation Group)** or a faster/additional DX connection, not ECMP — ECMP only multiplies VPN tunnel throughput through a TGW, it doesn't apply to DX.

> Exam-wording cue: "**surge in traffic**" across an existing **Site-to-Site VPN**, users experiencing **slower connectivity**, "**maximize the VPN throughput**" → **ECMP across multiple VPN tunnels via a Transit Gateway**. The slowdown is the tell that a single tunnel's ~1.25 Gbps cap has been hit — no per-tunnel setting fixes that, since it's a hard ceiling, not a tunable parameter. ECMP is what actually multiplies aggregate bandwidth by spreading traffic across several tunnels at once (e.g. 2.5/5.0/7.5 Gbps by combining tunnels), at the cost of TGW's per-GB data-processing charge on top of the existing VPN cost.

> Exam-wording cue: "multiple VPCs/accounts already connected via **Transit Gateway**, need **shared access to a common AWS service**, reduce cost **and** admin overhead" → **centralize Interface VPC Endpoints in one shared-services VPC**, reached by every spoke through the existing Transit Gateway — not one endpoint per VPC. TGW already provides the connectivity; use it to avoid duplicating endpoint deployments (and their per-AZ, per-VPC cost) across every account.

## Notes

<!-- Your own notes go here. -->

Content sourced from the slide deck, pages 721-870, has been merged into the topical sections above (Direct Connect, Transit Gateway, Site-to-Site VPN) rather than kept as standalone slide-page dumps or a sibling "additional detail" section.
