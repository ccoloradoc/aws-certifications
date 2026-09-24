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

### Virtual Private Gateway (VGW)

- The AWS-side VPN concentrator, attached to the VPC you're connecting; ASN is customizable

### Customer Gateway (CGW)

- The on-prem side (hardware or software); needs a public, internet-routable IP (or the public IP of a NAT-T-capable NAT device in front of it)

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
- **ECMP (Equal-Cost Multi-Path)** — spreads traffic across multiple Site-to-Site VPN tunnels to multiply bandwidth (e.g. combining tunnels for 2.5/5.0/7.5 Gbps); billed per-GB of TGW-processed data on top of the VPN cost

> Exam-wording cue: need more bandwidth out of a **Site-to-Site VPN** beyond a single tunnel's cap (~1.25 Gbps) → **ECMP** across multiple VPN tunnels via Transit Gateway. Need more bandwidth out of **Direct Connect** itself → provision a **LAG (Link Aggregation Group)** or a faster/additional DX connection, not ECMP — ECMP only multiplies VPN tunnel throughput through a TGW, it doesn't apply to DX.

## Notes

<!-- Your own notes go here. -->

Content sourced from the slide deck, pages 721-870, has been merged into the topical sections above (Direct Connect, Transit Gateway, Site-to-Site VPN) rather than kept as standalone slide-page dumps or a sibling "additional detail" section.
